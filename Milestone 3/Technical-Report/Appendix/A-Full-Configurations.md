# APPENDIX A — FULL DEVICE CONFIGURATIONS

## A.1 Overview

This appendix contains the complete `show running-config` output from each device in the network. These configurations were captured after the network was fully tested and verified.

**Device list:**
- A.2 — ISP-Router
- A.3 — Edge-Router
- A.4 — Core-SW
- A.5 — SW-Admin
- A.6 — SW-Fin
- A.7 — SW-HR
- A.8 — SW-Public
- A.9 — SW-CCTV
- A.10 — Servers (Static IP Configuration)

---

## A.2 ISP-Router
hostname ISP-Router
!
interface GigabitEthernet0/0/0
ip address 192.168.1.1 255.255.255.252
no shutdown
!
interface GigabitEthernet0/0/1
ip address 8.8.8.1 255.255.255.0
no shutdown
!
end


---

## A.3 Edge-Router
hostname Edge-Router
enable secret cisco123
service password-encryption
!
interface GigabitEthernet0/0/0
description INSIDE-LAN
ip address 10.40.1.1 255.255.255.0
ip nat inside
no shutdown
!
interface GigabitEthernet0/0/1
description OUTSIDE-ISP
ip address 192.168.1.2 255.255.255.252
ip nat outside
no shutdown
!
ip route 0.0.0.0 0.0.0.0 192.168.1.1
ip route 10.40.0.0 255.255.0.0 10.40.1.254
!
access-list 1 permit 10.40.1.0 0.0.0.255
access-list 1 permit 10.40.2.0 0.0.0.255
access-list 1 permit 10.40.3.0 0.0.0.255
access-list 1 permit 10.40.4.0 0.0.0.255
access-list 1 permit 10.40.5.0 0.0.0.255
access-list 1 permit 10.40.20.0 0.0.0.15
!
ip nat inside source list 1 interface GigabitEthernet0/0/1 overload
ip nat inside source static 10.40.20.2 192.168.1.3
ip nat inside source static 10.40.20.3 192.168.1.4
!
end


---

## A.4 Core-SW
hostname Core-SW
enable secret cisco123
service password-encryption
!
ip routing
!
vlan 10
name ADMIN
vlan 20
name FINANCE
vlan 30
name HR
vlan 40
name PUBLIC
vlan 50
name IT
vlan 100
name CCTV
vlan 200
name SERVERS
vlan 999
name MANAGEMENT
!
interface FastEthernet0/1
switchport mode access
switchport access vlan 200
!
interface FastEthernet0/2
switchport mode access
switchport access vlan 200
!
interface FastEthernet0/3
switchport mode access
switchport access vlan 200
!
interface FastEthernet0/4
switchport mode access
switchport access vlan 200
!
interface FastEthernet0/5
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,50,200
!
interface FastEthernet0/6
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 20
!
interface FastEthernet0/7
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 30
!
interface FastEthernet0/8
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 40
!
interface FastEthernet0/9
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 100
!
interface FastEthernet0/10
switchport mode access
switchport access vlan 200
!
interface GigabitEthernet0/1
switchport mode access
switchport access vlan 10
!
interface Vlan10
ip address 10.40.1.254 255.255.255.0
no shutdown
!
interface Vlan20
ip address 10.40.2.254 255.255.255.0
no shutdown
!
interface Vlan30
ip address 10.40.3.254 255.255.255.0
no shutdown
!
interface Vlan40
ip address 10.40.4.254 255.255.255.0
no shutdown
!
interface Vlan50
ip address 10.40.5.254 255.255.255.0
no shutdown
!
interface Vlan100
ip address 10.40.10.254 255.255.255.0
ip access-group 101 in
no shutdown
!
interface Vlan200
ip address 10.40.20.14 255.255.255.240
ip access-group 102 in
no shutdown
!
interface Vlan999
ip address 10.40.255.254 255.255.255.240
no shutdown
!
ip dhcp pool VLAN10_POOL
network 10.40.1.0 255.255.255.0
default-router 10.40.1.254
dns-server 8.8.8.8
!
ip dhcp pool VLAN20_POOL
network 10.40.2.0 255.255.255.0
default-router 10.40.2.254
dns-server 8.8.8.8
!
ip dhcp pool VLAN30_POOL
network 10.40.3.0 255.255.255.0
default-router 10.40.3.254
dns-server 8.8.8.8
!
ip dhcp pool VLAN40_POOL
network 10.40.4.0 255.255.255.0
default-router 10.40.4.254
dns-server 8.8.8.8
!
ip dhcp pool VLAN50_POOL
network 10.40.5.0 255.255.255.0
default-router 10.40.5.254
dns-server 8.8.8.8
!
ip route 0.0.0.0 0.0.0.0 10.40.1.1
!
access-list 101 deny ip 10.40.10.0 0.0.0.255 10.40.0.0 0.0.255.255
access-list 101 permit ip any any
access-list 102 permit ip 10.40.1.0 0.0.0.255 10.40.20.0 0.0.0.15
access-list 102 permit ip 10.40.2.0 0.0.0.255 10.40.20.0 0.0.0.15
access-list 102 permit ip 10.40.5.0 0.0.0.255 10.40.20.0 0.0.0.15
access-list 102 permit ip 10.40.20.0 0.0.0.15 10.40.1.0 0.0.0.255
access-list 102 permit ip 10.40.20.0 0.0.0.15 10.40.2.0 0.0.0.255
access-list 102 permit ip 10.40.20.0 0.0.0.15 10.40.5.0 0.0.0.255
access-list 102 permit ip 10.40.20.0 0.0.0.15 10.40.20.0 0.0.0.15
access-list 102 deny ip any any
!
end


---

## A.5 SW-Admin
hostname SW-Admin
!
vlan 10
name ADMIN
vlan 20
name FINANCE
vlan 30
name HR
vlan 50
name IT
vlan 200
name SERVERS
!
interface FastEthernet0/1
switchport mode access
switchport access vlan 10
spanning-tree portfast
!
interface FastEthernet0/2
switchport mode access
switchport access vlan 10
spanning-tree portfast
!
interface FastEthernet0/3
switchport mode access
switchport access vlan 10
spanning-tree portfast
!
interface FastEthernet0/4
switchport mode access
switchport access vlan 10
spanning-tree portfast
!
interface FastEthernet0/5
switchport mode access
switchport access vlan 10
spanning-tree portfast
!
interface GigabitEthernet0/1
switchport mode trunk
switchport trunk allowed vlan 10,20,30,50,200
!
end


---

## A.6 SW-Fin
hostname SW-Fin
!
vlan 20
name FINANCE
!
interface FastEthernet0/1
switchport mode trunk
switchport trunk allowed vlan 20
switchport trunk native vlan 1
!
interface FastEthernet0/2
switchport mode access
switchport access vlan 20
spanning-tree portfast
!
interface FastEthernet0/3
switchport mode access
switchport access vlan 20
spanning-tree portfast
!
interface FastEthernet0/4
switchport mode access
switchport access vlan 20
spanning-tree portfast
!
end


---

## A.7 SW-HR
hostname SW-HR
!
vlan 30
name HR
!
interface FastEthernet0/1
switchport mode trunk
switchport trunk allowed vlan 30
switchport trunk native vlan 1
!
interface range FastEthernet0/2-4
switchport mode access
switchport access vlan 30
spanning-tree portfast
!
end


---

## A.8 SW-Public
hostname SW-Public
!
vlan 40
name PUBLIC
!
interface FastEthernet0/1
switchport mode trunk
switchport trunk allowed vlan 40
switchport trunk native vlan 1
!
interface range FastEthernet0/2-4
switchport mode access
switchport access vlan 40
spanning-tree portfast
!
end


---

## A.9 SW-CCTV
hostname SW-CCTV
!
vlan 100
name CCTV
!
interface FastEthernet0/1
switchport mode trunk
switchport trunk allowed vlan 100
switchport trunk native vlan 1
!
interface range FastEthernet0/2-4
switchport mode access
switchport access vlan 100
spanning-tree portfast
!
end


---

## A.10 Server Static IP Configuration

### App-Server
IPv4 Address: 10.40.20.2
Subnet Mask: 255.255.255.240
Default Gateway: 10.40.20.14
DNS Server: 8.8.8.8
HTTP/HTTPS: Enabled


### File-Server
IPv4 Address: 10.40.20.3
Subnet Mask: 255.255.255.240
Default Gateway: 10.40.20.14
FTP: Enabled (admin / admin123)


### DHCP-Server
IPv4 Address: 10.40.20.4
Subnet Mask: 255.255.255.240
Default Gateway: 10.40.20.14
DHCP Service: Disabled (Core-SW handles DHCP)


---

*End of Appendix A*