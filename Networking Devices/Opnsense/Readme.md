# Homelab Issue

The main issue I've identified with my homelab is that updating the firewall causes me to lose access to the rest of the network and the internet. To resolve this, I've decided to deploy BUNCEPROX02 as a secondary firewall gateway and configure both in High Availability mode.

# <img src="/Images/Readme-images/Cogs-Icon.png" width="25" height="25" /> Requirements

    2 Ethernet ports (I used 2× TP-Link USB Ethernet adapters)
    Proxmox Virtual Environment
    Recommended hardware specs for OPNsense: docs.opnsense.org/manual/hardware.html

The VM will be allocated:

    8 GB RAM
    4 CPU cores
    50 GB storage

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step-by-Step Guide

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 1 – Download the ISO

First, grab the OPNsense installer ISO from the official download page. Click the "Download OPNsense" button, right-click → Copy Link Address, then head back to Proxmox:

    Go to local → ISO images
    Click Download from URL
    Paste the URL you copied and let it download

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 2 – Create the VM

With Proxmox open, click Create VM and walk through the wizard. Allocate at least the specs listed above (8 GB RAM / 4 cores / 50 GB disk) so the firewall has room to breathe.

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 3 – Passthrough & Install

    ⚠️ Don't boot the VM yet.

Before starting it, pass through both PCIe NICs:
VM → Hardware → Add → PCI Device → Raw Device → select each adapter from the list.

Now boot the VM. On first launch OPNsense will ask you to log in as root or installer — choose installer to begin the OS setup. Follow the wizard through to the homescreen.

Issues I ran into

So, a few things tripped me up during this:

First off — if you've got a switch with VLANs set up on it, make sure the cable you're plugged into for the install is on an untagged LAN-side port. Trust me, I learned this the hard way. You'll lose access mid-install and it's not fun figuring out why.

Second — once the install finishes and you're at the homescreen, I'd reboot the Proxmox host just to be safe. For whatever reason, my WAN interface wouldn't come up properly until I did. 

Third — if you already have an OPNsense box running, don't bother re-entering all your settings by hand. Export the config from the old one, nudge the bits that need changing (VLAN IPs, interface names, whatever), then just import it straight onto the new firewall. That's honestly what I ended up doing and it saved me a ton of time.

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 4 – Configure High Availability

With both firewalls installed and patched in, it's time to set up the High Availability (CARP) pair on OPNsense so traffic can fail over automatically if one node goes down.


# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" align="middle" /> Step 5: High Availability (HA) Setup

## Step 1: Pre-Configuration & Virtual IP

1. **Log in to both firewalls.**
2. **Ensure both firewalls are running the same patch level.** This is critical for compatibility.
3. **Review all interfaces and reserve an IP address for the virtual IP (VIP).**
   - You may need to reassign interface IPs to free up a suitable address.
   - *Example:* I was running a single firewall with `.1` on the interface. I migrated that to make `.1` available as the shared virtual IP.
4. **Designate the HA link interface.**
   - Navigate to **Interfaces**.
   - Choose the interface to use for the HA link.
   - I used a dedicated VLAN for this, but if you have enough physical ports, I'd recommend assigning a dedicated physical interface.
   - *(Optional)* Rename the interface to `HA` for easier identification of traffic.
5. **Create the Virtual IP (CARP).**
   - Navigate to **Virtual IPs → Settings** and click the **+** button.
   - Set **Mode** to `CARP`.
   - Select the appropriate **Interface** (ensure interfaces are created for each network, including VLANs).
   - Assign the IPs:
     - **FW 1:** `10.10.4.2`
     - **FW 2:** `10.10.4.100`
     - **Virtual IP (shared gateway):** `10.10.4.1`
   - Set a **password** for the HA connection.
   - Click **Save**.

## Step 2: Configure the HA Link

**On the primary firewall:**

1. Navigate to **System → High Availability → Settings**.
2. Enter the **Synchronize peer IP** — this should be the IP of the backup firewall.
3. Enable **Synchronize config**.
4. Enter the **username and password** for the backup firewall.
5. Click **Apply**.

**On the backup firewall:**

1. Navigate to **System → High Availability → Settings**.
2. Enter the **IP address of the primary (live) firewall**.
3. No username or password is required here.

## Step 3: Select Services to Fail Over

On **both** firewalls, ensure the same services are selected for failover. I chose the following:

- Aliases
- Firewall Groups
- Firewall Rules
- Kea DHCP
- Virtual IPs

> 🎉 **Congrats! You now have High Availability on your firewalls!** 🎉
>
> 🔥 Both firewalls are in sync, failover is active, and your network is protected.