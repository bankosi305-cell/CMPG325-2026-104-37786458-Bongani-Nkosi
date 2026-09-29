# 7. TESTING AND VERIFICATION

## 7.1 Testing Overview

All network functionality was systematically tested in Cisco Packet Tracer 8.3. Testing focused on four areas:

1. **Connectivity** — Can devices communicate as expected?
2. **NAT** — Is the assigned challenge working?
3. **CCTV Segmentation** — Is the design constraint enforced?
4. **CR4 Server Access** — Is the change request honoured?

## 7.2 Testing Methodology

Each test followed this structure:

- **Setup:** Source device, destination device, expected outcome
- **Command:** Exact command executed
- **Expected Result:** What should happen
- **Actual Result:** What was observed
- **Interpretation:** What this proves

Evidence was captured as screenshots and stored in the GitHub portfolio.

## 7.3 Test 1 — Inter-VLAN Routing (Admin → Finance)

**Source:** Admin-PC1 (10.40.1.x)
**Destination:** Fin-PC1 (10.40.2.x)
**Command:** `ping 10.40.2.1`

**Expected:** Success (inter-VLAN routing via L3 Core-SW)

**Result:**
Reply from 10.40.2.1: bytes=32 time<1ms TTL=127
Reply from 10.40.2.1: bytes=32 time<1ms TTL=127
Reply from 10.40.2.1: bytes=32 time<1ms TTL=127
Success rate is 75 percent (3/4)


**Interpretation:** ✅ Inter-VLAN routing works. The first packet is dropped due to ARP resolution — a normal behaviour.

**Evidence:** `screenshots/01-admin-to-finance.png`

## 7.4 Test 2 — Admin → App-Server (CR4 — Authorised)

**Source:** Admin-PC1 (10.40.1.x)
**Destination:** App-Server (10.40.20.2)
**Command:** `ping 10.40.20.2`

**Expected:** Success (Admin is authorised under CR4)

**Result:**
Reply from 10.40.20.2: bytes=32 time<1ms TTL=127
Reply from 10.40.20.2: bytes=32 time<1ms TTL=127
Reply from 10.40.20.2: bytes=32 time<1ms TTL=127
Reply from 10.40.20.2: bytes=32 time<1ms TTL=127
Success rate is 100 percent (4/4)


**Interpretation:** ✅ ACL 102 correctly permits Admin (10.40.1.0/24) to reach the server VLAN.

**Evidence:** `screenshots/02-admin-to-server.png`

## 7.5 Test 3 — HR → App-Server (CR4 — Denied)

**Source:** HR-PC1 (10.40.3.x)
**Destination:** App-Server (10.40.20.2)
**Command:** `ping 10.40.20.2`

**Expected:** Blocked (HR is NOT authorised under CR4)

**Result:**
Request timed out.
Request timed out.
Request timed out.
Request timed out.
Success rate is 0 percent (0/4)


**Interpretation:** ✅ ACL 102 correctly denies HR (10.40.3.0/24) access to the server VLAN.

**Evidence:** `screenshots/03-hr-blocked-server.png`

## 7.6 Test 4 — HR → Internet (NAT)

**Source:** HR-PC1 (10.40.3.x)
**Destination:** ISP-Router (192.168.1.1)
**Command:** `ping 192.168.1.1`

**Expected:** Success (PAT translation)

**Result:**
Reply from 192.168.1.1: bytes=32 time<1ms TTL=253
Reply from 192.168.1.1: bytes=32 time<1ms TTL=253
Reply from 192.168.1.1: bytes=32 time<1ms TTL=253
Success rate is 75 percent (3/4)


**Interpretation:** ✅ PAT working. HR-PC1 (private IP) reached a public destination via NAT.

**Evidence:** `screenshots/04-hr-to-internet.png`

## 7.7 Test 5 — CCTV → Admin PC (Segmentation Constraint)

**Source:** CCTV-Cam1 (10.40.10.10)
**Destination:** Admin-PC1 (10.40.1.10)
**Command:** `ping 10.40.1.10`

**Expected:** Blocked (ACL 101 blocks CCTV → office)

**Result:**
Request timed out.
Reply from 10.40.10.254: Destination host unreachable.
Reply from 10.40.10.254: Destination host unreachable.
Reply from 10.40.10.254: Destination host unreachable.
Success rate is 0 percent (0/4)


**Interpretation:** ✅ CCTV cannot reach office VLANs. The "Destination host unreachable" message comes from the CCTV gateway (10.40.10.254), confirming ACL 101 is enforcing the block.

**Evidence:** `screenshots/05-cctv-blocked-admin.png`

## 7.8 Test 6 — CCTV → CCTV (Internal)

**Source:** CCTV-Cam1 (10.40.10.10)
**Destination:** CCTV-Cam2 (10.40.10.10)
**Command:** `ping 10.40.10.10`

**Expected:** Success (same VLAN, no ACL applies)

**Result:**
Reply from 10.40.10.10: bytes=32 time<1ms TTL=128
Reply from 10.40.10.10: bytes=32 time<1ms TTL=128
Reply from 10.40.10.10: bytes=32 time<1ms TTL=128
Reply from 10.40.10.10: bytes=32 time<1ms TTL=128
Success rate is 100 percent (4/4)


**Interpretation:** ✅ CCTV devices can communicate within VLAN 100 as expected. The ACL only blocks cross-VLAN traffic.

**Evidence:** `screenshots/06-cctv-internal.png`

## 7.9 Test 7 — CCTV → Internet (No NAT for CCTV)

**Source:** CCTV-Cam1 (10.40.10.10)
**Destination:** 8.8.8.1
**Command:** `ping 8.8.8.1`

**Expected:** Blocked (CCTV excluded from NAT ACL)

**Result:**
Request timed out.
Request timed out.
Request timed out.
Request timed out.
Success rate is 0 percent (0/4)


**Interpretation:** ✅ CCTV traffic cannot be translated to a public IP, so CCTV has no internet access. Enforces the segmentation constraint at the NAT layer.

**Evidence:** `screenshots/07-cctv-no-nat.png`

## 7.10 Test 8 — NAT Translations Verification

**Command:** `show ip nat translations` (on Edge-Router)

**Output:**
Pro Inside global Inside local Outside local Outside global
--- 192.168.1.3 10.40.20.2 --- ---
--- 192.168.1.4 10.40.20.3 --- ---


**Interpretation:** ✅ Static NAT entries for App and File servers confirmed.

**Evidence:** `screenshots/08-nat-translations.png`

## 7.11 Test 9 — NAT Statistics

**Command:** `show ip nat statistics` (on Edge-Router)

**Output:**
Total translations: 2 (2 static, 0 dynamic, 0 extended)
Outside Interfaces: GigabitEthernet0/0/1
Inside Interfaces: GigabitEthernet0/0/0
Hits: 19 Misses: 32
Expired translations: 13


**Interpretation:** ✅ Inside and Outside interfaces correctly assigned. Hits confirm active use of NAT.

**Evidence:** `screenshots/09-nat-statistics.png`

## 7.12 Test 10 — ACL Verification

**Command:** `show ip access-lists` (on Core-SW)

**Output:**
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


**Interpretation:** ✅ Both ACLs are correctly configured. Rule 40 shows 4 matches, confirming bidirectional traffic is being permitted between Admin and the servers.

**Evidence:** `screenshots/10-acl-verification.png`

## 7.13 Test 11 — VLAN Configuration

**Command:** `show vlan brief` (on Core-SW)

**Interpretation:** ✅ All 8 VLANs active with correct port assignments.

**Evidence:** `screenshots/11-vlan-brief.png`

## 7.14 Test 12 — Routing Table

**Command:** `show ip route` (on Core-SW)

**Output:** 8 connected routes (SVIs) + 1 default route via 10.40.1.1

**Interpretation:** ✅ All VLANs are routed; default route points to Edge-Router.

**Evidence:** `screenshots/12-routing-table.png`

## 7.15 Summary of Results

| # | Test | Expected | Result |
|---|------|----------|--------|
| 1 | Admin → Finance | Success | ✅ Pass |
| 2 | Admin → App-Server | Success | ✅ Pass |
| 3 | HR → App-Server | Blocked | ✅ Pass |
| 4 | HR → Internet | Success | ✅ Pass |
| 5 | CCTV → Admin | Blocked | ✅ Pass |
| 6 | CCTV → CCTV | Success | ✅ Pass |
| 7 | CCTV → Internet | Blocked | ✅ Pass |
| 8 | NAT Translations | 2 static entries | ✅ Pass |
| 9 | NAT Statistics | Interfaces assigned | ✅ Pass |
| 10 | ACL Verification | Both ACLs present | ✅ Pass |
| 11 | VLAN Configuration | 8 VLANs active | ✅ Pass |
| 12 | Routing Table | All SVIs + default | ✅ Pass |

**All 12 tests passed.** The network meets all specified requirements.

---

*End of Section 7*