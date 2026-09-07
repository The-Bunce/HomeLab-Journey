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

Gotchas I hit along the way:

    If you're on a switch with VLANs, make sure your management cable is plugged into an untagged (LAN-side) port, otherwise you'll lose access mid-install.
    After the install finishes, I had to reboot the Proxmox host before the WAN interface came up properly. Do a reboot just to be safe.
    If you already have a running OPNsense box, you can export its config, tweak things like VLAN interface IPs, then import it onto the new firewall — that's the route I took and it saved a lot of re-typing.

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 4 – Configure High Availability

With both firewalls installed and patched in, it's time to set up the High Availability (CARP) pair on OPNsense so traffic can fail over automatically if one node goes down.


# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 5

