# Switch Security
  

The main issue I've identified with my homelab is that updating the firewall causes me to lose access to the rest of the network and the internet. To resolve this, I've decided to deploy BUNCE-FW-02 as a secondary firewall gateway and configure both in High Availability mode.
  

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Current unsecure configuration
  
  ```bash
  Building configuration...

Current configuration : 5880 bytes
!
! Last configuration change at 14:12:05 UTC Tue Sep 8 2026
! NVRAM config last updated at 12:10:44 UTC Mon Sep 7 2026
!
version 15.2
no service pad
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname
!
boot-start-marker
boot-end-marker
!
enable secret 9 
!
username privilege 15 secret 9 
no aaa new-model
switch 1 provision ws-c2960x-48ts-l
!
!
!
!
!
!
ip domain-name
!
!
!
!
!
!
!
crypto pki trustpoint TP-self-signed-3458041600
 enrollment selfsigned
 subject-name cn=IOS-Self-Signed-Certificate-3458041600
 revocation-check none
 rsakeypair TP-self-signed-3458041600
!
!
crypto pki certificate chain TP-self-signed-3458041600
 certificate self-signed 01
  	quit
!
spanning-tree mode rapid-pvst
spanning-tree extend system-id
!
!
!
!
vlan internal allocation policy ascending
!
!
! 
!
!
!
!
!
!
!
!
interface FastEthernet0
 no ip address
 shutdown
!
interface GigabitEthernet1/0/1
 description "Uplink-to-Firewall"
 switchport trunk allowed vlan 3,4,7,9,100
 switchport mode trunk
!
interface GigabitEthernet1/0/2
 description "Uplink"
 switchport trunk allowed vlan 3,4,7,9,100
 switchport mode trunk
!
interface GigabitEthernet1/0/3
!
interface GigabitEthernet1/0/4
!
interface GigabitEthernet1/0/5
!
interface GigabitEthernet1/0/6
!
interface GigabitEthernet1/0/7
!
interface GigabitEthernet1/0/8
!
interface GigabitEthernet1/0/9
!
interface GigabitEthernet1/0/10
!
interface GigabitEthernet1/0/11
!
interface GigabitEthernet1/0/12
 description uplink to BUNCEPROX03
 switchport access vlan 9
 switchport trunk allowed vlan 3,4,7,9,100
 switchport mode access
!
interface GigabitEthernet1/0/13
 description BUNCEPROX-01
 switchport trunk allowed vlan 3,4,7,9,100
 switchport mode trunk
!
interface GigabitEthernet1/0/14
 description BUNCEPROX-02
 switchport trunk allowed vlan 3,4,7,9,100
 switchport mode trunk
!
interface GigabitEthernet1/0/15
!
interface GigabitEthernet1/0/16
 description "BUNCEPROX-02"
 switchport access vlan 3
 switchport mode access
 spanning-tree portfast edge
!
interface GigabitEthernet1/0/17
 description "WERO-DESKTOP-01"
 switchport access vlan 4
 switchport mode access
 spanning-tree portfast edge
!
interface GigabitEthernet1/0/18
 description "BUNCE-ROUTER-01"
 switchport access vlan 100
 switchport trunk allowed vlan 3,4,100
 switchport mode access
!
interface GigabitEthernet1/0/19
 shutdown
!
interface GigabitEthernet1/0/20
 shutdown
!
interface GigabitEthernet1/0/21
 shutdown
!
interface GigabitEthernet1/0/22
 shutdown
!
interface GigabitEthernet1/0/23
 shutdown
!
interface GigabitEthernet1/0/24
 shutdown
!
interface GigabitEthernet1/0/25
 shutdown
!
interface GigabitEthernet1/0/26
 shutdown
!
interface GigabitEthernet1/0/27
 shutdown
!
interface GigabitEthernet1/0/28
 shutdown
!
interface GigabitEthernet1/0/29
 shutdown
!
interface GigabitEthernet1/0/30
 shutdown
!
interface GigabitEthernet1/0/31
 shutdown
!
interface GigabitEthernet1/0/32
 shutdown
!
interface GigabitEthernet1/0/33
 shutdown
!
interface GigabitEthernet1/0/34
 shutdown
!
interface GigabitEthernet1/0/35
 shutdown
!
interface GigabitEthernet1/0/36
 shutdown
!
interface GigabitEthernet1/0/37
 shutdown
!
interface GigabitEthernet1/0/38
 shutdown
!
interface GigabitEthernet1/0/39
 shutdown
!
interface GigabitEthernet1/0/40
 shutdown
!
interface GigabitEthernet1/0/41
 shutdown
!
interface GigabitEthernet1/0/42
 shutdown
!
interface GigabitEthernet1/0/43
 shutdown
!
interface GigabitEthernet1/0/44
 shutdown
!
interface GigabitEthernet1/0/45
 shutdown
!
interface GigabitEthernet1/0/46
 shutdown
!
interface GigabitEthernet1/0/47
 shutdown
!
interface GigabitEthernet1/0/48
 shutdown
!
interface GigabitEthernet1/0/49
 shutdown
!
interface GigabitEthernet1/0/50
 shutdown
!
interface GigabitEthernet1/0/51
 shutdown
!
interface GigabitEthernet1/0/52
!
interface Vlan1
 ip dhcp client client-id ascii cisco-28c7.ce1d.8740-Vl1
 ip address dhcp
!
interface Vlan3
 ip address 10.10.3.62 255.255.255.0
!
interface Vlan4
 ip address 10.10.4.18 255.255.255.0
!
ip default-gateway 10.10.3.1
!
ip http server
ip http banner
ip http secure-server
ip ssh time-out 90
ip ssh authentication-retries 2
ip ssh version 2
!
logging trap debugging
logging origin-id string BUNCE-SW-01
logging host 10.10.4.32
!
!
!
line con 0
line vty 0 4
 login local
 transport input ssh
line vty 5 15
 login local
 transport input ssh
!
!
end
```

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Issues identified
  

The list of issues I have identified within this configuration is the following:

http is enabled
energy wise is enabled
management on default vlan 1 port
no negotiate set on the trunks
no port security
fixing misconfigurations


# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> http is enabled

There is many ways you can fix this:

1. Serial 
    plugin the serial cable to the switch
    enter password
    ```bash
    enable
    ```
    ```bash
    configure terminal
    ```
    ```bash
    no ip http server
    ```
2. Webui
    go to the webui address
    go to general settings -> management -> turn off HTTP access 
3. Webui Configuration
    go to the webui address
    go to general settings -> system -> config file -> transfer from switch -> Src/Dest (local hard drive)
    open it once downloaded 
    find the http server line and remove it 
    reupload to switch

do the same for enerywise 

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> management on default vlan 1 port








# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Intial fixes
  
  

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Final secured configuration
  
