# MILESTONE 2 — NETWORK IMPLEMENTATION & TESTING

## CMPG325-2026-104 | Bongani Nkosi | 37786458

---

## 1. PROJECT OVERVIEW

This milestone delivers the working Cisco Packet Tracer implementation of the Ramotshere Moiloa Local Municipality network (Zeerust), with all VLANs, ACLs, DHCP, and the assigned **NAT (Inside/Outside Address Translation)** challenge configured and verified.

---

## 2. DELIVERABLES SUMMARY

| # | Deliverable | Location | Status |
|---|-------------|----------|--------|
| 1 | Working Packet Tracer file | `Packet-Tracer/CMPG325-2026-104-M2.pkt` | ✅ |
| 2 | Device configurations | `Configurations/*.txt` | ✅ |
| 3 | Connectivity testing evidence | `Testing/Connectivity-Tests.md` | ✅ |
| 4 | NAT verification | `Testing/NAT-Verification.md` | ✅ |
| 5 | ACL verification | `Testing/ACL-Verification.md` | ✅ |
| 6 | Screenshots | `Testing/screenshots/*.png` | ✅ (12 images) |

---

## 3. ASSIGNED NETWORKING CHALLENGE — NAT

**Challenge:** NAT (Inside/Outside Address Translation)

**Implementation:**

- **PAT (Overload):** All office department traffic (VLANs 10, 20, 30, 40, 50) is translated to a single public IP (`192.168.1.2`) using port address translation.
- **Static NAT:** The Application Server (`10.40.20.2`) and File Server (`10.40.20.3`) have static one-to-one mappings to `192.168.1.3` and `192.168.1.4` respectively.
- **CCTV Excluded:** VLAN 100 (CCTV) is deliberately excluded from NAT — it has no internet access, enforcing the segmentation constraint.

**Verification commands:**