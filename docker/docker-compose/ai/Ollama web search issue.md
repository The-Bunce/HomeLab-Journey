## 🛠️ Troubleshooting Log

### AI Unable to Query SearXNG

**Symptom**  
AI tooling could not pull results from SearXNG — requests were failing silently.
**Root Cause**  
Two issues compounding:
| # | Area | Problem | Fix |
| --- | --- | --- | --- |
| 1 | Ports | SearXNG and Lubelogger were both bound to `8080` | Moved SearXNG to `8081` |
| 2 | Settings | SearXNG was returning HTML only — the JSON API was disabled | Enabled the `json` format in `settings.yml` |

**Resolution**

1.  **Port conflict** — Lubelogger was already occupying `8080:8080`, so SearXNG never came up on its expected port. Rebound SearXNG to `8081`:
    
    ```yaml
    ports:
      - "8081:8080"
    ```
    
2.  **JSON API disabled** — the AI client queries the SearXNG JSON endpoint, but the defaults only serve HTML. Added `json` to the `formats` list in `settings.yml`:
    
    ```bash
    # Read the documentation before extending the defaults:
    # https://docs.searxng.org/admin/settings/
    
    use_default_settings: true
    
    server:
      secret_key: "CHANGE_ME"   # <--- Set your own strong, random value
      image_proxy: true
    
    search:
      safe_search: 0
      autocomplete: ""
      default_lang: ""
      formats:
        - html
        - json   # <--- You MUST add this line manually
    ```
    
3.  **Rotate the secret key** — generate a fresh, strong value and swap it into the config, then restart the container:
    
    ```bash
    # Generate a new 64-character hex key
    openssl rand -hex 32
    
    # Restart SearXNG to apply the new settings + key
    docker compose restart searxng
    ```
    

> ⚠️ The `json` format is **not** included in the SearXNG defaults and must be added manually — without it the JSON API returns nothing and AI clients can't consume results.

> ⚠️ Never commit a real `secret_key` to a public repo — always use a generated value and rotate any key that has been exposed.

**Result**  
SearXNG now serves on `:8081` with the JSON API enabled, and AI tooling queries it successfully.

* * *

### 🔑 What the `secret_key` Does

In SearXNG, `secret_key` is a **server-side secret used to sign and encrypt a few sensitive things**. It's not a password and it doesn't authenticate your searches — it's an internal cryptographic seed. Concretely, it powers:

1.  **The rate-limiter / bot detection 🔐** — SearXNG signs the client IP + user-agent limiter cookie using this key, which is how it distinguishes a returning, rate-limited client from a fresh one.
    
2.  **Session / form signing (CSRF)** — it signs internal tokens so stateful requests can be verified as genuinely coming from SearXNG and not forged by a third party.
    
3.  **Encryption of small server-side data** — anything signed for a client is derived from this key, so it must be unique and unpredictable.
    

| Concern | Why the key is involved |
| --- | --- |
| **Security** | A weak or predictable key lets an attacker forge limiter/CSRF tokens and bypass bot protection |
| **HA / clustering** | If you run two SearXNG nodes behind a load balancer, they must share the _same_ key so signed cookies are valid across both |
| **Uniqueness** | Every instance should have its _own_ key — random, secret, and never committed |

**What it does _not_ do:**

*   ❌ It does not authenticate _you_ or gate the JSON API
    
*   ❌ It doesn't change search results
    
*   ❌ It's not a login password
    

> 💡 **Bottom line:** it's the seed SearXNG uses to make signed cookies/tokens trustworthy — random and unique per instance, kept secret, and shared only when clustering multiple instances.

* * *

### ✅ How to Verify the Key Changed

The key only "counts" if it's (a) actually in the file SearXNG reads, and (b) the container restarted to pick it up.

1.  **Confirm the file SearXNG reads has the new key** — SearXNG reads its active config from `/etc/searxng/settings.yml` inside the container:
    
    ```bash
    docker compose exec searxng grep -n secret_key /etc/searxng/settings.yml
    ```
    
    You should see your new 64-char hex value (or `CHANGE_ME` if you haven't swapped it yet) — **not** the old one.
    
2.  **Confirm the old key is gone everywhere** — substitute whatever the _previous_ key was:
    
    ```bash
    # Should return NOTHING if fully replaced (replace <OLD_KEY> with the previous value)
    docker compose exec searxng grep -rn "<OLD_KEY>" /etc/searxng/
    docker compose config | grep -n secret_key
    ```
    
3.  **Confirm the container actually restarted:**
    
    ```bash
    # "Up X seconds/minutes" proves it's a fresh process
    docker compose ps searxng
    
    # Look for a clean startup, no errors
    docker compose logs --tail 20 searxng
    ```
    
4.  **Sanity-check the service responds:**
    
    ```bash
    # JSON API should return valid JSON (confirm it's alive + serving)
    curl -s "http://localhost:8081/search?q=test&format=json" | head -c 200
    ```
    

> ⚠️ SearXNG's `secret_key` is used for **bot-detection / rate-limiting** and CSRF/session signing — it does **not** change search results. So the real "proof" it changed is the file + fresh process checks, not the search output. The cleanest single check is step 1.