#  Welcome to THEBUNCE Network
 # <img src="images\Setup.jpeg" width="800" height="500" />
The purpose of this repo is to document my journey and the processes I use through building my homelab, the homelab will be used for hosting critical services that i used on a daily basis and to hone my skills in deploying secure systems, this will include:
- SIEM (Wazuh) with clamAV
- Vulnerability Scanner (OpenVAS)
- Firewall Setup (OPNsense)
- Login Security (Fail2Ban, PassKeys, and 2FA)
# Network Diagram
 # <img src="images\Network Layout.png" width="800" height="700" />
# <img src="images\Server Icon.png" alt="Hammer and Wrench" width="25" height="25" /> Racks
- # <img src="images\Downstairs.jpeg" width="500" height="500" />
- Contains: 
- BUNCEPROX02
- # <img src="images\Rack 2 pt1.jpeg" width="500" height="500" />
- Contains: 
- BUNCE-CISCO-ROUTER-01
- BUNCE-LINUX-02
- BUNCE-CISCO-ASA-01
- BUNCE-JUNIPER-01
- # <img src="images\Rack 1 pt2.jpeg " width="500" height="500" />
- Contains: 
- BUNCE-DESKTOP-01
- BUNCEPROX01
- BUNCE-FW-01
- BUNCE-CISCO-SW-01
# <img src="images\Server Icon.png" alt="Hammer and Wrench" width="25" height="25" /> Servers
# <img src="images\Server Icon.png"  alt="Hammer and Wrench" width="25" height="25" /> BUNCEPROX01
-Specs: 
- CPU: Ryzen Threadripper 1950x 16 cores 32 threads
- RAM: 128GB Ram
- Mobo: ASUS PRIME X399-A MotherBoard
- Chassis: 3U Case
-Storage
- 1	CT4000BX500SSD101	4000GB
- 2	CT4000BX500SSD101	4000GB
- 3	Kingston SA400	480GB
- 4	Kingston SA400	480GB
- 5	Kingston SA400	480GB
- 6	Kingston SA400	480GB
- 7	ST1000	1000GB
- 8	ST1000	1000GB
- 9	nvme0n1	512GB

-Description
- This is the main host which is used to run the Docker prod containers, proxmox vm backup server, Truenas scale NAS, and the DNS (pihole), this host is vital to ensure the network continues to run without issues. 

# <img src="images\Server Icon.png"  alt="Hammer and Wrench" width="25" height="25" /> BUNCEPROX02
-Specs:
- CPU: Ryzen 9 3900x (Manual Undervolt)
- RAM: 48GB Ram
- Mobo: MPG B550 GAMING PLUS
- Chassis: INWIN 905
- Storage: 
- 1 ST2000	2000
- 2 Fanxiang_S100	512
- 3 CT1000BX500SSD1	1000
- 4 AFOX	240

-Description
- This host is used to run the DEV environment for Docker and the AI server (as it contains the RTX 3090 GPU), it also is used as a secondary PC running Linux. 


# <img src="images\Server Icon.png" alt="Hammer and Wrench" width="25" height="25" /> BUNCE-DESKTOP-01
-Specs:
- CPU: Ryzen 9 5950x (Manual Undervolt)
- RAM: 48GB 3200mhz (32 + 16)
- Mobo: Gigabyte X570 AORUS Elite
- Chassis: 5U Silverstone
- Storage:
- 1 nvme0n1	1000GB

-Description
- This host is my main PC and is used for gaming and accessing the hosts via vscode. 

- # <img src="images\Firewall Icon.png" alt="Hammer and Wrench" width="25" height="25" /> BUNCEFW01
-Specs:
- CPU: i5-7500 CPU
- RAM: 8GB
- Chassis: HP ProDesk 400
- Storage:
- 1 nvme0n1	256GB

-Description
- This host is used for the firewall running OPNsense. 

- # <img src="images\Service.png" alt="Hammer and Wrench" width="25" height="25" /> Homepage
- The dashboard is my single source of access and also provides useful stats to ensure if issues occur i will be aware 
- # <img src="images\homepage.png" width="500" height="500" />