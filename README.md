# CCNA-LABS

Hands-on Cisco Packet Tracer labs built alongside active study for the **Cisco CCNA 200-301** certification.

This repository documents my practical lab work as I progress through the CCNA curriculum. Each lab is paired with study notes, CLI verification, and a write-up explaining not just what was configured — but *why* the network behaves the way it does.

---

## About This Repository

I'm working through Jeremy's IT Lab's free CCNA 200-301 course and building hands-on labs in Cisco Packet Tracer to reinforce every topic. Labs are organized by topic block and follow a consistent structure:

- **Objective** — what the lab is trying to prove or demonstrate
- **Topology** — devices and connections used
- **Configuration** — commands applied with context
- **Verification** — show commands and what to look for
- **Key Observations** — what actually happened and why
- **Troubleshooting** — real errors encountered and how they were fixed
- **CCNA Exam Alignment** — which exam objective the lab supports

---

## Lab Progress

### ✅ Layer 2 Foundation

| Lab | Topic | Protocol / Method | Status |
|-----|-------|-------------------|--------|
| Lab 01 | Inter-VLAN Routing (Router-on-a-Stick) | 802.1Q, Subinterfaces | ✅ Complete |
| Lab 02 | STP Analysis | IEEE 802.1D PVST | ✅ Complete |
| Lab 03 | STP Root Manipulation & Port Protection | PVST+, PortFast, BPDU Guard | ✅ Complete |
| Lab 04 | Rapid STP Analysis | RSTP / Rapid PVST+ | ✅ Complete |
| Lab 05 | EtherChannel (Layer 2 & Layer 3) | LACP, PAgP, Static | ✅ Complete |
| Lab 08 | VLANs Part 1 — Access Ports | 802.1Q, Access Ports | ✅ Complete |
| Lab 09 | VLANs Part 2 — Trunking & ROAS | 802.1Q Trunking, Router-on-a-Stick | ✅ Complete |
| Lab 10 | VLANs Part 3 — Multilayer Switch & SVIs | SVIs, Layer 3 Switching | ✅ Complete |
| Lab 11 | DTP & VTP | Dynamic Trunking Protocol, VLAN Trunking Protocol | ✅ Complete |

### ✅ Layer 3 Core

| Lab | Topic | Protocol / Method | Status |
|-----|-------|-------------------|--------|
| Lab 06 | Static Routing | IPv4, Next-Hop, Default Route | ✅ Complete |
| Lab 07 | Subnetting (VLSM) | Variable-Length Subnet Masks | ✅ Complete |
| Lab 12 | Floating Static Routes | Administrative Distance, Backup Routes | ✅ Complete |

### ✅ Dynamic Routing

| Lab | Topic | Protocol / Method | Status |
|-----|-------|-------------------|--------|
| Lab 13 | EIGRP Configuration & Unequal-Cost Load-Balancing | EIGRP AS100, Variance, Feasible Successor | ✅ Complete |
| Lab 14 | OSPFv2 Single Area (Part 1) | OSPFv2, Area 0, ASBR, Default Route | ✅ Complete |
| Lab 15 | OSPFv2 Single Area (Part 2) | ip ospf interface command, Reference Bandwidth, Cost | ✅ Complete |
| Lab 16 | OSPFv2 Part 3 — DR/BDR, Network Types & Troubleshooting | Broadcast/P2P network types, Neighbor requirements, LSA types | ✅ Complete |

### ✅ Capstone

| Lab | Topic | Protocol / Method | Status |
|-----|-------|-------------------|--------|
| Lab 17 | Integrated Network Design | VLANs, EtherChannel, VTP, STP, OSPFv2, Floating Static, Serial WAN | ✅ Complete |

### ✅ IP Services & First Hop Redundancy

| Lab | Topic | Protocol / Method | Status |
|-----|-------|-------------------|--------|
| Lab 18 | HSRP Configuration | HSRP v2, Active/Standby, Preemption, Virtual IP/MAC | ✅ Complete |

### ✅ IPv6

| Lab | Topic | Protocol / Method | Status |
|-----|-------|-------------------|--------|
| Lab 19 | IPv6 Configuration Part 1 — Dual-Stack | Static IPv6, Dual-Stack, ipv6 unicast-routing | ✅ Complete |
| Lab 20 | IPv6 Address Types & EUI-64 (Part 2) | EUI-64, ipv6 enable, Link-Local | ✅ Complete |
| Lab 21 | IPv6 Static Routes (Part 3) | SLAAC, NDP, Fully Specified Static, Floating Static | ✅ Complete |

### ✅ Security — Access Control Lists

| Lab | Topic | Protocol / Method | Status |
|-----|-------|-------------------|--------|
| Lab 22 | Standard ACLs (Numbered & Named) | Standard ACLs, Wildcard Masks, OSPF | ✅ Complete |
| Lab 23 | Extended ACLs | TCP/UDP/IP matching, Port numbers, ACL editing | ✅ Complete |

### ✅ Network Management & IP Services

| Lab | Topic | Protocol / Method | Status |
|-----|-------|-------------------|--------|
| Lab 24 | CDP & LLDP | Cisco Discovery Protocol, Link Layer Discovery Protocol | ✅ Complete |
| Lab 25 | NTP | Network Time Protocol, Stratum, Authentication | ✅ Complete |
| Lab 26 | DNS | Domain Name System, Host Table, ip name-server | ✅ Complete |
| Lab 27 | DHCP | DHCP Server, Relay Agent, DHCP Client | ✅ Complete |
| Lab 28 | SNMP | Community Strings, MIB Browser, Get/Set Operations | ✅ Complete |
| Lab 29 | Syslog | Logging Destinations, Severity Levels, External Server | ✅ Complete |
| Lab 30 | SSH | RSA Keys, VTY Security, ACL-based Access Restriction | ✅ Complete |
| Lab 31 | FTP & TFTP | IOS Upgrade, File Transfer, Flash Management | ✅ Complete |
| Lab 32 | Static NAT | Inside/Outside Interfaces, One-to-One Mappings | ✅ Complete |
| Lab 33 | Dynamic NAT & PAT | NAT Pool, PAT Overload, Pool Exhaustion | ✅ Complete |

### ✅ Security — Layer 2 & QoS

| Lab | Topic | Protocol / Method | Status |
|-----|-------|-------------------|--------|
| Lab 34 | Voice VLANs | switchport voice vlan, ROAS, PoE, 802.1Q Tagging | ✅ Complete |
| Lab 35 | QoS | Class Maps, Policy Maps, DSCP Marking, Service Policy | ✅ Complete |

### ✅ Security — Layer 2 Hardening

| Lab | Topic | Protocol / Method | Status |
|-----|-------|-------------------|--------|
| Lab 36 | Port Security | Sticky MAC, Violation Modes | ✅ Complete |
| Lab 37 | DHCP Snooping | Trusted/Untrusted Ports, Binding Table | ✅ Complete |
| Lab 38 | Dynamic ARP Inspection | DAI, DHCP Snooping Integration | ✅ Complete |

### ✅ Redundancy & Tunneling

| Lab | Topic | Protocol / Method | Status |
|-----|-------|-------------------|--------|
| Lab 39 | STP & HSRP Synchronization | Root/Active Alignment, Priority Tuning | ✅ Complete |
| Lab 40 | GRE Tunnels | GRE over IPv4, OSPF over Tunnel | ✅ Complete |

### ✅ Wireless

| Lab | Topic | Protocol / Method | Status |
|-----|-------|-------------------|--------|
| Lab 41 | Wireless LAN | SSIDs, WLAN Security, AP Modes | ✅ Complete |

---

## Tools & Resources

| Tool | Purpose |
|------|---------|
| Cisco Packet Tracer | Network simulation and lab environment |
| Cisco IOS CLI | Configuration and verification |
| Jeremy's IT Lab | Primary CCNA study resource |
| Anki Flashcards | Daily review of key concepts |

---

## Certification Target

**Cisco Certified Network Associate (CCNA) 200-301)**

Exam domains covered by this lab series:

- **1.0 Network Fundamentals** — OSI model, Ethernet, switching, subnetting
- **2.0 Network Access** — VLANs, trunking, STP, RSTP, EtherChannel, SVIs, DTP, VTP
- **3.0 IP Connectivity** — Routing tables, static routes, floating static routes, EIGRP, OSPFv2, FHRP
- **4.0 IP Services** — DHCP, DNS, NTP, SNMP, CDP/LLDP, NAT, SSH, FTP/TFTP, QoS, Voice VLANs
- **5.0 Security Fundamentals** — Standard ACLs, Extended ACLs, Port Security, DHCP Snooping, Dynamic ARP Inspection
- **6.0 Automation & Programmability** — *(coming)*

---

## Repository Structure

```
CCNA-LABS/
├── LAB 01- Inter VLAN Routing/
├── LAB 02-STP_ANALYSIS/
├── LAB 03- STP/
├── LAB 04- Rapid STP/
├── LAB 05-EtherChannel/
├── LAB 06-Static Routing/
├── LAB 07- VLSM/
├── LAB 08-VLANs PT1/
├── LAB 09-VLANs PT2/
├── LAB 10-VLANs PT3/
├── LAB 11- DTP+ VTP/
├── LAB 12- Dynamic Routing/
├── LAB 13-EIGRP Config/
├── LAB 14- OSPF PT1/
├── LAB 15- OSPF PT2/
├── LAB 16- OSPF PT3/
├── LAB 17- Integrated Network Design/
├── LAB 18- HSRP Config/
├── LAB 19- IPv6 Config PT1/
├── LAB 20- IPv6 Config PT2/
├── LAB 21- IPv6 Config PT3/
├── LAB 22- Standard ACLs/
├── LAB 23- Extended ACLs/
├── LAB 24- CDP & LLDP/
├── LAB 25- NTP/
├── LAB 26- DNS/
├── LAB 27- DHCP/
├── LAB 28- SNMP/
├── LAB 29- Syslog/
├── LAB 30- SSH/
├── LAB 31- FTP & TFTP/
├── LAB 32- NAT PT. 1/
├── LAB 33- NAT PT.2/
├── LAB 34- Voice VLANS/
├── LAB 35- QoS/
├── LAB 36- Port Security/
├── LAB 37- DHCP Snooping/
├── LAB 38- Dynamic ARP Inspection/
├── LAB 39- STP & HSRP Synchronization/
├── LAB 40- GRE Tunnels/
├── LAB 41- Wireless LAN/
└── README.md  ← you are here
```

---

*Updated as new labs are completed. Each lab README contains full configuration details, CLI output, and exam alignment.*README contains full configuration details, CLI output, and exam alignment.*
