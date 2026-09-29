# 2. INTRODUCTION

## 2.1 Background

The Ramotshere Moiloa Local Municipality, based in Zeerust, operates multiple administrative departments that require reliable and secure data communications. Like many South African municipalities, it faces the dual challenge of:
1. Providing modern digital services to internal departments and the public
2. Protecting sensitive data and systems from unauthorised access

This project addresses those needs by designing and simulating a **secure, segmented, and service-rich enterprise network** using Cisco technologies.

## 2.2 Project Scope

### In Scope
- Physical topology design (device placement, cabling)
- Logical topology design (VLANs, subnets, routing, security)
- IP addressing plan based on the assigned block **10.40.0.0/16**
- Implementation and configuration in Cisco Packet Tracer 8.3
- **Assigned challenge:** NAT (Inside/Outside Address Translation)
- **Constraint:** CCTV traffic segmentation
- **Change Request CR4:** Restricted server access for authorised departments
- Testing and verification of all functionality

### Out of Scope
- Actual physical deployment
- Real hardware procurement or installation
- Production DNS or email services
- External firewall appliances (unless simulated in Packet Tracer)

## 2.3 Project Objectives

| # | Objective |
|---|-----------|
| 1 | Design a topology appropriate for a mid-sized municipal office |
| 2 | Segment the network into logical VLANs |
| 3 | Provide inter-VLAN routing |
| 4 | Configure DHCP for office departments |
| 5 | Implement and verify NAT (PAT + Static) |
| 6 | Enforce CCTV segmentation via ACLs |
| 7 | Enforce CR4 server access via ACLs |
| 8 | Test all connectivity and security requirements |
| 9 | Document all work in a professional GitHub portfolio |
| 10 | Produce a working .pkt file, technical report, and demonstration video |

## 2.4 Methodology

The project followed a structured engineering approach:

1. **Requirement Analysis** — Extract requirements from project brief
2. **Assumption Documentation** — Make and justify reasonable assumptions where details were unspecified
3. **Design** — Physical and logical topology, IP plan, VLAN scheme, security model
4. **Implementation** — Build and configure in Packet Tracer
5. **Testing** — Verify each requirement with evidence
6. **Documentation** — Produce report, portfolio, and video

## 2.5 Tools Used

| Tool | Purpose |
|------|---------|
| **Cisco Packet Tracer 8.3** | Network design and simulation |
| **Visual Studio Code** | Markdown documentation writing |
| **Draw.io** | Topology diagrams |
| **GitHub** | Portfolio of evidence |
| **Microsoft Word / VS Code** | Report authoring (Markdown → PDF) |

## 2.6 Assumptions Summary

The following major assumptions were made (detailed in Section 3):

- 5 main departments exist (Administration, Finance, HR, Public Services, IT)
- ~75 total users across all departments
- 8-12 CCTV cameras monitoring the premises
- Two servers (Application + File) plus a dedicated DHCP server
- Three-building campus layout

## 2.7 Deliverables

The final deliverables for Milestone 3 are:

1. **Working Packet Tracer file** (`.pkt`)
2. **GitHub portfolio** with all documentation and evidence
3. **This technical report** (Markdown → PDF)
4. **15-20 minute individual video demonstration** with webcam inset
5. **Supporting files** (configuration exports, screenshots)