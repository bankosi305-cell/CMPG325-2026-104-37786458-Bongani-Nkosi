# 1. EXECUTIVE SUMMARY

## 1.1 Project Overview

This report documents the design, implementation, and testing of an enterprise-grade computer network for the **Ramotshere Moiloa Local Municipality Offices** in Zeerust, North West Province. The network was built and simulated entirely in **Cisco Packet Tracer 8.3** and addresses the connectivity, security, and service requirements specified in the project brief (CMPG325-2026-104).

## 1.2 Problem Statement

The municipality required a network that:
- Provides reliable connectivity across five departments (Administration, Finance, Human Resources, Public Services, and IT)
- Segments sensitive **CCTV traffic** from office data traffic for security
- Provides **NAT (Network Address Translation)** for external communication using the assigned block **10.40.0.0/16**
- Restricts access to a new **application/file server** so only authorised departments can reach it (Change Request CR4)

## 1.3 Solution Delivered

The final network solution includes:

- **Hierarchical three-tier design**: Core, Distribution, and Access layers
- **8 VLANs** for logical segmentation
- **Inter-VLAN routing** via a Layer 3 Core Switch
- **DHCP services** for all office departments (centralised on the Core-SW)
- **NAT (PAT + Static)** on the Edge Router to provide internet access for office users while excluding CCTV
- **ACL 101** to enforce CCTV segmentation
- **ACL 102** to enforce CR4 server access restrictions
- **Full documentation** including configs, testing evidence, and troubleshooting logs

## 1.4 Key Outcomes

| Requirement | Status |
|-------------|--------|
| Departmental connectivity | ✅ Achieved |
| Inter-VLAN routing | ✅ Achieved |
| CCTV segmentation (Constraint) | ✅ Achieved |
| NAT (Inside/Outside Address Translation) | ✅ Achieved |
| CR4 Server access control | ✅ Achieved |
| DHCP for office users | ✅ Achieved |
| Tested, working Packet Tracer implementation | ✅ Achieved |

## 1.5 Report Structure

The remainder of this report is organised as follows:

- **Section 2** provides an introduction and project scope.
- **Section 3** analyses the client requirements, constraints, and design assumptions.
- **Section 4** describes the physical and logical network design.
- **Section 5** details the implementation and device configurations.
- **Section 6** explains the assigned NAT challenge in depth.
- **Section 7** presents the testing and verification evidence.
- **Section 8** documents the troubleshooting performed during development.
- **Section 9** reflects on the project, lessons learned, and future improvements.
- **Section 10** lists references.

The **Appendix** contains full device configurations and evidence screenshots.