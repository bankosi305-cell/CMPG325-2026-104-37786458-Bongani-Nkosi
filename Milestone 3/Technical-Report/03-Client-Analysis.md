# 3. CLIENT ANALYSIS

## 3.1 Client Overview

| Field | Value |
|-------|-------|
| **Organisation** | Ramotshere Moiloa Local Municipality Offices |
| **Location** | Zeerust, North West Province, South Africa |
| **Industry** | Municipal Services |
| **Client ID** | CLI-104 |
| **Project ID** | CMPG325-2026-104 |
| **Assigned Block** | 10.40.0.0/16 |

## 3.2 Organisational Context

The Ramotshere Moiloa Local Municipality provides essential public services to the Zeerust community. Its administrative offices manage:
- Financial records (rates, billing, procurement)
- Human resources and payroll
- Public-facing services (permits, licensing, community engagement)
- Security monitoring (CCTV across municipal buildings)

The network must serve multiple departments with distinct data, privacy, and access requirements.

## 3.3 Client Requirements (from Project Brief)

| # | Requirement |
|---|-------------|
| 1 | Provide appropriate connectivity and network services |
| 2 | Accommodate the stated design constraint (CCTV segmentation) |
| 3 | Address Change Request CR4 (restricted server access) |
| 4 | Implement the assigned challenge: NAT (Inside/Outside Address Translation) |
| 5 | Use the assigned addressing block: 10.40.0.0/16 |
| 6 | Produce a working, testable Packet Tracer implementation |

## 3.4 Design Constraint

> **"CCTV traffic must be segmented from office data traffic."**

This is a **non-negotiable security requirement**. CCTV cameras handle sensitive video streams and must not be reachable from office VLANs, and vice versa. This requires:
- A dedicated CCTV VLAN
- ACL enforcement on the CCTV gateway interface
- CCTV devices to be excluded from NAT (no internet access)

## 3.5 Change Request CR4

> **"A new application/file server is installed and must be reachable by authorised departments only."**

This change request adds a **server access control layer**:
- Only **Administration, Finance, and IT** departments can reach the servers
- **HR and Public Services** are denied
- **CCTV** is also denied (in line with the segmentation constraint)

## 3.6 Assumptions

Because the project brief does not specify all organisational details, the following **reasonable engineering assumptions** were made and justified.

### 3.6.1 Departmental Structure

| Department | Assumed Users | Justification |
|------------|--------------|---------------|
| Administration | 15 | Executive, legal, registry |
| Finance | 12 | Budget, payroll, procurement |
| Human Resources | 8 | Staff records, recruitment |
| Public Services | 20 | Customer-facing services |
| IT/Technical | 6 | Network and systems support |
| Security/CCTV | 4 | Monitoring operators |
| **Total** | **65 users** | Mid-sized municipal office |

### 3.6.2 Device Count

| Device Type | Quantity | Justification |
|-------------|----------|---------------|
| PCs | ~12 (simulated) | One per department minimum, scaled down for simulation |
| Printers | 5 (1 per department) | Typical municipal setup |
| Servers | 3 | Application, File, DHCP |
| CCTV Cameras | 8-12 | Standard perimeter + internal coverage |
| Switches | 6 | Core + 5 access |
| Routers | 2 | Edge + ISP simulation |

### 3.6.3 Building Layout

Assumed **three-building campus**:
1. **Main Admin Building** — Admin, Finance, HR, IT
2. **Public Services Centre** — Public-facing services
3. **IT/Server Room** — Core infrastructure + servers

Plus a **separate CCTV building/zone** for camera operations.

### 3.6.4 Services Required

| Service | Reason |
|---------|--------|
| DHCP | Simplify client management |
| Inter-VLAN routing | Enable departmental communication |
| NAT (PAT) | Internet access for office staff |
| Static NAT | Server-side external reachability |
| ACLs | Security enforcement |
| DNS (public) | External name resolution |

### 3.6.5 Internet Connectivity

Assumed:
- Single ISP uplink
- Public IP range **192.168.1.0/30** for external facing
- Public IP pool **192.168.1.2 - 192.168.1.4** for NAT translations

## 3.7 Justification of Design Decisions

| Decision | Justification |
|----------|---------------|
| VLAN per department | Standard best practice; isolates broadcast domains; enables ACL policies |
| Layer 3 Core Switch | Centralises routing; efficient inter-VLAN traffic; scalable |
| Centralised DHCP on Core-SW | Simplifies management; avoids external DHCP relay complexity |
| Separate server VLAN (200) | Enables precise ACL control for CR4 |
| Static IPs for servers | Required for reliable NAT static mappings and services |
| DHCP for office PCs | Reduces admin overhead |
| CCTV VLAN isolated | Enforces security constraint |
| ACL 101 for CCTV | Prevents CCTV ↔ office traffic |
| ACL 102 with bidirectional rules | Extended ACLs are stateless; both directions must be permitted for authorised flows |

## 3.8 Requirements Traceability Matrix

| Requirement | Design Element | Section |
|-------------|---------------|---------|
| Connectivity | VLANs, trunk links, inter-VLAN routing | 4 |
| NAT (assigned challenge) | PAT + Static NAT on Edge Router | 5, 6 |
| CCTV Segmentation | VLAN 100 + ACL 101 | 5, 6 |
| CR4 Server Access | VLAN 200 + ACL 102 | 5, 6 |
| DHCP | Core-SW DHCP pools | 5 |
| Addressing Block 10.40.0.0/16 | VLSM subnet allocation | 4 |
| Packet Tracer Implementation | Working .pkt file | Appendix A |