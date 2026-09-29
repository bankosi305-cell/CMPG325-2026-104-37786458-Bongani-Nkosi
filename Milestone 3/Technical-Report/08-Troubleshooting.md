# 8. TROUBLESHOOTING

## 8.1 Overview

During the design and implementation of the network, several technical challenges were encountered. This section documents the most significant issues and the troubleshooting steps taken to resolve them.

## 8.2 Issue 1 — DHCP Clients Received APIPA Addresses (169.254.x.x)

### Problem
PCs in the Admin, Finance, HR, and Public Services VLANs could not obtain IP addresses via DHCP. They automatically assigned themselves APIPA addresses (169.254.x.x), indicating a failure to reach the DHCP server.

### Root Cause
Access switch ports were not assigned to their departmental VLANs. Without VLAN tagging, DHCP requests stayed in VLAN 1 and never reached the DHCP service on the Core-SW.

### Resolution
Assigned the correct VLAN to each access port:
interface range FastEthernet0/1-5
switchport mode access
switchport access vlan 10
spanning-tree portfast


Applied the same pattern on SW-Fin (VLAN 20), SW-HR (VLAN 30), SW-Public (VLAN 40), and SW-CCTV (VLAN 100).

### Verification
After the fix, all PCs successfully obtained DHCP addresses on subsequent requests.

## 8.3 Issue 2 — Native VLAN Mismatch on Trunk Links

### Problem
The CDP log on Core-SW showed repeated warnings:
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/9 (1),
with SW-CCTV FastEthernet0/1 (100).


### Root Cause
The native VLAN on the trunk port did not match between Core-SW and SW-CCTV. This was a side effect of the port-configuration commands used earlier.

### Resolution
Explicitly set the native VLAN to 1 on both ends:

**On Core-SW:**
interface FastEthernet0/9
switchport trunk native vlan 1


**On SW-CCTV:**
interface FastEthernet0/1
switchport trunk native vlan 1


### Verification
The CDP warning disappeared after the configuration was saved on both ends.

## 8.4 Issue 3 — VLAN 200 SVI Used the Wrong /28 Subnet

### Problem
Servers in VLAN 200 (10.40.20.0/28) could not communicate with the Core-SW, even though the physical links were up.

### Root Cause
The SVI for VLAN 200 was initially configured as:
interface Vlan200
ip address 10.40.20.254 255.255.255.240


A `/28` subnet spans `10.40.20.0` to `10.40.20.15`. The address `10.40.20.254` falls **outside this /28 range** (it would belong to `10.40.20.240/28`). Servers with the mask `/28` did not consider `.254` to be on their subnet, so ARP requests never reached it.

### Resolution
The SVI was reconfigured to the last usable IP in the correct `/28` subnet:
interface Vlan200
ip address 10.40.20.14 255.255.255.240


Servers were also updated:

| Server | IP | Gateway |
|--------|-----|---------|
| App-Server | 10.40.20.2 | 10.40.20.14 |
| File-Server | 10.40.20.3 | 10.40.20.14 |
| DHCP-Server | 10.40.20.4 | 10.40.20.14 |

### Verification
Ping tests from Core-SW to each server succeeded after the correction. Server-to-SVI ping also succeeded.

## 8.5 Issue 4 — ACL 102 Blocked Return Traffic

### Problem
Even after ACL 102 was applied, servers were unreachable. Admin-PC1 could not ping the App-Server.

### Root Cause
Extended ACLs are **stateless** — they do not track connections. The initial ACL only permitted traffic **to** the server, not the reply **from** the server. Return packets were denied by the implicit `deny any any` at the end of the ACL.

### Resolution
Added explicit bidirectional rules to ACL 102:
access-list 102 permit ip 10.40.20.0 0.0.0.15 10.40.1.0 0.0.0.255
access-list 102 permit ip 10.40.20.0 0.0.0.15 10.40.2.0 0.0.0.255
access-list 102 permit ip 10.40.20.0 0.0.0.15 10.40.5.0 0.0.0.255
access-list 102 permit ip 10.40.20.0 0.0.0.15 10.40.20.0 0.0.0.15


### Verification
Admin-PC1 could reach the App-Server with 4/4 packets after this change.

## 8.6 Issue 5 — ACLs Lost After Reload

### Problem
During Milestone 3 preparation, verification tests showed that HR-PC1 could reach the App-Server (which should be blocked). The ACLs were no longer applied to the VLAN interfaces.

### Root Cause
When the .pkt file was saved and reloaded between milestones, the ACL **interface assignments** were not preserved (the ACL rules themselves remained, but the `ip access-group` commands had to be re-applied).

### Resolution
Re-applied the ACLs to the VLAN interfaces:
interface vlan 100
ip access-group 101 in
exit

interface vlan 200
ip access-group 102 in
exit


### Verification
Subsequent tests confirmed HR was blocked from the server, and CCTV was blocked from office subnets.

### Lesson Learned
Always verify `show running-config interface vlan X` after each milestone — to ensure ACLs are still attached to the correct interfaces.

## 8.7 Issue 6 — CCTV Devices Had No Internet Access (Desired)

### Observation
CCTV pings to 8.8.8.1 returned 100% packet loss.

### Root Cause
**This is by design.** The NAT ACL (access-list 1) does not permit `10.40.10.0/24`. CCTV traffic cannot be translated to a public IP.

### Resolution
No action taken — this is the intended behaviour.

### Interpretation
This behaviour explicitly enforces the CCTV segmentation constraint at the NAT layer. It is documented as a security feature rather than a bug.

## 8.8 Summary of Troubleshooting

| # | Issue | Resolution | Status |
|---|-------|-----------|--------|
| 1 | DHCP failures | Assigned VLAN to access ports | ✅ |
| 2 | Native VLAN mismatch | Set native vlan 1 on both trunk ends | ✅ |
| 3 | VLAN 200 SVI subnet mismatch | Changed SVI IP to 10.40.20.14 | ✅ |
| 4 | ACL 102 blocked replies | Added bidirectional permit rules | ✅ |
| 5 | ACLs lost after reload | Re-applied to VLAN interfaces | ✅ |
| 6 | CCTV no internet | By design — enforces constraint | ✅ |

## 8.9 Key Lessons Learned

1. **Subnet masks matter** — a single misconfigured mask can break connectivity across an entire VLAN.
2. **Extended ACLs are stateless** — bidirectional rules are required when both request and reply must flow.
3. **Trunk consistency is critical** — native VLAN mismatches silently break inter-switch communication.
4. **Save-and-verify** — always verify critical config (ACLs, SVIs) after reloading a .pkt file.
5. **"Failures" aren't always failures** — some 100% packet loss results (CCTV → internet) prove that security policies are working.

---

*End of Section 8*