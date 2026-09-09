**# Firewall Availability Issues**
  

The main issue I've identified with my homelab is that updating the firewall causes me to lose access to the rest of the network and the internet. To resolve this, I've decided to deploy BUNCE-FW-02 as a secondary firewall gateway and configure both in High Availability mode.
  

**# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Requirements**
  

- 2 Ethernet ports (I used 2× TP-Link USB Ethernet adapters)
- Proxmox Virtual Environment
- Recommended hardware specs for OPNsense: https://docs.opnsense.org/manual/hardware.html
  

The VM will be allocated:
  

- 8 GB RAM
- 4 CPU cores
- 50 GB storage
  

**# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 1 – Download the ISO**
  

First, grab the OPNsense installer ISO from the official download page. Click the "Download OPNsense" button, right-click → Copy Link Address, then head back to Proxmox:
  

1. Go to local → ISO images
2. Click Download from URL
3. Paste the URL you copied and let it download
  

**# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 2 – Create the VM**
  

With Proxmox open, click Create VM and walk through the wizard. Allocate at least the specs listed above (8 GB RAM / 4 cores / 50 GB disk) so the firewall has room to breathe.
  

**# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 3 – Passthrough & Install**
  

> ⚠️ Don't boot the VM yet.
  

Before starting it, pass through both USB NICs:
  

1. VM → Hardware → Add → USB Device
2. Select each TP-Link adapter from the list
3. Repeat for the second adapter
  

Now boot the VM. The installer starts automatically. If you land on a shell prompt instead, type `installer` to launch the setup wizard. Follow the wizard through to the homescreen.
  

**# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 4 – Configure High Availability (CARP)**
  

With both firewalls installed and patched in, it's time to set up the High Availability (CARP) pair on OPNsense so traffic can fail over automatically if one node goes down.
  

**## 4a. Pre-Configuration & Virtual IP**
  

1. Log in to both firewalls.
2. Ensure both firewalls are running the same patch level. This is critical for compatibility.
3. Review all interfaces and reserve an IP address for the virtual IP (VIP).
   - You may need to reassign interface IPs to free up a suitable address.
   - Example: I was running a single firewall with .1 on the interface. I migrated that to make .1 available as the shared virtual IP.
4. Designate the HA link interface.
   - Navigate to Interfaces.
   - Choose the interface to use for the HA link.
   - I used a dedicated VLAN for this, but if you have enough physical ports, I'd recommend assigning a dedicated physical interface.
   - (Optional) Rename the interface to HA for easier identification of traffic.
5. Create the Virtual IP (CARP).
   - Navigate to Virtual IPs → Settings and click the + button.
   - Set Mode to CARP.
   - Select the appropriate Interface (ensure interfaces are created for each network, including VLANs).
   - Assign the IPs:
     - BUNCE-FW-01: 10.10.4.2
     - BUNCE-FW-02: 10.10.4.100
     - Virtual IP (shared gateway): 10.10.4.1
     - These are my addresses — use whatever fits your subnet.
   - Click Save.
  

**## 4b. HA Sync Settings**
  

On BUNCE-FW-01 (primary):
  

1. Navigate to System → High Availability → Settings.
2. Enter the Synchronize peer IP — this should be the IP of BUNCE-FW-02.
3. Enable Synchronize config.
4. Enter the admin username and password for BUNCE-FW-02.
5. Click Apply.
  

On BUNCE-FW-02 (backup):
  

1. Navigate to System → High Availability → Settings.
2. Enter the Synchronize peer IP — this should be the IP of BUNCE-FW-01.
3. No credentials are required here.
4. Click Apply.
  

**## 4c. Select Services to Fail Over**
  

On both firewalls, navigate to System → High Availability → Settings and ensure the same services are selected for failover. I chose the following:
  

- Aliases
- Firewall Groups
- Firewall Rules
- Kea DHCP
- Virtual IPs
  

> Note: If you're still using the "disable Kea on the backup" workaround (see Troubleshooting below), leave Kea DHCP unchecked until you've deployed the watchdog script in Step 5.
  

**# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 5 – Kea DHCP CARP Failover Watchdog**
  

If you have Kea DHCP running on your firewall, it will cause split-brain once both nodes are active. The quick fix is to disable Kea on the backup (BUNCE-FW-02), but the proper fix is a watchdog script that starts/stops Kea based on CARP state.
  

Create /usr/local/etc/rc.d/kea-dhcp-ha on both firewalls:
  

```bash
#!/bin/sh
# kea-dhcp-ha – CARP-aware Kea DHCP watchdog
# Ensures only the CARP master runs Kea DHCP.
#
# On BUNCE-FW-02, set MANAGE_CTRL_AGENT="yes" to also
# manage kea-ctrl-agent.

# --- Configuration (adjust to your setup) ---
CARP_IF="carp0"             # CARP interface (check with: ifconfig | grep carp)
PIDFILE="/var/run/kea/kea-dhcp4.pid"
CTRLPID="/var/run/kea/kea-ctrl-agent.pid"
CHECK_INTERVAL=5            # seconds between checks
MANAGE_CTRL_AGENT="no"      # set to "yes" on BUNCE-FW-02

# --- Helpers ---
is_master() {
    ifconfig "$CARP_IF" 2>/dev/null | grep -q "MASTER"
}

start_kea() {
    if [ ! -f "$PIDFILE" ]; then
        logger -t kea-dhcp-ha "CARP master detected – starting kea-dhcp4"
        service kea-dhcp4 start
    fi
    if [ "$MANAGE_CTRL_AGENT" = "yes" ] && [ ! -f "$CTRLPID" ]; then
        logger -t kea-dhcp-ha "Starting kea-ctrl-agent"
        service kea-ctrl-agent start
    fi
}

stop_kea() {
    if [ -f "$PIDFILE" ]; then
        logger -t kea-dhcp-ha "Demoted to BACKUP – stopping kea-dhcp4"
        service kea-dhcp4 stop
    fi
    if [ "$MANAGE_CTRL_AGENT" = "yes" ] && [ -f "$CTRLPID" ]; then
        logger -t kea-dhcp-ha "Stopping kea-ctrl-agent"
        killall kea-ctrl-agent 2>/dev/null
        rm -f "$CTRLPID"
    fi
}

# --- Main loop ---
while true; do
    if is_master; then
        start_kea
    else
        stop_kea
    fi
    sleep "$CHECK_INTERVAL"
done
```
  

> Adjust CARP_IF, PIDFILE, and CTRLPID if your Kea version writes them elsewhere. Run `ls /var/run/kea/` to verify.
  

Now deploy it on both boxes:
  

```bash
# Verify the script looks correct
cat /usr/local/etc/rc.d/kea-dhcp-ha

# Make it executable & enable at boot
chmod +x /usr/local/etc/rc.d/kea-dhcp-ha
sysrc kea_dhcp_ha_enable="YES"

# Start it now
service kea-dhcp-ha start

# Confirm it's running
ps aux | grep kea-dhcp-ha

# Check which Kea daemons are active
ps aux | grep -i kea
```
  

> Note: service start already runs the script in the background. You do not need nohup … & separately.
  

To stop or troubleshoot:
  

```bash
# Stop the watchdog (kills script + children)
pkill -f kea-dhcp-ha

# Confirm it's gone
ps aux | grep kea-dhcp-ha

# Disable at boot
sysrc kea_dhcp_ha_enable="NO"
```
  

The difference between the two boxes:
  

| | BUNCE-FW-01 | BUNCE-FW-02 |
|:--|:--|:--|
| Daemons managed | kea-dhcp4 only | kea-dhcp4 + kea-ctrl-agent |
| MANAGE_CTRL_AGENT | "no" | "yes" |
| Ctrl-agent cleanup on failover | — | killall kea-ctrl-agent + PID-file removal |
  

**# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Troubleshooting & Pitfalls**
  

These are the issues that tripped me up. They don't block the steps above, but they'll save you an hour.
  

**1. VLAN switch: use an untagged LAN port for the install.**
  

If you've got a switch with VLANs set up, make sure the cable you're plugged into for the install is on an untagged LAN-side port. You'll lose access mid-install and it's not fun figuring out why.
  

**2. Reboot the Proxmox host after install.**
  

Once the install finishes and you're at the homescreen, reboot the Proxmox host. For whatever reason, my WAN interface wouldn't come up properly until I did.
  
**3. Import config instead of re-entering by hand.**
  
If you already have an OPNsense box running, don't bother re-entering all your settings. Export the config from the old one, nudge the bits that need changing (VLAN IPs, interface names, whatever), then import it straight onto the new firewall. That's what I ended up doing and it saved me a ton of time.
  
**4. Kea DHCP split-brain.**
  
If you have Kea DHCP enabled, you'll experience split-brain once both nodes are active. The quick fix I used was to disable the service on BUNCE-FW-02. The proper fix is the watchdog script in Step 5 — deploy it, re-enable Kea on the backup, then add Kea DHCP to the failover service list in Step 4c.
  

**# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Verifying Failover**
  
Before you trust the setup, do a quick failover test:
  
1. From a test client on the LAN, run ping 10.10.4.1 (your VIP) and confirm it responds.
2. On BUNCE-FW-01, check CARP state: ifconfig carp0 should show MASTER.
3. Shut down BUNCE-FW-01 (Power Off, not Halt, so it doesn't try to release the VIP gracefully).
4. Wait 5–10 seconds. On BUNCE-FW-02, ifconfig carp0 should now show MASTER.
5. Confirm the test client can still ping the VIP and that DHCP hands out addresses.
6. Power BUNCE-FW-01 back on. It should come back up as BACKUP and stay quiet.
7. Check System → High Availability → Status on both boxes — they should report the same config revision.