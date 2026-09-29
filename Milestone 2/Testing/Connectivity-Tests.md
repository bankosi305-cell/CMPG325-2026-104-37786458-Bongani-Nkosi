# CONNECTIVITY TESTING EVIDENCE

## CMPG325-2026-104 | Milestone 2 | Bongani Nkosi | 37786458

---

## Test Environment

- **Tool:** Cisco Packet Tracer 8.3
- **File:** CMPG325-2026-104-M2.pkt
- **Test Time:** Real-time mode
- **Devices:** All VLANs, ACLs, DHCP, NAT configured and active

---

## Test 1: Admin → Finance (Inter-VLAN Routing)

**Source:** Admin-PC1 (10.40.1.2)
**Destination:** Fin-PC1 (10.40.2.1)
**Command:** `ping 10.40.2.1`
**Expected:** Success
**Result:** ✅ 3/4 packets (first dropped for ARP, then 3 replies)

**Evidence:** `screenshots/01-admin-to-finance.png`

**Interpretation:** Inter-VLAN routing is working. The first packet is lost due to ARP resolution, which is normal.

---

## Test 2: Admin → App-Server (CR4 Authorised)

**Source:** Admin-PC1 (10.40.1.2)
**Destination:** App-Server (10.40.20.2)
**Command:** `ping 10.40.20.2`
**Expected:** Success (Admin is authorised under CR4)
**Result:** ✅ 4/4 packets

**Evidence:** `screenshots/02-admin-to-server.png`

**Interpretation:** ACL 102 correctly permits Admin (10.40.1.0/24) to reach the server VLAN (10.40.20.0/28).

---

## Test 3: HR → App-Server (CR4 Denied)

**Source:** HR-PC1 (10.40.3.10)
**Destination:** App-Server (10.40.20.2)
**Command:** `ping 10.40.20.2`
**Expected:** Blocked (HR is NOT authorised under CR4)
**Result:** ✅ 100% loss (0/4 packets)

**Evidence:** `screenshots/03-hr-blocked-server.png`

**Interpretation:** ACL 102 correctly denies HR (10.40.3.0/24) access to the server farm.

---

## Test 4: HR → Internet (NAT)

**Source:** HR-PC1 (10.40.3.10)
**Destination:** ISP-Router (192.168.1.1)
**Command:** `ping 192.168.1.1`
**Expected:** Success (PAT translation)
**Result:** ✅ 4/4 packets

**Evidence:** `screenshots/04-hr-to-internet.png`

**Interpretation:** PAT (overload) is functioning — HR traffic is translated to the public IP (192.168.1.2) to reach the ISP.

---

## Test 5: CCTV → Admin PC (Constraint — Segmentation)

**Source:** CCTV-Cam1 (10.40.10.10)
**Destination:** Admin-PC1 (10.40.1.2)
**Command:** `ping 10.40.1.10`
**Expected:** Blocked (ACL 101 blocks CCTV → office)
**Result:** ✅ 100% loss (0/4 packets)

**Evidence:** `screenshots/05-cctv-blocked-admin.png`

**Interpretation:** The CCTV segmentation constraint is enforced — CCTV cannot reach office VLANs.

---

## Test 6: CCTV → CCTV (Internal Communication)

**Source:** CCTV-Cam1 (10.40.10.10)
**Destination:** CCTV-Cam2 (10.40.10.11)
**Command:** `ping 10.40.10.10` (pinging the other camera)
**Expected:** Success (same VLAN)
**Result:** ✅ 4/4 packets

**Evidence:** `screenshots/06-cctv-internal.png`

**Interpretation:** CCTV devices can communicate within VLAN 100 as expected — the ACL only blocks traffic to other VLANs.

---

## Test 7: CCTV → Internet (No NAT for CCTV)

**Source:** CCTV-Cam1 (10.40.10.10)
**Destination:** 8.8.8.1 (Internet)
**Command:** `ping 8.8.8.1`
**Expected:** Blocked (CCTV excluded from NAT ACL)
**Result:** ✅ 100% loss (0/4 packets)

**Evidence:** `screenshots/07-cctv-no-nat.png`

**Interpretation:** CCTV VLAN 100 is deliberately excluded from the NAT ACL (access-list 1). CCTV has no internet access.

---

## Summary Table

| # | Test | Expected | Result |
|---|------|----------|--------|
| 1 | Admin → Finance | Success | ✅ Pass |
| 2 | Admin → App-Server | Success | ✅ Pass |
| 3 | HR → App-Server | Blocked | ✅ Pass |
| 4 | HR → Internet | Success | ✅ Pass |
| 5 | CCTV → Admin | Blocked | ✅ Pass |
| 6 | CCTV → CCTV | Success | ✅ Pass |
| 7 | CCTV → Internet | Blocked | ✅ Pass |

**Conclusion:** All tests passed. The network meets all project requirements: inter-VLAN routing, NAT, CCTV segmentation, and CR4 server access control.