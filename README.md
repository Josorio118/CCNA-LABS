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

### 🔲 Coming Next

| Lab | Topic | Status |
|-----|-------|--------|
| Lab 16 | OSPFv2 Part 3 — DR/BDR | 🔲 Planned |
| Lab 17 | DHCP & DNS | 🔲 Planned |
| Lab 18 | NAT / PAT | 🔲 Planned |
| Lab 19 | Access Control Lists (ACLs) | 🔲 Planned |
| Lab 20 | Wireless Fundamentals | 🔲 Planned |

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

**Cisco Certified Network Associate (CCNA) 200-301**

Exam domains covered by this lab series:

- **1.0 Network Fundamentals** — OSI model, Ethernet, switching, subnetting
- **2.0 Network Access** — VLANs, trunking, STP, RSTP, EtherChannel, SVIs, DTP, VTP
- **3.0 IP Connectivity** — Routing tables, static routes, floating static routes, EIGRP, OSPFv2
- **4.0 IP Services** — DHCP, NAT, NTP *(coming)*
- **5.0 Security Fundamentals** — ACLs, port security *(coming)*
- **6.0 Automation & Programmability** — *(coming)*

---

## Repository Structure

```
CCNA-LABS/
├── LAB 01- Inter VLAN Routing/
│   ├── README.md
│   └── Lab01-Inter_VLAN_Routing.pkt
├── LAB 02-STP_ANALYSIS/
│   ├── README.md
│   └── Lab02-STP_Analysis.pkt
├── LAB 03- STP/
│   ├── README.md
│   └── Lab03-STP_Root_Manipulation.pkt
├── LAB 04- Rapid STP/
│   ├── README.md
│   └── Lab04-Rapid_STP.pkt
├── LAB 05-EtherChannel/
│   ├── README.md
│   └── Lab05-EtherChannel.pkt
├── LAB 06-Static Routing/
│   ├── README.md
│   └── Lab06-Static_Routing.pkt
├── LAB 07- VLSM/
│   ├── README.md
│   └── Lab07-VLSM.pkt
├── LAB 08-VLANs PT1/
│   ├── README.md
│   └── Lab08-VLANs_PT1.pkt
├── LAB 09-VLANs PT2/
│   ├── README.md
│   └── Lab09-VLANs_PT2.pkt
├── LAB 10-VLANs PT3/
│   ├── README.md
│   └── Lab10-VLANs_PT3.pkt
├── LAB 11- DTP+ VTP/
│   ├── README.md
│   └── Lab11-DTP_VTP.pkt
├── LAB 12- Dynamic Routing/
│   ├── README.md
│   └── Lab12-Floating_Static_Routes.pkt
├── LAB 13-EIGRP Config/
│   ├── README.md
│   └── Lab13-EIGRP_Configuration.pkt
├── LAB 14- OSPF PT1/
│   ├── README.md
│   └── Lab14-OSPF_Part1.pkt
├── LAB 15- OSPF PT2/
│   ├── README.md
│   └── Lab15-OSPF_Part2.pkt
└── README.md  ← you are here
```

---

*Updated as new labs are completed. Each lab README contains full configuration details, CLI output, and exam alignment.*
