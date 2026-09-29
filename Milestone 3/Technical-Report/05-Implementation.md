# 5. IMPLEMENTATION

## 5.1 Implementation Overview

All devices were configured in **Cisco Packet Tracer 8.3** using IOS CLI commands. This section documents the key configurations applied to each device in the network.

## 5.2 Device Configuration Summary

| Device | Hostname | Key Configurations |
|--------|----------|--------------------|
| Edge Router | Edge-Router | NAT (PAT + Static), routing, NAT inside/outside interfaces |
| Core Switch | Core-SW | VLANs, SVIs, inter-VLAN routing, DHCP pools, ACLs |
| Access Switches | SW-Admin, SW-Fin, SW-HR, SW-Public, SW-CCTV | VLANs, trunk ports, access ports |
| Servers | App-Server, File-Server, DHCP-Server | Static IPs, service configurations |
| PCs | Multiple | DHCP clients |
| CCTV Cameras | CCTV-Cam1, CCTV-Cam2 | Static IPs in VLAN 100 |

## 5.3 Edge-Router Configuration

The Edge-Router performs the **NAT (Inside/Outside Address Translation)** — the assigned challenge for this project.

### 5.3.1 Interfaces

interface GigabitEthernet0/0/0
description INSIDE-LAN
ip address 10.40.1.1 255.255.255.0
ip nat inside
no shutdown

interface GigabitEthernet0/0/1
description OUTSIDE-ISP
ip address 192.168.1.2 255.255.255.252
ip nat outside
no shutdown


### 5.3.2 Routing

ip route 0.0.0.0 0.0.0.0 192.168.1.1
ip route 10.40.0.0 255.255.0.0 10.40.1.254


### 5.3.3 NAT Configuration

**ACL for NAT (excludes CCTV):**
access-list 1 permit 10.40.1.0 0.0.0.255
access-list 1 permit 10.40.2.0 0.0.0.255
access-list 1 permit 10.40.3.0 0.0.0.255
access-list 1 permit 10.40.4.0 0.0.0.255
access-list 1 permit 10.40.5.0 0.0.0.255
access-list 1 permit 10.40.20.0 0.0.0.15


**PAT (Overload):**
ip nat inside source list 1 interface GigabitEthernet0/0/1 overload


**Static NAT (CR4 Servers):**
ip nat inside source static 10.40.20.2 192.168.1.3
ip nat inside source static 10.40.20.3 192.168.1.4


## 5.4 Core-SW Configuration

The Core-SW is the heart of the network — providing inter-VLAN routing, DHCP, and ACL enforcement.

### 5.4.1 IP Routing Enabled
ip routing


### 5.4.2 VLANs Created
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


### 5.4.3 SVIs (Gateways)
interface Vlan10
ip address 10.40.1.254 255.255.255.0
no shutdown

interface Vlan20
ip address 10.40.2.254 255.255.255.0
no shutdown

interface Vlan30
ip address 10.40.3.254 255.255.255.0
no shutdown

interface Vlan40
ip address 10.40.4.254 255.255.255.0
no shutdown

interface Vlan50
ip address 10.40.5.254 255.255.255.0
no shutdown

interface Vlan100
ip address 10.40.10.254 255.255.255.0
no shutdown

interface Vlan200
ip address 10.40.20.14 255.255.255.240
no shutdown

interface Vlan999
ip address 10.40.255.254 255.255.255.240
no shutdown


### 5.4.4 Trunk Ports to Access Switches
interface FastEthernet0/5
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,50,200

interface FastEthernet0/6
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 20

interface FastEthernet0/7
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 30

interface FastEthernet0/8
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 40

interface FastEthernet0/9
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 100


### 5.4.5 DHCP Pools
ip dhcp pool VLAN10_POOL
network 10.40.1.0 255.255.255.0
default-router 10.40.1.254
dns-server 8.8.8.8

ip dhcp pool VLAN20_POOL
network 10.40.2.0 255.255.255.0
default-router 10.40.2.254
dns-server 8.8.8.8

ip dhcp pool VLAN30_POOL
network 10.40.3.0 255.255.255.0
default-router 10.40.3.254
dns-server 8.8.8.8

ip dhcp pool VLAN40_POOL
network 10.40.4.0 255.255.255.0
default-router 10.40.4.254
dns-server 8.8.8.8

ip dhcp pool VLAN50_POOL
network 10.40.5.0 255.255.255.0
default-router 10.40.5.254
dns-server 8.8.8.8


### 5.4.6 Default Route
ip route 0.0.0.0 0.0.0.0 10.40.1.1


### 5.4.7 ACLs

**ACL 101 (CCTV Segmentation):**
access-list 101 deny ip 10.40.10.0 0.0.0.255 10.40.0.0 0.0.255.255
access-list 101 permit ip any any

interface Vlan100
ip access-group 101 in


**ACL 102 (CR4 Server Access):**
access-list 102 permit ip 10.40.1.0 0.0.0.255 10.40.20.0 0.0.0.15
access-list 102 permit ip 10.40.2.0 0.0.0.255 10.40.20.0 0.0.0.15
access-list 102 permit ip 10.40.5.0 0.0.0.255 10.40.20.0 0.0.0.15
access-list 102 permit ip 10.40.20.0 0.0.0.15 10.40.1.0 0.0.0.255
access-list 102 permit ip 10.40.20.0 0.0.0.15 10.40.2.0 0.0.0.255
access-list 102 permit ip 10.40.20.0 0.0.0.15 10.40.5.0 0.0.0.255
access-list 102 permit ip 10.40.20.0 0.0.0.15 10.40.20.0 0.0.0.15
access-list 102 deny ip any any

interface Vlan200
ip access-group 102 in


## 5.5 Access Switch Configuration

Each access switch was configured with:
- A dedicated VLAN for its department
- A trunk port connecting to Core-SW
- Access ports for end devices with `portfast` enabled

### 5.5.1 SW-Admin (Admin department)
vlan 10
name ADMIN

interface FastEthernet0/2
switchport mode access
switchport access vlan 10
spanning-tree portfast

interface FastEthernet0/3
switchport mode access
switchport access vlan 10
spanning-tree portfast

interface FastEthernet0/4
switchport mode access
switchport access vlan 10
spanning-tree portfast

interface FastEthernet0/5
switchport mode access
switchport access vlan 10
spanning-tree portfast

interface GigabitEthernet0/1
switchport mode trunk
switchport trunk allowed vlan 10,20,30,50,200


### 5.5.2 SW-Fin (Finance department)
vlan 20
name FINANCE

interface FastEthernet0/1
switchport mode trunk
switchport trunk allowed vlan 20
switchport trunk native vlan 1

interface FastEthernet0/2
switchport mode access
switchport access vlan 20
spanning-tree portfast

interface FastEthernet0/3
switchport mode access
switchport access vlan 20
spanning-tree portfast

interface FastEthernet0/4
switchport mode access
switchport access vlan 20
spanning-tree portfast


### 5.5.3 SW-HR (Human Resources)
vlan 30
name HR

interface FastEthernet0/1
switchport mode trunk
switchport trunk allowed vlan 30
switchport trunk native vlan 1

interface range FastEthernet0/2-4
switchport mode access
switchport access vlan 30
spanning-tree portfast


### 5.5.4 SW-Public (Public Services)
vlan 40
name PUBLIC

interface FastEthernet0/1
switchport mode trunk
switchport trunk allowed vlan 40
switchport trunk native vlan 1

interface range FastEthernet0/2-4
switchport mode access
switchport access vlan 40
spanning-tree portfast


### 5.5.5 SW-CCTV (CCTV — Segmented)
vlan 100
name CCTV

interface FastEthernet0/1
switchport mode trunk
switchport trunk allowed vlan 100
switchport trunk native vlan 1

interface range FastEthernet0/2-4
switchport mode access
switchport access vlan 100
spanning-tree portfast


## 5.6 Server Configuration

### 5.6.1 App-Server
IP Address: 10.40.20.2
Subnet Mask: 255.255.255.240
Default Gateway: 10.40.20.14
DNS Server: 8.8.8.8
HTTP/HTTPS Services: Enabled


### 5.6.2 File-Server
IP Address: 10.40.20.3
Subnet Mask: 255.255.255.240
Default Gateway: 10.40.20.14
FTP Service: Enabled (user: admin / password: admin123)


### 5.6.3 DHCP-Server
IP Address: 10.40.20.4
Subnet Mask: 255.255.255.240
Default Gateway: 10.40.20.14
DHCP Service: Disabled (Core-SW handles DHCP)


## 5.7 End Device Configuration

### 5.7.1 Office PCs

All office PCs (Admin, Finance, HR, Public Services, IT) are configured as **DHCP clients**.

Expected IP ranges:

| Department | VLAN | Expected Range |
|------------|------|----------------|
| Admin | 10 | 10.40.1.x |
| Finance | 20 | 10.40.2.x |
| HR | 30 | 10.40.3.x |
| Public | 40 | 10.40.4.x |
| IT | 10 | 10.40.1.x |

### 5.7.2 CCTV Cameras

CCTV cameras are configured with **static IPs** in VLAN 100:

| Device | IP | Mask | Gateway |
|--------|-----|------|---------|
| CCTV-Cam1 | 10.40.10.10 | 255.255.255.0 | 10.40.10.254 |
| CCTV-Cam2 | 10.40.10.11 | 255.255.255.0 | 10.40.10.254 |

## 5.8 Implementation Verification

After all configurations were applied, the following commands were used to verify correct operation:

| Command | Purpose |
|---------|---------|
| `show ip interface brief` | Verify IP addresses on all interfaces |
| `show vlan brief` | Verify VLANs and port assignments |
| `show interfaces trunk` | Verify trunk ports |
| `show ip route` | Verify routing tables |
| `show ip nat translations` | Verify NAT mappings |
| `show ip nat statistics` | Verify NAT operation |
| `show ip access-lists` | Verify ACL entries and match counts |
| `show ip dhcp binding` | Verify DHCP leases |
| `show mac address-table` | Verify MAC learning |

All verifications were successful — see Section 7 for detailed evidence.

---

*End of Section 5*