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

Second — once the install finishes and you're at the homescreen, I'd reboot the Proxmox host just to be safe. For whatever reason, my WAN interface wouldn't come up properly until I did. Sounded like a fluke, but yeah, it was real.

Third — if you already have an OPNsense box running, don't bother re-entering all your settings by hand. Export the config from the old one, nudge the bits that need changing (VLAN IPs, interface names, whatever), then just import it straight onto the new firewall. That's honestly what I ended up doing and it saved me a ton of time.

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 4 – Configure High Availability

With both firewalls installed and patched in, it's time to set up the High Availability (CARP) pair on OPNsense so traffic can fail over automatically if one node goes down.


# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 5

