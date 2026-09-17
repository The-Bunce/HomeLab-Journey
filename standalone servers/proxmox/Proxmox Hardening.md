# Proxmox Security Hardening

My Homelab has currently not under gone hardening of the proxmox hosts, I intend to follow best practise and will implement the following .

## Table of Contents

1. Unattended-upgrades enabled
2. SSH: no root, no passwords, key-only, AllowUsers
3. Proxmox firewall: default DROP inbound, only mgmt IPs allowed
4. root + default admin locked
5. 2FA on all admin users
6. Unnecessary services disabled / removed
7. NTP via chrony verified
8. Backup jobs configured + tested restore
9. VM/LXC firewalls enabled, VLANs for segmentation	
10. Alerts / monitoring in place
11. pve-audit.log shipping or reviewed regularly
12. Web UI on HTTPS, bound to mgmt network only

* * *

## Unattended-upgrades enabled

Keeping the system up to date is one of the key pillars to securing Proxmox
to do this 
go to Proxmox -> Node (mine is BUNCEPROX01) -> shell -> then type the following 
```bash
apt install -y unattended-upgrades
```
if you want this to be done automatically add the following line 
```bash
dpkg-reconfigure -plow unattended-upgrades
```


## SSH Hardening

create admin account 
first in the shell 
adduser username 
enter password 
This is required before we disable root access 

Now we can create the new SSH key 
on the client end device run the following 
windows 
```bash
ssh-keygen -t ed25519 -f "$env:USERPROFILE\.ssh\id_ed25519" -C "username@host
```

once done you want to add the private key to the proxmox authorized key section 
.ssh/authorized_keys
then add the private key in here 
you now can ssh into the server from the admin account 


First we want to backup the current configuration 
```bash
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```
```bash
nano /etc/ssh/sshd_config
``` 


give PVEAdmin Permissions to / 
The following security issues were found in the original configuration:
| # | Issue | Risk |
| --- | --- | --- |
| 1 | HTTP server enabled (`ip http server`) | Plaintext web management exposes credentials and config over the network |
| 2 | Management interface on default VLAN 1 | VLAN 1 is non-negotiable and a common targeting vector in VLAN-hopping attacks |
| 3 | Trunk ports using DTP negotiation | Allows an attacker on the other end to negotiate an unexpected trunk |
| 4 | Native VLAN left at default (1) | Untagged frames on trunks are implicitly mapped to VLAN 1, enabling double-tagging attacks |
| 5 | No port security on access ports | Any MAC can be plugged in and assume an existing identity (MAC spoofing / rogue switch) |

* * *

## HTTP Server Enabled

> **Goal:** Disable unencrypted HTTP management and rely on HTTPS / SSH only.

There are several ways to apply this change:

### Option 1 – Serial Console (preferred, no network dependency)

1.  Connect a serial cable to the switch console port.
    
2.  Open a terminal session at the appropriate baud rate.
    
3.  Log in, then apply the fix:
    

```bash
enable
configure terminal
no ip http server
end
write memory
```

### Option 2 – Web UI (if you can still reach the switch)

1.  Navigate to the switch's management IP in a browser.
    
2.  Go to **General Settings → Management** and disable **HTTP Access**.
    

### Option 3 – Edit the Config File Directly

1.  Navigate to the switch's management IP.
    
2.  Go to **General Settings → System → Config File → Transfer from Switch** and download the running config to your local machine.
    
3.  Open the file in a text editor, locate `ip http server`, and change it to `no ip http server`.
    
4.  Re-upload the modified file to the switch.
    

### Fallback – USB Method (if re-upload over network fails)

1.  Save the corrected config file to a **FAT32-formatted USB drive**.
    
2.  Plug the USB into the switch.
    
3.  SSH into the switch (older IOS may require legacy key-exchange algorithms):
    

```bash
ssh -oKexAlgorithms=+diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha1 \
    -oHostKeyAlgorithms=+ssh-rsa \
    -oPubkeyAcceptedAlgorithms=+ssh-rsa \
    user@10.10.3.62
```

4.  Back up the current running config just in case:
    

```bash
copy running-config usbflash1:backup.cfg
```

5.  Load the new configuration:
    

```bash
copy usbflash1:your_new_config.cfg system:running-config
reload
```

* * *

## Management on Default VLAN 1

The original config assigned an IP address to the VLAN 1 SVI for management. Because VLAN 1 cannot be removed and is a default in every Cisco deployment, it is a high-value target for attackers.
**Fix:** Remove the IP from VLAN 1 and move management to a dedicated, non-default VLAN (VLAN 3 in this case).

```bash
interface Vlan1
 no ip address
!
interface Vlan3
 ip address 10.10.3.62 255.255.255.0
```

* * *

## Native VLAN Not Hardened

By default, the native (untagged) VLAN on a trunk is VLAN 1. Any untagged frame arriving on a trunk is implicitly assigned to the native VLAN. Because VLAN 1 is shared, this opens the door to **double-tagging (VLAN-hopping)** attacks.
**Fix:** Assign a dedicated, unused "blackhole" VLAN as the native VLAN on every trunk.

```bash
vlan 999
 name BLACKHOLE
!
interface GigabitEthernet1/0/1
 switchport trunk native vlan 999
```

> **Note:** The native VLAN must match on both ends of a trunk. If the uplink switch (e.g., the firewall / Proxmox host) still uses VLAN 1 as native, you'll get a mismatch. Coordinate the change on the other end as well.

* * *

## No Port Security

No access ports had port-security configured, meaning any number of MAC addresses could be presented on a given port.
**Fix – Access port example (GigabitEthernet1/0/17):**

```bash
interface GigabitEthernet1/0/17
 description "WERO-DESKTOP-01"
 switchport access vlan 4
 switchport mode access
 switchport port-security maximum 1
 switchport port-security violation restrict
 switchport port-security
 spanning-tree portfast edge
```

| Command | Purpose |
| --- | --- |
| `switchport mode access` | Place the port in access mode (single-VLAN membership). |
| `switchport port-security maximum 1` | Allow exactly **one** MAC address on the port. |
| `switchport port-security violation restrict` | On violation (e.g., a 2nd MAC appears), silently drop the offending frames and log the event—no port shutdown. |
| `switchport port-security` | Activate port-security. Without this, the settings above are stored but inert. |
| `spanning-tree portfast edge` | Skip STP listening/learning states so the port reaches _forwarding_ in ~1 s; also prevents the port from participating in BPDU topology changes. |

**Fix – Trunk port example (GigabitEthernet1/0/13):**

```bash
interface GigabitEthernet1/0/13
 description "BUNCEPROX-01"
 switchport trunk allowed vlan 3,4,7,9,100
 switchport trunk native vlan 999
 switchport mode trunk
 switchport nonegotiate
```

| Command | Purpose |
| --- | --- |
| `switchport mode trunk` | Place the port in trunk mode (multi-VLAN). |
| `switchport trunk allowed vlan 3,4,7,9,100` | Prune all other VLANs; only the listed VLANs are carried. |
| `switchport trunk native vlan 999` | Set the untagged (native) VLAN to the blackhole VLAN 999 instead of the default 1. |
| `switchport nonegotiate` | Disable DTP so the port does not negotiate trunking; it stays in the mode you explicitly configured. |

* * *

## Trunk Ports Negotiating via DTP

The original trunk ports had no `switchport nonegotiate` command, meaning DTP was active. An attacker who can reach a port on the opposite side of a trunk could use DTP to coax the switch into trunking a port that should remain an access port.
Adding `switchport nonegotiate` to every trunk eliminates this vector (see the trunk example above).

* * *

## SSH Note

Older Cisco IOS 15(2) images may reject modern SSH clients that no longer offer legacy key-exchange / host-key algorithms. If you get a connection refused, fall back to:

```bash
ssh -oKexAlgorithms=+diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha1 \
    -oHostKeyAlgorithms=+ssh-rsa \
    -oPubkeyAcceptedAlgorithms=+ssh-rsa \
    username@ip
```

* * *

## Final Secured Configuration

```bash
version 15.2
no service pad
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname BUNCE-SW-01
!
boot-start-marker
boot-end-marker
!
enable secret 9 redacted
!
username privilege 15 secret 9 redacted
no aaa new-model
switch 1 provision ws-c2960x-48ts-l
!
!
!
!
!
!
ip domain-name redacted
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
 description "Uplink-to-BUNCEPROX-01"
 switchport trunk allowed vlan 3,4,7,9,100
 switchport trunk native vlan 999
 switchport mode trunk
 switchport nonegotiate
!
interface GigabitEthernet1/0/2
 description "Uplink-to-BUNCEPROX-02"
 switchport trunk allowed vlan 3,4,7,9,100
 switchport trunk native vlan 999
 switchport mode trunk
 switchport nonegotiate
!
interface GigabitEthernet1/0/3
 shutdown
!
interface GigabitEthernet1/0/4
 shutdown
!
interface GigabitEthernet1/0/5
 shutdown
!
interface GigabitEthernet1/0/6
 shutdown
!
interface GigabitEthernet1/0/7
 shutdown
!
interface GigabitEthernet1/0/8
 shutdown
!
interface GigabitEthernet1/0/9
 shutdown
!
interface GigabitEthernet1/0/10
 shutdown
!
interface GigabitEthernet1/0/11
 shutdown
!
interface GigabitEthernet1/0/12
 shutdown
!
interface GigabitEthernet1/0/13
 description "BUNCEPROX-01"
 switchport trunk allowed vlan 3,4,7,9,100
 switchport trunk native vlan 999
 switchport mode trunk
 switchport nonegotiate
!
interface GigabitEthernet1/0/14
 description "BUNCEPROX-02"
 switchport trunk allowed vlan 3,4,7,9,100
 switchport trunk native vlan 999
 switchport mode trunk
 switchport nonegotiate
!
interface GigabitEthernet1/0/15
 shutdown
!
interface GigabitEthernet1/0/16
 description "BUNCE-DESKTOP-01"
 switchport access vlan 3
 switchport mode access
 switchport port-security violation restrict
 switchport port-security maximum 1
 switchport port-security
 spanning-tree portfast edge
!
interface GigabitEthernet1/0/17
 description "WERO-DESKTOP-01"
 switchport access vlan 4
 switchport mode access
 switchport port-security violation restrict
 switchport port-security maximum 1
 switchport port-security
 spanning-tree portfast edge
!
interface GigabitEthernet1/0/18
 description "BUNCE-ROUTER-01"
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
 description "Living-Room"
 switchport access vlan 4
 switchport mode access
 switchport port-security violation restrict
 switchport port-security
 spanning-tree portfast edge
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
 no ip address
!
interface Vlan3
 ip address 10.10.3.62 255.255.255.0
!
interface Vlan4
 no ip address
!
interface Vlan7
 no ip address
!
interface Vlan9
 no ip address
!
interface Vlan100
 no ip address
!
interface Vlan999
 no ip address
!
ip default-gateway 10.10.3.1
!
no ip http server
ip http secure-server
ip ssh time-out 90
ip ssh authentication-retries 5
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

* * *

## Key Changes Summary

| Change | Before | After |
| --- | --- | --- |
| HTTP | `ip http server` | `no ip http server` |
| Management VLAN | VLAN 1 (DHCP) | VLAN 3 (static `10.10.3.62/24`) |
| Trunk native VLAN | Default (1) | `999` (BLACKHOLE) |
| Trunk DTP | Enabled (negotiating) | `switchport nonegotiate` |
| Access-port security | None | Port-security, max 1 MAC, restrict |
| Unused ports | Left in default state | Explicitly `shutdown` |