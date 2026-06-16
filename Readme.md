#  Welcome to THEBUNCE Network
 # <img src="images\Readme-images" width="800" height="500" />
The purpose of this repo is to document my journey and the processes I use through building my homelab, the homelab will be used for hosting critical services that i used on a daily basis and to hone my skills in deploying secure systems
# Network Diagram Overview

# <img src="Images/Readme-images/Server-Icon.png" width="25" height="25" /> Servers & Hardware



# <img src="Images/Readme-images/Server-Icon.png" width="25" height="25" /> BUNCEPROX01
# <img src="Images/Readme-images/BUNCEPROX-01.jpeg" width="800" height="500" />
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

# <img src="Images/Readme-images/Server-Icon.png" width="25" height="25" /> BUNCEPROX02
# <img src="Images/Readme-images/BUNCEPROX-02.jpeg" width="800" height="500" />
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


# <img src="Images/Readme-images/Server-Icon.png" width="25" height="25" /> BUNCEPROX03
# <img src="Images/Readme-images/BUNCEPROX-03.jpeg" width="800" height="500" />
-Specs:
- CPU: Ryzen 9 5950x (Manual Undervolt)
- RAM: 48GB 3200mhz (32 + 16)
- Mobo: Gigabyte X570 AORUS Elite
- Chassis: 5U Silverstone
- Storage:
- 1 nvme0n1	1000GB

-Description
- This host is my main PC and is used for gaming and accessing the hosts via vscode. 

# <img src="Images/Readme-images/Server-Icon.png" width="25" height="25" /> Main Rack 1
# <img src="Images/Readme-images/LargeDepthServerRack.jpeg" width="800" height="500" />

# <img src="Images/Readme-images/Server-Icon.png" width="25" height="25" /> Main Rack 2
# <img src="Images/Readme-images/SamsonServerRack.jpeg" width="800" height="500" />