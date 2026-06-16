#  Welcome to THEBUNCE Network
 # <img src="images\Readme-images" width="800" height="500" />
The purpose of this repo is to document my journey and the processes I use through building my homelab, the homelab will be used for hosting critical services that i used on a daily basis and to hone my skills in deploying secure systems

# <img src="Images/Readme-images/Server-Icon.png" width="25" height="25" /> Goals/Objectives

The main objectives i hope to achieve with this homelab is the following
- High level understanding of github branching and merging 
- Ability to clearly document my journey and identify the best method for doing this
- To have a Homelab which is secure and as safe as possible (without disconnecting it from the internet :))

The Main Services i want to have running (note this list may change)
- SIEM (such as wazuh)
- Syslog Server (such as Graylog)
- Automation of updates and patching (Anisble/n8n)
- Self Hosted AI (Hermes for example)
- Windows AD server
- Docker Server hosting production application such as (Lubelogger, Mealie, Gitea, and tududi)
- Firewall with defined and commented ruling (such as OPNSENSE)
- Reverse Proxy (Traefik)
- Proxmox VM Backup server (ProxmoxBackupServer (PBS))
- Shared Network drive with locked down access (TrueNas)
- Uptime and Service Monitoring (uptime-kuma, etc)
- Central Repository of Assets (NetBox)
- Lab environment (EVE-NG)
- Coding VM (Python)
- Pentesting/Automated Scanner (Linux Mint, Lynis, OpenVAS etc)

To do all of the above i will be using the following servers

# <img src="Images/Readme-images/Server-Icon.png" width="25" height="25" /> Servers & Hardware

# <img src="Images/Readme-images/Server-Icon.png" width="25" height="25" /> Prod Server BUNCEPROX01
# <img src="Images/Readme-images/BUNCEPROX-01.jpeg" width="800" height="500" />
-Specs: 
- CPU: Ryzen Threadripper 1950x 16 cores 32 threads
- RAM: 128GB Ram
- GPU 1: RTX 3090 24GB
- GPU 2: Nvidia Quadro K620
- Mobo: ASUS PRIME X399-A MotherBoard
- Chassis: 5U Case

-Storage
- CT4000BX500SSD101	4000
- Kingston SA400	480
- Kingston SA400	480
- nvme0n1	512GB


# <img src="Images/Readme-images/Server-Icon.png" width="25" height="25" /> Dev Server BUNCEPROX02
# <img src="Images/Readme-images/BUNCEPROX-02.jpeg" width="800" height="800" />
-Specs:
- CPU: Ryzen Threadripper 1920x 16 cores 32 threads
- RAM: 32GB Ram (8 x 2)
- Mobo: X399 AORUS Gaming 7
- Chassis: 4U

- Storage: 
- 1 ST2000	2000
- 2 Fanxiang_S100	512
- 3 CT1000BX500SSD1	1000
- 4 AFOX	240

-Description
- This host is used to run the DEV environment for Docker and the AI server (as it contains the RTX 3090 GPU), it also is used as a secondary PC running Linux. 


# <img src="Images/Readme-images/Server-Icon.png" width="25" height="25" /> Backup Server BUNCEPROX03
# <img src="Images/Readme-images/BUNCEPROX-03.jpeg" width="800" height="800" />
-Specs:
- CPU: I5-4590
- RAM: 16GB DDR3 (4 x 4)
- Mobo: ROG MAXIMUS VII RANGER
- Chassis: 4U

- Storage:
- Samsung 860 EVO 250
- CT4000BX500SSD101	4000
- ST1000DM003	1000
- ST1000DM003	1000
- ST2000VM003	2000

-Description
- This host is my main PC and is used for gaming and accessing the hosts via vscode. 

# <img src="Images/Readme-images/Server-Icon.png" width="25" height="25" /> Main Rack 1
# <img src="Images/Readme-images/LargeDepthServerRack.jpeg" width="800" height="800" />

# <img src="Images/Readme-images/Server-Icon.png" width="25" height="25" /> Main Rack 2
# <img src="Images/Readme-images/SamsonServerRack.jpeg" width="800" height="800" />