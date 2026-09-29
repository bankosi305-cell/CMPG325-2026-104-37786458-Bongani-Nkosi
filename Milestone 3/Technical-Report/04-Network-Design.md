# 4. NETWORK DESIGN

## 4.1 Design Overview

The network follows a **hierarchical three-tier architecture**:

- **Core Layer** — High-speed Layer 3 switching and inter-VLAN routing
- **Distribution Layer** — (Merged with Core in this design for scale)
- **Access Layer** — Departmental switches providing user connectivity

This design balances cost, performance, and manageability — appropriate for a mid-sized municipal network.

## 4.2 Physical Topology

### 4.2.1 Device Inventory

| Device | Model | Quantity | Role |
|--------|-------|----------|------|
| ISP Router | Cisco ISR 4321 | 1 | Simulated ISP uplink |
| Edge Router | Cisco ISR 4321 | 1 | NAT Gateway, routing |
| Core Switch | Cisco 3650-24PS (L3) | 1 | Inter-VLAN routing, DHCP, ACLs |
| Access Switches | Cisco 2960-24TT | 5 | Department access |
| Servers | Server-PT | 3 | App, File, DHCP |
| PCs | PC-PT | 12 | End users |
| CCTV Cameras | PC-PT (simulated cameras) | 2 | CCTV endpoints |

### 4.2.2 Physical Layout

Three-building campus layout:

| Building | Departments | Access Switch |
|----------|-------------|---------------|
| Main Admin Building | Admin, Finance, HR, IT | SW-Admin, SW-Fin, SW-HR |
| Public Services Centre | Public Services | SW-Public |
| CCTV Building | Security cameras | SW-CCTV |
| IT/Server Room | Core infrastructure | (Core-SW, Edge-Router, Servers) |

### 4.2.3 Cabling

| Link Type | Medium | Speed |
|-----------|--------|-------|
| Core ↔ Edge Router | Copper Straight-Through | 1 Gbps |
| Core ↔ Access Switches | Copper Cross-Over | 100 Mbps |
| Access ↔ End Devices | Copper Straight-Through | 100 Mbps |
| Router ↔ ISP Router | Copper Straight-Through | 1 Gbps |

### 4.2.4 Physical Diagram

![Physical Topology](image.png)

---

## 4.3 Logical Topology

### 4.3.1 VLAN Design

| VLAN | Name | Subnet | Gateway | Purpose |
|------|------|--------|---------|---------|
| 10 | ADMIN | 10.40.1.0/24 | 10.40.1.254 | Administration department |
| 20 | FINANCE | 10.40.2.0/24 | 10.40.2.254 | Finance department |
| 30 | HR | 10.40.3.0/24 | 10.40.3.254 | Human Resources |
| 40 | PUBLIC | 10.40.4.0/24 | 10.40.4.254 | Public Services |
| 50 | IT | 10.40.5.0/24 | 10.40.5.254 | IT Department (reserved) |
| 100 | CCTV | 10.40.10.0/24 | 10.40.10.254 | CCTV (Segmented) |
| 200 | SERVERS | 10.40.20.0/28 | 10.40.20.14 | Server farm |
| 999 | MANAGEMENT | 10.40.255.0/28 | 10.40.255.254 | Network management |

**Note:** VLAN 50 (IT) is reserved. In the current implementation, IT PCs are connected via SW-Admin and reside in VLAN 10 (see Section 8 for details and future improvement).

### 4.3.2 VLSM Summary

The assigned block **10.40.0.0/16** (65,536 addresses) is divided using VLSM:

| Subnet | Range | Mask | Size | Usage |
|--------|-------|------|------|-------|
| 10.40.1.0/24 | .1.1 - .1.254 | /24 | 254 | Admin |
| 10.40.2.0/24 | .2.1 - .2.254 | /24 | 254 | Finance |
| 10.40.3.0/24 | .3.1 - .3.254 | /24 | 254 | HR |
| 10.40.4.0/24 | .4.1 - .4.254 | /24 | 254 | Public |
| 10.40.5.0/24 | .5.1 - .5.254 | /24 | 254 | IT |
| 10.40.10.0/24 | .10.1 - .10.254 | /24 | 254 | CCTV |
| 10.40.20.0/28 | .20.1 - .20.14 | /28 | 14 | Servers |
| 10.40.255.0/28 | .255.1 - .255.14 | /28 | 14 | Management |
| 10.40.0.0/16 remainder | — | /16 | Large | Reserved for expansion |

### 4.3.3 IP Addressing Plan — Key Devices

| Device | Interface | IP | Mask |
|--------|-----------|-----|------|
| Edge-Router | G0/0/0 (Inside) | 10.40.1.1 | /24 |
| Edge-Router | G0/0/1 (Outside) | 192.168.1.2 | /30 |
| Core-SW | VLAN 10 | 10.40.1.254 | /24 |
| Core-SW | VLAN 20 | 10.40.2.254 | /24 |
| Core-SW | VLAN 30 | 10.40.3.254 | /24 |
| Core-SW | VLAN 40 | 10.40.4.254 | /24 |
| Core-SW | VLAN 50 | 10.40.5.254 | /24 |
| Core-SW | VLAN 100 | 10.40.10.254 | /24 |
| Core-SW | VLAN 200 | 10.40.20.14 | /28 |
| Core-SW | VLAN 999 | 10.40.255.254 | /28 |
| App-Server | NIC | 10.40.20.2 | /28 |
| File-Server | NIC | 10.40.20.3 | /28 |
| DHCP-Server | NIC | 10.40.20.4 | /28 |

### 4.3.4 Logical Diagram

![Logical Topology](image-1.png)

---

## 4.4 Routing Design

### 4.4.1 Routing Method

- **Inter-VLAN routing** performed by Core-SW (Layer 3)
- **Static routing** between Core-SW and Edge-Router
- **Default route** on Edge-Router to ISP

### 4.4.2 Routing Table (Core-SW)

| Route | Type | Next Hop |
|-------|------|----------|
| 10.40.1.0/24 | Connected (Vlan10) | — |
| 10.40.2.0/24 | Connected (Vlan20) | — |
| 10.40.3.0/24 | Connected (Vlan30) | — |
| 10.40.4.0/24 | Connected (Vlan40) | — |
| 10.40.5.0/24 | Connected (Vlan50) | — |
| 10.40.10.0/24 | Connected (Vlan100) | — |
| 10.40.20.0/28 | Connected (Vlan200) | — |
| 10.40.255.240/28 | Connected (Vlan999) | — |
| 0.0.0.0/0 | Static (default) | 10.40.1.1 |

### 4.4.3 Routing Table (Edge-Router)

| Route | Type | Next Hop |
|-------|------|----------|
| 10.40.1.0/24 | Connected | — |
| 192.168.1.0/30 | Connected | — |
| 10.40.0.0/16 | Static | 10.40.1.254 |
| 0.0.0.0/0 | Static (default) | 192.168.1.1 |

---

## 4.5 NAT Design

### 4.5.1 NAT Types Used

| Type | Purpose |
|------|---------|
| **PAT (Overload)** | Many office PCs → single public IP (192.168.1.2) |
| **Static NAT** | Server public mappings (192.168.1.3, 192.168.1.4) |

### 4.5.2 NAT Mappings

| Inside Local | Inside Global | Purpose |
|--------------|---------------|---------|
| 10.40.1.0/24 | 192.168.1.2 | PAT (dynamic) |
| 10.40.2.0/24 | 192.168.1.2 | PAT (dynamic) |
| 10.40.3.0/24 | 192.168.1.2 | PAT (dynamic) |
| 10.40.4.0/24 | 192.168.1.2 | PAT (dynamic) |
| 10.40.5.0/24 | 192.168.1.2 | PAT (dynamic) |
| 10.40.20.2 | 192.168.1.3 | Static (App Server) |
| 10.40.20.3 | 192.168.1.4 | Static (File Server) |
| **10.40.10.0/24** | **NONE (excluded)** | CCTV excluded by design |

### 4.5.3 NAT Interfaces

| Interface | Role |
|-----------|------|
| Edge-Router G0/0/0 | `ip nat inside` |
| Edge-Router G0/0/1 | `ip nat outside` |

---

## 4.6 Security Design

### 4.6.1 Security Layering

| Layer | Control |
|-------|---------|
| VLAN segmentation | Isolates departments |
| ACL 101 | Blocks CCTV → office traffic |
| ACL 102 | Restricts server access to Admin/Finance/IT |
| NAT exclusion | CCTV has no internet access |
| Password protection | `enable secret cisco123` on all network devices |

### 4.6.2 ACL Design

**ACL 101 — CCTV Segmentation**
Extended IP access list 101
10 deny ip 10.40.10.0 0.0.0.255 10.40.0.0 0.0.255.255
20 permit ip any any

Applied to: `interface vlan 100` (in)

**ACL 102 — CR4 Server Access**
Extended IP access list 102
10 permit ip 10.40.1.0 0.0.0.255 10.40.20.0 0.0.0.15
20 permit ip 10.40.2.0 0.0.0.255 10.40.20.0 0.0.0.15
30 permit ip 10.40.5.0 0.0.0.255 10.40.20.0 0.0.0.15
40 permit ip 10.40.20.0 0.0.0.15 10.40.1.0 0.0.0.255
50 permit ip 10.40.20.0 0.0.0.15 10.40.2.0 0.0.0.255
60 permit ip 10.40.20.0 0.0.0.15 10.40.5.0 0.0.0.255
70 permit ip 10.40.20.0 0.0.0.15 10.40.20.0 0.0.0.15
80 deny ip any any

Applied to: `interface vlan 200` (in)

**Why bidirectional rules?**
Extended ACLs are **stateless** — they don't track connections. To allow both requests and replies, both directions must be explicitly permitted. This is a deliberate design decision documented in Section 8.

---

## 4.7 Services Design

### 4.7.1 DHCP

Centralised on Core-SW, providing IPs for VLANs 10, 20, 30, 40, 50.

| Pool | Network | Gateway | DNS |
|------|---------|---------|-----|
| VLAN10_POOL | 10.40.1.0/24 | 10.40.1.254 | 8.8.8.8 |
| VLAN20_POOL | 10.40.2.0/24 | 10.40.2.254 | 8.8.8.8 |
| VLAN30_POOL | 10.40.3.0/24 | 10.40.3.254 | 8.8.8.8 |
| VLAN40_POOL | 10.40.4.0/24 | 10.40.4.254 | 8.8.8.8 |
| VLAN50_POOL | 10.40.5.0/24 | 10.40.5.254 | 8.8.8.8 |

### 4.7.2 Servers

| Server | IP | Service |
|--------|-----|---------|
| App-Server | 10.40.20.2 | HTTP/HTTPS |
| File-Server | 10.40.20.3 | FTP |
| DHCP-Server | 10.40.20.4 | (Backup / repurposed) |

---

## 4.8 Design Decisions Summary

| Decision | Rationale |
|----------|-----------|
| L3 Core-SW for routing | Centralised, scalable, cost-effective |
| DHCP on Core-SW | Simplicity, no relay complexity |
| Servers in dedicated VLAN | Precise ACL control for CR4 |
| Bidirectional ACL 102 | Stateless ACLs require explicit return permits |
| CCTV VLAN excluded from NAT ACL | Enforces no-internet security requirement |
| Static IPs for servers | Reliable NAT mappings + service hosting |

---

*End of Section 4*