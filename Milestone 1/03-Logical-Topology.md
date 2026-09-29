# LOGICAL TOPOLOGY
## CMPG325-2026-104 | Bongani Nkosi | 37786458

---

## 1. VLAN DESIGN

| VLAN ID | VLAN Name   | Subnet          | Gateway       | Purpose           |
|---------|-------------|-----------------|---------------|-------------------|
| 10      | ADMIN       | 10.40.1.0/24    | 10.40.1.254   | Administration    |
| 20      | FINANCE     | 10.40.2.0/24    | 10.40.2.254   | Finance Dept      |
| 30      | HR          | 10.40.3.0/24    | 10.40.3.254   | HR Dept           |
| 40      | PUBLIC      | 10.40.4.0/24    | 10.40.4.254   | Public Services   |
| 50      | IT          | 10.40.5.0/24    | 10.40.5.254   | IT Dept           |
| 100     | CCTV        | 10.40.10.0/24   | 10.40.10.254  | CCTV (Segmented)  |
| 200     | SERVERS     | 10.40.20.0/28   | 10.40.20.254  | Server Farm       |
| 999     | MANAGEMENT  | 10.40.255.0/28  | 10.40.255.254 | Network Mgmt      |

---

## 2. ROUTING DESIGN

| Device      | Routing Type    | Routes                      |
|-------------|-----------------|-----------------------------|
| Edge Router | Static Default  | 0.0.0.0/0 → 192.168.1.1     |
| Edge Router | Static          | 10.40.0.0/16 → 10.40.1.254  |
| Core Switch | Static Default  | 0.0.0.0/0 → 10.40.1.1       |

---

## 3. SECURITY ACLs

### ACL 101: CCTV Segmentation
```cisco
access-list 101 deny ip 10.40.10.0 0.0.0.255 10.40.0.0 0.0.255.255
access-list 101 permit ip any any