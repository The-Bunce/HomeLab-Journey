# 🏠 Wazuh Setup

This readme will detail the steps I have taken to deploy Wazuh on a standalone VM. The main reason I have deployed it on a standalone VM rather than within the Docker-Compose stack is to make it easier to control access via OPNsense firewall rules.


## <img src="images/Readme-images/Cogs-Icon.png" width="25" height="25" /> System requirements

please see the link here to see the latest requirements:
https://documentation.wazuh.com/current/installation-guide/wazuh-server/index.html


## <img src="images/Readme-images/Cogs-Icon.png" width="25" height="25" /> Install the wazuh application

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

wait for the install to finish

check it is all running     

systemctl status wazuh-manager wazuh-indexer wazuh-dashboard

check that it is listening 
ss -tlnp | grep -E '5500|9200|443'


## <img src="images/Readme-images/Server-Icon.png" width="25" height="25" /> Adding an agent

login to the web page 
agents management
groups
then you can create all the groups you want
I am creating groups based on their criticality


click deploy an agent 
select OS of the system
server address (wazuh ip)
tick remember server address
agent name (keep it consistent with the firewall names/proxmox name)
wazuh will give you the codes to run per device

for docker you have to 
go to the OSSEC file 
add the following lines 

```bash
  <wodle name="docker-listener">
    <disabled>no</disabled>
    <attempts>5</attempts>
    <interval>10m</interval>
    <run_on_start>yes</run_on_start>
  </wodle>

  <localfile>
    <location>/var/lib/docker/containers/**/*.log</location>
    <log_format>syslog</log_format>
  </localfile>
```

Important note 
To connect a Windows Wazuh agent to your Wazuh manager, you need to allow outbound TCP ports 1514 and 1515 from the Windows endpoint to the Wazuh server.Because the agent initiates the connection outward, you normally do not need to open inbound listening ports on the Windows machine itself, but your network and host firewalls must permit this outbound traffic.

to check that its working on windows pc 
netstat -ano | findstr 1514



Make adjustments to what is logged
nano /var/ossec/etc/ossec.conf



remove agent 
sudo systemctl stop wazuh-agent
sudo systemctl disable --now wazuh-agent
sudo apt-get remove --purge -y wazuh-agent
sudo apt-get purge -y 'wazuh*agent*'
sudo apt-get autoremove -y

systemctl list-units | grep wazuh
ps aux | grep -E 'wazuh|ossec' | grep -v grep


check if you can talk via the ports 
 nc -zv 10.10.3.100 1514

 deploying on proxmox 
 export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
set the path first 
then run the installation script 


changing management ip 
remote into ubuntu 
sudo nano /var/ossec/etc/ossec.conf
change the IP 
sudo systemctl restart wazuh-agent.service

windows 
C:\Program Files (x86)\ossec-agent
copy the OSSEC.conf file 
change ip 
replace 
restart service
open powershell 
NET STOP Wazuh
NET START Wazuh