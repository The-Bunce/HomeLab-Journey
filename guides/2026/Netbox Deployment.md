# NetBox Homelab — Deployment & Incident Report

> **Objective:** Document the layout, connections, and virtual machines running within the homelab. NetBox was chosen for its versatility, active community, and enterprise-grade feature set.

**Stack:** Docker Compose · NetBox v4.6 · PostgreSQL 18 · Valkey 9.0 (Redis-compatible)

* * *

## Table of Contents

*   [Prerequisites](#prerequisites)
    
*   [Step-by-Step Deployment](#step-by-step-deployment)
    *   [Step 1 — Project Directory](#step-1--project-directory)
        
    *   [Step 2 — Clone the Repository](#step-2--clone-the-repository)
        
    *   [Step 3 — Pull Images & Initial Boot](#step-3--pull-images--initial-boot)
        
    *   [Step 4 — Hardening: Rotate All Default Credentials](#step-4--hardening-rotate-all-default-credentials)
        
    *   [Step 5 — Clean Rebuild](#step-5--clean-rebuild)
        
    *   [Step 6 — Handle Postgres Readiness](#step-6--handle-postgres-readiness)
        
    *   [Step 7 — Create Superuser](#step-7--create-superuser)
        
    *   [Step 8 — Verification & Security Checks](#step-8--verification--security-checks)
        
*   [Incident:](#incident-redis-cache-silent-death) `redis-cache` [Silent Death](#incident-redis-cache-silent-death)
    *   [Symptoms](#symptoms)
        
    *   [Investigation](#investigation)
        
    *   [Root Cause](#root-cause)
        
    *   [The Fix](#the-fix)
        
    *   [Verification](#verification-after-fix)
        
*   [Lessons Learned](#lessons-learned)
    
*   [Future Improvements](#future-improvements)
    

* * *

## Prerequisites

| Requirement | Details |
| --- | --- |
| **OS** | Linux (Debian/Ubuntu recommended) |
| **Docker Engine** | v25+ with Compose plugin v2 |
| **Docker Compose** | `docker compose version` — must support v2 syntax |
| **RAM** | 4 GB minimum (Postgres + NetBox + 2× Valkey + worker) |
| **Disk** | 10 GB free for images + data volumes |
| **Port** | 8080 (NetBox web UI) — internal only, not exposed to LAN |
| **Git** | For cloning the deployment repo |

```bash
# Verify Docker is running
docker --version
docker compose version
docker ps
```

* * *

## Step-by-Step Deployment

### Step 1 — Project Directory

Create a dedicated project directory. NetBox's Docker deployment generates multiple folders (`env/`, `configuration/`, `netbox/`, `postgres/`, `redis/`, `redis-cache/`), so give it room to breathe.

```bash
mkdir -p ~/projects && cd ~/projects
```

### Step 2 — Clone the Repository

Clone the **release** branch of the official community Docker deployment. This is the tested, stable variant — not `main` which tracks development.

```bash
git clone -b release https://github.com/netbox-community/netbox-docker.git
cd netbox-docker
```

> **Note:** The repo's `docker-compose.yml` is the base. You can layer overrides on top via `docker-compose.override.yml` (covered below).

### Step 3 — Pull Images & Initial Boot

Copy the override example so you can customise later (port mappings, extra env vars, resource limits):

```bash
cp docker-compose.override.yml.example docker-compose.override.yml
```

Pull all images (NetBox, Postgres, 2× Valkey, worker):

```bash
docker compose pull
```

> ⚠️ **Do NOT run** `createsuperuser` **yet.** The superuser is created _after_ the credential rotation in Step 4. Creating it now would reset it when we wipe the database.

Start the stack in the foreground so you can watch the boot logs:

```bash
docker compose up
```

You should see Postgres initialise, NetBox run its migrations, and the two Valkey instances come up. Leave this running — we'll `Ctrl+C` in Step 5.

### Step 4 — Hardening: Rotate All Default Credentials

The stock `env/` files ship with **weak or placeholder passwords**. Before putting NetBox on your network, rotate every secret.
**4a. Back up the original env files** (safety net):

```bash
cd env/
for f in *.env; do cp "$f" "${f}.backup"; done
```

**4b. Generate strong random secrets:**

```bash
# 32 bytes (256-bit) — good for passwords, API tokens, DB credentials
openssl rand -base64 32

# 50 bytes (~400-bit) — good for SECRET_KEY (Django requires long random strings)
openssl rand -base64 50
```

**4c. Update the following variables in the relevant env files:**
| Variable | File(s) | Purpose |
| --- | --- | --- |
| `POSTGRES_PASSWORD` / `DB_PASSWORD` | `postgres.env`, `netbox.env` | Database authentication. **Must match across files.** |
| `REDIS_PASSWORD` | `redis.env`, `netbox.env`, `netbox-worker.env` | RQ job queue auth. **Must match across files.** |
| `REDIS_CACHE_PASSWORD` | `redis-cache.env`, `netbox.env`, `netbox-worker.env` | Django cache auth. **Must match across files.** |
| `SECRET_KEY` | `netbox.env`, `netbox-worker.env` | Django signing key (sessions, CSRF, crypto). **Must match across files.** |
| `API_TOKEN_PEPPER_1` | `netbox.env`, `netbox-worker.env` | Salt for API token hashing. **Must match across files.** |

> ⚠️ **Critical:** Every variable that appears in `netbox.env` **must have the identical value** in `netbox-worker.env` and any other file that references it. A mismatch will cause silent auth failures that are very hard to debug.

**4d. Quick sanity check:**

```bash
cd env/
grep -E 'PASSWORD|SECRET_KEY|PEPPER' netbox.env postgres.env redis.env redis-cache.env netbox-worker.env
```

Visually confirm the values match across files.

### Step 5 — Clean Rebuild

Because we changed the database password, the existing Postgres data (initialised with the old password) is incompatible. We need to wipe the volumes and let Postgres re-initialise.

> ⚠️ `-v` **flag removes all Docker volumes.** Any data in NetBox (sites, devices, IPAM, etc.) will be lost. This is only safe on a fresh install.

```bash
# Stop everything and remove volumes
docker compose down -v

# Start the stack again in detached mode
docker compose up -d
```

### Step 6 — Handle Postgres Readiness

On first boot, NetBox will likely **fail to start** because PostgreSQL is still:

1.  Initialising a fresh data directory
    
2.  Creating roles and databases
    
3.  Running NetBox's ~100+ database migrations
    
4.  Seeding initial data (content types, permissions, etc.)
    

NetBox's `depends_on` tells Docker to start Postgres _first_, but it doesn't wait for Postgres to be _ready_. This is a known race condition.
**Fix — just restart:**

```bash
docker compose down
# Wait for all containers to fully stop
docker compose up -d
```

The second boot succeeds because Postgres already has the data directory initialised and just needs to replay migrations. Give it 2–3 minutes.
**Verify:**

```bash
docker compose ps
# All services should show "healthy"

docker compose logs netbox --tail 20
# Should show "Starting development server" or similar
```

### Step 7 — Create Superuser

Now that the database is populated and stable, create the admin account:

```bash
docker compose exec netbox /opt/netbox/netbox/manage.py createsuperuser
```

You'll be prompted for username, email, and password. Use a strong password — this is your root login.

### Step 8 — Verification & Security Checks

**8a. Confirm all services are healthy:**

```bash
docker compose ps
```

Expected output (all `healthy` or `Up`):
| Service | Health |
| --- | --- |
| `netbox` | ✅ healthy |
| `netbox-worker` | ✅ healthy |
| `postgres` | ✅ healthy |
| `redis` | ✅ healthy |
| `redis-cache` | ✅ healthy |

**8b. Confirm ports are NOT exposed to the host:**

```bash
docker ps
```

You want to see:

```
8080/tcp          ← internal only (Docker network)
5432/tcp          ← internal only
6379/tcp          ← internal only
```

You do **NOT** want to see:

```
0.0.0.0:8080->8080/tcp    ← EXPOSED to all interfaces
0.0.0.0:5432->5432/tcp    ← EXPOSED to all interfaces
```

If ports are exposed, remove the port mappings from `docker-compose.override.yml` and `docker compose up -d` again.
**8c. Test the web UI:**

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/login/
# → 200
```

Log in with your superuser credentials.
**8d. Verify Redis memory limits are set** (after applying the incident fix below):

```bash
docker exec netbox-docker-redis-cache-1 valkey-cli -a $REDIS_CACHE_PASSWORD INFO memory | grep maxmemory_human
docker exec netbox-docker-redis-1 valkey-cli -a $REDIS_PASSWORD INFO memory | grep maxmemory_human
```

* * *

## Incident: `redis-cache` Silent Death

### Symptoms

After ~37 hours of uptime, the NetBox stack became unresponsive:

*   Web UI returning **500 Internal Server Error**
    
*   RQ worker crashing repeatedly with `ConnectionInterrupted`
    
*   `redis-cache` container showing **Exited (255)**
    
*   Primary `redis` (RQ queue) still **Up and healthy**
    
*   **No automatic recovery** — stayed down until manual intervention
    

**RQ worker traceback (abridged):**

```
django_redis.exceptions.ConnectionInterrupted: Redis ConnectionError:
  Error -3 connecting to redis-cache:6379. Temporary failure in name resolution.

redis.exceptions.ConnectionError: Error -3 connecting to redis-cache:6379.
  Temporary failure in name resolution.
```

> ⚠️ **This error is misleading.** It reads like a DNS or networking failure, but the real cause is that the `redis-cache` container was already dead. Docker's embedded DNS (127.0.0.11) cannot resolve the name of a container that isn't running.

### Investigation

**Step 1 — Check container status:**

```bash
docker ps -a | grep redis
```

```
9f822b0b94   valkey/valkey:9.0-alpine   "docker-entrypoint.s…"   41 hours ago   Exited (255) 37 hours ago   6379/tcp   netbox-docker-redis-cache-1
49f84ae2aaeb   valkey/valkey:9.0-alpine   "docker-entrypoint.s…"   41 hours ago   Up 37 hours (healthy)       6379/tcp   netbox-docker-redis-1
```

The cache instance was dead. The queue instance was fine.
**Step 2 — Check the dead container's logs:**

```bash
docker logs netbox-docker-redis-cache-1 --tail 50
```

```
1:M 20 Sep 2026 15:35:08 * Done loading RDB, keys loaded: 3, keys expired: 0.
1:M 20 Sep 2026 15:35:08 * Ready to accept connections tcp
1:M 20 Sep 2026 16:35:09 * 1 changes in 3600 seconds. Saving...
24355:C 20 Sep 2026 16:35:09 * DB saved on disk
24355:C 20 Sep 2026 16:35:09 * Fork CoW for RDB: current 0 MB, peak 0 MB, average 0 MB
1:M 20 Sep 2026 16:35:09 * Background saving terminated with success
```

Clean logs, no crash, no error. The process was **killed from outside** — no time to write a shutdown message.
**Step 3 — Confirm OOM kill:**

```bash
dmesg -T | grep -i "oom\|killed process" | grep -i "valkey\|redis"
```

Confirmed: the kernel's OOM killer terminated the `redis-cache` process.
**Step 4 — Compare the two Redis services in** `docker-compose.yml`**:**

```yaml
# ✅ Primary Redis — has restart policy
redis:
    image: valkey/valkey:9.0-alpine
    restart: unless-stopped              # ← present
    command:
      - sh -c "valkey-server --appendonly yes --requirepass $$REDIS_PASSWORD"

# ❌ Cache Redis — MISSING restart policy, no memory cap
redis-cache:
    image: valkey/valkey:9.0-alpine
    # ← no restart: directive
    command:
      - sh -c "valkey-server --requirepass $$REDIS_PASSWORD"
      # ← no --maxmemory
```

### Root Cause

Two compounding issues:
| # | Issue | Impact |
| --- | --- | --- |
| 1 | **No** `restart` **policy** on `redis-cache` | When OOM-killed, Docker left it dead permanently. No auto-recovery. |
| 2 | **No** `maxmemory` **cap** on either Valkey instance | The cache grew unbounded over hours until the host ran out of RAM and the kernel killed the process. |

The `redis` (queue) instance survived because RQ job payloads are small and short-lived. The `redis-cache` instance held NetBox's full Django config cache, session data, and any per-request cached queries — this grows over time, especially with background jobs like "System Housekeeping" that the RQ worker runs periodically.

### The Fix

**1. Add** `restart` **policy to** `redis-cache`**:**

```yaml
redis-cache:
    restart: unless-stopped    # ← added
```

**2. Set memory limits on both Valkey instances:**

```yaml
redis:
    command:
      - sh -c "valkey-server --appendonly yes --requirepass $$REDIS_PASSWORD \
               --maxmemory 512mb --maxmemory-policy allkeys-lru"

redis-cache:
    command:
      - sh -c "valkey-server --requirepass $$REDIS_PASSWORD \
               --maxmemory 256mb --maxmemory-policy allkeys-lru"
```

| Setting | Value | Rationale |
| --- | --- | --- |
| `--maxmemory` (redis) | `512mb` | Generous headroom for RQ job queue |
| `--maxmemory` (redis-cache) | `256mb` | Sufficient for NetBox config + session cache |
| `--maxmemory-policy` | `allkeys-lru` | Evict least-recently-used keys when full. Safe because NetBox re-reads config from Postgres — the cache is a performance layer, not a source of truth. |

**3. Apply:**

```bash
cd ~/projects/netbox-docker
docker compose up -d redis-cache
```

No need to restart NetBox or the worker — they reconnect on the next cache call.

### Verification (After Fix)

```bash
# Both containers running with restart policies
docker ps | grep redis

# Confirm restart policy is active
docker inspect netbox-docker-redis-cache-1 --format='{{json .HostConfig.RestartPolicy}}'
# → {"Name":"unless-stopped","MaximumRetryCount":0}

# Confirm memory cap is enforced
docker exec netbox-docker-redis-cache-1 \
  valkey-cli -a "$REDIS_CACHE_PASSWORD" INFO memory | grep maxmemory_human
# → maxmemory_human: 256.00M

# Monitor for 24h
docker stats --no-stream
```

* * *

## Lessons Learned

*   **Always set** `restart: unless-stopped` **on every service.** A single missing line turned a recoverable OOM event into a 37-hour outage.
    
*   **Cap Redis/Valkey memory.** Without `maxmemory`, even a "small" cache grows unbounded. On a shared host (Proxmox, other containers), the OOM killer will eventually pick a victim — and it won't be polite about which one.
    
*   `allkeys-lru` **is the right eviction policy for NetBox.** The cache is disposable; Postgres is the source of truth.
    
*   **Error messages lie.** "Temporary failure in name resolution" pointed at DNS/networking. The real issue was a dead upstream container. Always check `docker ps -a` first.
    
*   **Race conditions on first boot.** `depends_on` controls _order_, not _readiness_. Plan for a restart cycle after initial migration.
    
*   **Rotate credentials before first use.** The stock env files are not production-safe.
    

* * *
