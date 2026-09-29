# APPENDIX B — EVIDENCE SCREENSHOTS

## B.1 Overview

This appendix contains the 12 evidence screenshots captured during Milestone 2 testing. Each screenshot demonstrates a specific tested behaviour of the network.

**Location:** All screenshots are stored in `Milestone-2/Testing/screenshots/`.

---

## B.2 Screenshot 01 — Admin → Finance (Inter-VLAN Routing)

![01-admin-to-finance](image.png)

**Demonstrates:** Inter-VLAN routing between VLAN 10 (Admin) and VLAN 20 (Finance).

**Evidence:**
Admin-PC1: ping 10.40.2.1
Reply from 10.40.2.1: bytes=32 time<1ms TTL=127
Reply from 10.40.2.1: bytes=32 time<1ms TTL=127
Reply from 10.40.2.1: bytes=32 time<1ms TTL=127
Success rate is 75 percent (3/4)


**Conclusion:** Inter-VLAN routing is functional across departments. The first packet is lost due to ARP resolution.

---

## B.3 Screenshot 02 — Admin → App-Server (CR4 Authorised)

![02-admin-to-server](image-1.png)

**Demonstrates:** Admin department reaching the App-Server, proving ACL 102 correctly authorises Admin.

**Evidence:**
Admin-PC1: ping 10.40.20.2
Reply from 10.40.20.2: bytes=32 time<1ms TTL=127
Reply from 10.40.20.2: bytes=32 time<1ms TTL=127
Reply from 10.40.20.2: bytes=32 time<1ms TTL=127
Reply from 10.40.20.2: bytes=32 time<1ms TTL=127
Success rate is 100 percent (4/4)


**Conclusion:** ACL 102 correctly permits Admin (10.40.1.0/24) to reach the server VLAN (10.40.20.0/28).

---

## B.4 Screenshot 03 — HR → App-Server (CR4 Denied)

![03-hr-blocked-server](image-2.png)

**Demonstrates:** HR department correctly blocked from the App-Server, proving ACL 102 enforces CR4 restrictions.

**Evidence:**
HR-PC1: ping 10.40.20.2
Request timed out.
Request timed out.
Request timed out.
Request timed out.
Success rate is 0 percent (0/4)


**Conclusion:** ACL 102 correctly denies HR (10.40.3.0/24) access to the server VLAN.

---

## B.5 Screenshot 04 — HR → Internet (NAT Working)

![04-hr-to-internet](image-3.png)

**Demonstrates:** HR department reaching the ISP via NAT, proving PAT translation works.

**Evidence:**
HR-PC1: ping 192.168.1.1
Reply from 192.168.1.1: bytes=32 time<1ms TTL=253
Reply from 192.168.1.1: bytes=32 time<1ms TTL=253
Reply from 192.168.1.1: bytes=32 time<1ms TTL=253
Success rate is 75 percent (3/4)


**Conclusion:** PAT (Overload) is functioning — HR's private IP was translated to a public IP to reach the ISP.

---

## B.6 Screenshot 05 — CCTV → Admin PC (Segmentation Constraint)

![05-cctv-blocked-admin](image-4.png)

**Demonstrates:** CCTV correctly blocked from office VLANs, proving ACL 101 enforces the segmentation constraint.

**Evidence:**
CCTV-Cam1: ping 10.40.1.10
Request timed out.
Reply from 10.40.10.254: Destination host unreachable.
Reply from 10.40.10.254: Destination host unreachable.
Reply from 10.40.10.254: Destination host unreachable.
Success rate is 0 percent (0/4)


**Conclusion:** ACL 101 correctly denies CCTV (10.40.10.0/24) access to office VLANs. The "Destination host unreachable" reply from the CCTV gateway confirms the ACL is enforcing the block.

---

## B.7 Screenshot 06 — CCTV → CCTV (Internal Communication)

![06-cctv-internal](image-5.png)

**Demonstrates:** CCTV devices communicating within their own VLAN, proving ACL 101 only blocks cross-VLAN traffic.

**Evidence:**
CCTV-Cam1: ping 10.40.10.10
Reply from 10.40.10.10: bytes=32 time<1ms TTL=128
Reply from 10.40.10.10: bytes=32 time<1ms TTL=128
Reply from 10.40.10.10: bytes=32 time<1ms TTL=128
Reply from 10.40.10.10: bytes=32 time<1ms TTL=128
Success rate is 100 percent (4/4)


**Conclusion:** CCTV devices can communicate within their VLAN — the ACL only blocks inter-VLAN traffic.

---

## B.8 Screenshot 07 — CCTV → Internet (No NAT for CCTV)

![07-cctv-no-nat](image-6.png)

**Demonstrates:** CCTV excluded from NAT, proving the segmentation constraint is enforced at the NAT layer.

**Evidence:**
CCTV-Cam1: ping 8.8.8.1
Request timed out.
Request timed out.
Request timed out.
Request timed out.
Success rate is 0 percent (0/4)


**Conclusion:** CCTV traffic is NOT translated by NAT (CCTV subnet excluded from ACL 1 on Edge-Router). CCTV has no internet access — enforced by design.

---

## B.9 Screenshot 08 — NAT Translations

![08-nat-translations](image-7.png)

**Demonstrates:** Static NAT entries for both servers, proving CR4 servers are externally reachable.

**Evidence (Edge-Router):**
show ip nat translations

Pro Inside global Inside local Outside local Outside global
--- 192.168.1.3 10.40.20.2 --- ---
--- 192.168.1.4 10.40.20.3 --- ---


**Conclusion:** Two static NAT entries confirmed — App-Server (10.40.20.2 → 192.168.1.3) and File-Server (10.40.20.3 → 192.168.1.4).

---

## B.10 Screenshot 09 — NAT Statistics

![09-nat-statistics](image-8.png)

**Demonstrates:** NAT operating correctly, with inside/outside interfaces assigned.

**Evidence (Edge-Router):**
show ip nat statistics

Total translations: 2 (2 static, 0 dynamic, 0 extended)
Outside Interfaces: GigabitEthernet0/0/1
Inside Interfaces: GigabitEthernet0/0/0
Hits: 19 Misses: 32
Expired translations: 13


**Conclusion:** NAT is functional — 2 static mappings confirmed, interfaces correctly assigned, and hits indicate active use.

---

## B.11 Screenshot 10 — ACL Verification

![10-acl-verification](image-9.png)

**Demonstrates:** Both ACLs (101 and 102) applied and active on Core-SW.

**Evidence (Core-SW):**
show ip access-lists

Extended IP access list 101
10 deny ip 10.40.10.0 0.0.0.255 10.40.0.0 0.0.255.255
20 permit ip any any

Extended IP access list 102
10 permit ip 10.40.1.0 0.0.0.255 10.40.20.0 0.0.0.15
20 permit ip 10.40.2.0 0.0.0.255 10.40.20.0 0.0.0.15
30 permit ip 10.40.5.0 0.0.0.255 10.40.20.0 0.0.0.15
40 permit ip 10.40.20.0 0.0.0.15 10.40.1.0 0.0.0.255 (4 match(es))
50 permit ip 10.40.20.0 0.0.0.15 10.40.2.0 0.0.0.255
60 permit ip 10.40.20.0 0.0.0.15 10.40.5.0 0.0.0.255
70 permit ip 10.40.20.0 0.0.0.15 10.40.20.0 0.0.0.15
80 deny ip any any


**Conclusion:** Both ACLs correctly configured. Rule 40 (bidirectional Admin ↔ Server) shows 4 matches, confirming active use.

---

## B.12 Screenshot 11 — VLAN Configuration

![11-vlan-brief](image-10.png)

**Demonstrates:** All 8 VLANs active on Core-SW with correct port assignments.

**Evidence (Core-SW):**
show vlan brief

VLAN Name Status Ports

10 ADMIN active Gig0/1
20 FINANCE active
30 HR active
40 PUBLIC active
50 IT active
100 CCTV active
200 SERVERS active Fa0/1, Fa0/2, Fa0/3, Fa0/4, Fa0/10
999 MANAGEMENT active


**Conclusion:** All VLANs configured and active. Server ports correctly assigned to VLAN 200.

---

## B.13 Screenshot 12 — Routing Table

![12-routing-table](image-11.png)

**Demonstrates:** Core-SW routing table showing all VLAN SVIs and default route.

**Evidence (Core-SW):**
show ip route

C 10.40.1.0/24 is directly connected, Vlan10
C 10.40.2.0/24 is directly connected, Vlan20
C 10.40.3.0/24 is directly connected, Vlan30
C 10.40.4.0/24 is directly connected, Vlan40
C 10.40.5.0/24 is directly connected, Vlan50
C 10.40.10.0/24 is directly connected, Vlan100
C 10.40.20.0/28 is directly connected, Vlan200
C 10.40.255.240/28 is directly connected, Vlan999
S* 0.0.0.0/0 [1/0] via 10.40.1.1


**Conclusion:** All VLANs routed; default route to Edge-Router functional.

---

## B.14 Summary of Evidence

| # | Screenshot | Demonstrates | Requirement |
|---|------------|--------------|-------------|
| 1 | 01-admin-to-finance.png | Inter-VLAN routing | Connectivity |
| 2 | 02-admin-to-server.png | CR4 authorised | Change Request CR4 |
| 3 | 03-hr-blocked-server.png | CR4 denied | Change Request CR4 |
| 4 | 04-hr-to-internet.png | NAT (PAT) working | Assigned challenge |
| 5 | 05-cctv-blocked-admin.png | CCTV segmentation | Design constraint |
| 6 | 06-cctv-internal.png | CCTV VLAN internal | Design constraint |
| 7 | 07-cctv-no-nat.png | CCTV excluded from NAT | Design constraint |
| 8 | 08-nat-translations.png | Static NAT entries | Assigned challenge |
| 9 | 09-nat-statistics.png | NAT statistics | Assigned challenge |
| 10 | 10-acl-verification.png | Both ACLs active | Security |
| 11 | 11-vlan-brief.png | All VLANs active | Design |
| 12 | 12-routing-table.png | Routing functional | Design |

**All evidence confirms the network meets its design requirements.**

---

*End of Appendix B*