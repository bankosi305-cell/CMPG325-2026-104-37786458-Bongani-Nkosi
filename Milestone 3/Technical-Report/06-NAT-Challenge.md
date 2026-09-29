# 6. ASSIGNED CHALLENGE — NAT (INSIDE/OUTSIDE ADDRESS TRANSLATION)

## 6.1 Challenge Overview

The assigned networking challenge for this project is:

> **NAT (Inside/Outside Address Translation)**

NAT allows private internal IP addresses to be translated to public IP addresses for external communication. This project implements two forms of NAT:

1. **PAT (Port Address Translation / Overload)** — Many internal hosts share a single public IP
2. **Static NAT** — One-to-one permanent mapping for servers

## 6.2 Why NAT is Required

The Ramotshere Moiloa Local Municipality uses the **private IP block 10.40.0.0/16**, which is not routable on the public internet. Without NAT:
- Internal PCs could not reach external websites or services
- External parties could not reach the municipality's servers

NAT solves this by translating internal private IPs to a small pool of public IPs (or a single public IP with PAT).

## 6.3 NAT Types Explained

### 6.3.1 PAT (Port Address Translation)

**Purpose:** Allow many internal hosts to share a single public IP address by using unique source port numbers.

**Example:**
- Admin-PC1 (10.40.1.10) sends a packet to a website
- Edge-Router translates the source to `192.168.1.2:1024`
- When the reply returns to `192.168.1.2:1024`, the router translates back to `10.40.1.10:1024`

This is efficient — thousands of internal devices can share one public IP.

### 6.3.2 Static NAT

**Purpose:** Provide a stable, predictable public IP mapping for internal servers so external parties can reliably reach them.

**Example:**
- App-Server is at `10.40.20.2` internally
- It maps to `192.168.1.3` externally
- Any external connection to `192.168.1.3` is translated to `10.40.20.2`

## 6.4 NAT Configuration (Edge-Router)

### 6.4.1 Interface Roles

NAT requires the router to know which interface faces the internal network (inside) and which faces the internet (outside).

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


**Explanation:**
- `ip nat inside` — traffic entering from the LAN is eligible for translation
- `ip nat outside` — traffic leaving to the ISP is translated

### 6.4.2 ACL for NAT (Which Traffic Gets Translated)
access-list 1 permit 10.40.1.0 0.0.0.255
access-list 1 permit 10.40.2.0 0.0.0.255
access-list 1 permit 10.40.3.0 0.0.0.255
access-list 1 permit 10.40.4.0 0.0.0.255
access-list 1 permit 10.40.5.0 0.0.0.255
access-list 1 permit 10.40.20.0 0.0.0.15


**Key design decision:** The CCTV subnet (`10.40.10.0/24`) is **deliberately excluded** from this ACL.

**Effect:** CCTV devices cannot be translated and therefore cannot reach the internet. This enforces the security constraint that CCTV must be isolated.

### 6.4.3 PAT (Overload) Configuration
ip nat inside source list 1 interface GigabitEthernet0/0/1 overload


**Breakdown:**
- `ip nat inside source list 1` — translates traffic matching ACL 1
- `interface GigabitEthernet0/0/1` — uses the outside interface's IP (`192.168.1.2`)
- `overload` — enables PAT (port-based multiplexing) — many-to-one

### 6.4.4 Static NAT Configuration
ip nat inside source static 10.40.20.2 192.168.1.3
ip nat inside source static 10.40.20.3 192.168.1.4


**Effect:**
| Inside Local | Inside Global | Purpose |
|--------------|---------------|---------|
| 10.40.20.2 | 192.168.1.3 | App Server (CR4) |
| 10.40.20.3 | 192.168.1.4 | File Server (CR4) |

These static entries persist permanently, allowing external access to the servers.

## 6.5 NAT Verification

### 6.5.1 Verification Command 1: `show ip nat translations`

**Output observed:**

Pro Inside global Inside local Outside local Outside global
--- 192.168.1.3 10.40.20.2 --- ---
--- 192.168.1.4 10.40.20.3 --- ---


**Interpretation:**
- Two static entries are present for the servers ✅
- Dynamic PAT entries appear when traffic is generated and expire after idle timeout
- The 2 static mappings confirm servers are externally reachable

### 6.5.2 Verification Command 2: `show ip nat statistics`

**Output observed:**
Total translations: 2 (2 static, 0 dynamic, 0 extended)
Outside Interfaces: GigabitEthernet0/0/1
Inside Interfaces: GigabitEthernet0/0/0
Hits: 19 Misses: 32
Expired translations: 13
Dynamic mappings:


**Interpretation:**
- **2 static translations confirmed** ✅
- **Inside and Outside interfaces correctly assigned** ✅
- **Hits: 19** — NAT has actively translated traffic
- **Expired translations: 13** — Dynamic PAT entries have been created and expired (proof of active use)

### 6.5.3 Functional Verification — Ping Tests

**Test 1: HR-PC1 → ISP-Router (192.168.1.1)**
C:>ping 192.168.1.1
Reply from 192.168.1.1: bytes=32 time<1ms TTL=253
Reply from 192.168.1.1: bytes=32 time<1ms TTL=253
Reply from 192.168.1.1: bytes=32 time<1ms TTL=253
Success rate is 75 percent (3/4)


**Result:** ✅ NAT working. HR-PC1 (10.40.3.x) reached a public IP via PAT.

**Test 2: CCTV-Cam1 → 8.8.8.1 (Internet)**
C:>ping 8.8.8.1
Request timed out.
Request timed out.
Request timed out.
Request timed out.
Success rate is 0 percent (0/4)


**Result:** ✅ CCTV correctly blocked from internet access (excluded from NAT ACL).

## 6.6 Why NAT Is Appropriate Here

| Reason | Explanation |
|--------|-------------|
| **Conserves public IPs** | Only one public IP is needed for all office users |
| **Hides internal structure** | External parties cannot see the 10.40.0.0/16 topology |
| **Enables internet access** | Office staff can reach external services |
| **Allows external server access** | Static NAT makes servers reachable |
| **Enforces security policy** | CCTV is excluded — a design decision tying to the constraint |
| **Standard practice** | Matches real-world municipal network designs |

## 6.7 NAT Design Decisions

| Decision | Rationale |
|----------|-----------|
| PAT (Overload) for office users | Cost-effective; one public IP; sufficient for municipal scale |
| Static NAT for both servers | Predictable external reachability; required for CR4 server access |
| Exclude CCTV from NAT ACL | Reinforces the CCTV segmentation constraint |
| Use a single public IP (192.168.1.2) | Simple design; scalable for the office's user count |
| Use Loopback for simulated internet | Optional polish — provides a pingable "internet" endpoint |

## 6.8 Summary

The assigned NAT challenge has been:

- ✅ **Configured** on the Edge-Router (PAT + Static NAT)
- ✅ **Verified** using `show ip nat translations` and `show ip nat statistics`
- ✅ **Tested** via functional ping tests (HR reaches ISP; CCTV cannot)
- ✅ **Integrated** with the CCTV segmentation constraint (CCTV excluded from NAT)
- ✅ **Documented** with evidence screenshots (08-nat-translations.png, 09-nat-statistics.png)

**Assigned challenge successfully demonstrated.**

---

*End of Section 6*