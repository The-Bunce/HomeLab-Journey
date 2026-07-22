#  Homelab Backup of Vitual Machines
The issue I have identified in my homelab is that if an issue was to occur in one of the virtual machines or one of the virtual machines become corrupted, i would have no way to restore it and would have to restore it manually which would take a long time. To remediate this i will 
# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step By Step Guide

Here is a step by step breakdown on how to get the Proxmox Backup environment up and running

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Information

First step is deciding whether you will be hosting the backup server virtually (on the proxmox virtual environment) or baremetal and installing it via the usb method, i will be deploying it on the proxmox virtual environment as this host will also contain my truenas vm (used for mass storage for my network).

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 1 Creating the VM

First head to the proxmox website and download the ISO file https://www.proxmox.com/en/downloads/proxmox-backup-server
once you have downloaded it head to the proxmox virtual environment
upload the ISO file to the proxmox virtual environment 
now load/create a vm using the ISO file
once the vm is created you can start it and follow the prompts (wizard)#
ensure you have two disks assigned (1 for the OS and 1 for the storage)
# <img src="/Images/Docker-Images/Proxmox-Backup-Server Standup.png" width="400" height="400" />

*** if you run into an issue where there is an error when the installer is finished try installing the older varient then upgrading

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 2 

once you have rebooted the system on the proxmox ve it should like this:

# <img src="/Images/Docker-Images/Proxmox-Backup-Server Reloaded.png" width="400" height="400" />

go to the ip address listed (on your machine you would have specified this during creation)

you will recieve this error: 
# <img src="/Images/Docker-Images/Proxmox-Backup-Server Subscription error .png" width="400" height="400" />

to fix this go to Administration -> Repositories -> disable the enterprise variant 
no click add then select no-subscription 


# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 3 creating the data store

go to storage / disks 
go to disk intialize the other disk
now go to directory and create a directory (name it something you will remember)
# <img src="/Images/Docker-Images/Proxmox-Backup-Server Datastore .png" width="400" height="400" />
once that has been created 
you will see the path it created and see that it is now in the datastore section of the drop down menu
click into it to view the settings page 
you can see content saved 
prune jobs & garabage collection 


# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 4 adding the backup server to the proxmox servers

Create a new user within proxmox backup server (this will be used to allow access from the other proxmox hosts)
# <img src="/Images/Docker-Images/Proxmox-Backup-Server Users.png" width="400" height="400" />
now create an api token for this user by heading to API token tab 
clicking add 
selecting the new user we have created 
adding a token name 
it is important that you copy and save this secret value 


once you have created the user 
go to permissions add the permissions 
    - /datastore & role being DatastoreBackup
this will be for both the user and the api token 

next head to your proxmox host 
go to datacentre -> storage 
click add proxmox backup server
ID: give it a name 
Server: IP of the backup server
username: root@pam
password: password of the user 
Datastore: the datastore mount 

Now go to backup (just under storage)
select all VMs you want to backup 
create a schedule of when you want the backup to occur 
mine will be weekly 
you can choose which mode the backup does if its just a snapshot or full (full requires the VM be powered down)
you can set retention policy aswell 
i will be keeping 1 daily , 1 monthly and 1 weekly 
you can test the back up by clicking run now 
you should now see the backup working and can check to ensure the data is being sent to the PBS server 
you should see data shown under content 


Congratulations you now have a backup server 

