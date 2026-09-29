# 9. REFLECTION

## 9.1 Overall Reflection

Designing, implementing, and testing the Ramotshere Moiloa Local Municipality network has been a challenging but highly rewarding project. It combined theory (subnetting, VLANs, ACLs, NAT) with practical application in Cisco Packet Tracer and delivered a fully functional, service-rich enterprise network.

## 9.2 What I Learned

### 9.2.1 Technical Skills

- **VLSM and IP planning** — dividing a /16 block into functional subnets
- **Layer 2 segmentation** — VLAN design, trunk configuration, native VLAN consistency
- **Layer 3 routing** — SVIs, inter-VLAN routing, static routing, default routes
- **NAT** — understanding and configuring PAT (Overload) and Static NAT
- **ACL design** — extended ACLs, direction (`in` vs `out`), and stateless behaviour
- **DHCP** — configuring scope, gateway, DNS, and troubleshooting failures
- **Packet Tracer** — device selection, cabling, simulation, CLI configuration

### 9.2.2 Professional Skills

- **Documentation discipline** — using Markdown, screenshots, and a GitHub portfolio
- **Troubleshooting methodology** — systematic approach (isolate, verify, resolve)
- **Design justification** — explaining decisions in engineering terms
- **Change management** — handling CR4 within the existing design

### 9.2.3 Conceptual Understanding

- **Layered troubleshooting** — checking L1 (cables), L2 (VLANs), L3 (routing), L4+ (ACLs) in order
- **The value of "expected failures"** — packets dropped by ACLs are proof of working security policy
- **Trade-offs** — centralised DHCP vs. external, L3 core vs. router-on-a-stick, PAT vs. Static NAT
- **Defensive design** — planning for failures, verifying after each change

## 9.3 Challenges Faced

### 9.3.1 Subnet Mask Misconfiguration

The VLAN 200 SVI was initially configured with an IP outside its own /28 subnet. This caused hours of debugging before it was identified. Lesson: always re-check that a gateway IP falls within its subnet range.

### 9.3.2 Stateless ACLs

The initial ACL 102 only permitted traffic in one direction. This is a common conceptual trap. The fix — adding bidirectional permit rules — taught me how stateless ACLs actually operate in Cisco IOS.

### 9.3.3 Native VLAN Mismatch

The trunk between Core-SW and SW-CCTV silently dropped VLAN 100 traffic due to a native VLAN mismatch. This wasn't visible until the CDP log was checked. Lesson: always verify trunk configuration on both ends.

### 9.3.4 ACL Persistence

ACLs were found missing from interfaces after the .pkt was reloaded between milestones. This reinforced the importance of re-verifying configuration after each environment change.

## 9.4 What I Would Do Differently

| Area | Improvement |
|------|-------------|
| **Testing Order** | Test each VLAN and ACL right after configuration, not at the end |
| **Documentation** | Document as I go, rather than reconstructing later |
| **Backup Configs** | Save configs to text files after each session |
| **Version Control** | Use Git commits more frequently for the .pkt file |
| **Security** | Add port security and DHCP snooping for extra hardening |

## 9.5 Future Improvements

If the network were to be extended or productionised, the following improvements would be considered:

1. **Separate VLAN 50 for IT** — In this implementation, IT PCs share VLAN 10 with Admin. A future iteration would move IT to its own VLAN 50 subnet.
2. **Redundant links** — Implement a redundant uplink from Core-SW to Edge-Router, with HSRP or VRRP.
3. **DHCP Snooping** — Prevent rogue DHCP servers on the access layer.
4. **Port Security** — Limit MAC addresses per port to prevent unauthorised devices.
5. **Centralised Syslog Server** — For auditing and monitoring.
6. **NTP** — Sync timestamps across devices for accurate log correlation.
7. **AAA** — Implement TACACS+ or RADIUS for centralised authentication.
8. **Real Firewall** — Replace the Edge-Router's ACL with a dedicated firewall appliance for stateful inspection.
9. **VPN for Remote Access** — Allow secure remote connectivity for staff.
10. **Wi-Fi Integration** — Add wireless access points with WPA3, integrated into the same VLAN structure.

## 9.6 Personal Reflection

This project has given me a deep appreciation for how much invisible engineering goes into a functional network. What looks like "just an internet connection" is actually the result of careful VLAN planning, routing decisions, ACL rules, and NAT translations.

The most valuable lesson was learning to troubleshoot methodically. When HR-PC1 couldn't reach the App-Server despite "everything looking correct", we traced through L1 → L2 → L3 → L4+ in order, and found the problem was a stateless ACL blocking return traffic. That's the kind of insight only gained through hands-on experience.

I also appreciated the discipline required to document everything — not just the working parts, but the journey of fixing things. That's what separates a working project from a professional one.

## 9.7 Conclusion

The network successfully meets all project requirements:

- ✅ Connectivity across all departments
- ✅ Inter-VLAN routing
- ✅ CCTV segmentation (constraint)
- ✅ NAT (assigned challenge) — configured, verified, and demonstrated
- ✅ CR4 server access control (change request)
- ✅ DHCP services for office users
- ✅ Fully documented, tested, and reproducible

I am confident this design would serve the Ramotshere Moiloa Local Municipality well — providing secure, reliable, and scalable connectivity that could be extended as the municipality grows.

---

*End of Section 9*