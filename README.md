# CCNA-LABS

> Hands-on Cisco Packet Tracer labs built alongside active study for the **Cisco CCNA 200-301** certification.

This repository documents my practical lab work as I progress through the CCNA curriculum. Each lab is paired with study notes, CLI verification, and a write-up explaining not just *what* was configured — but *why* the network behaves the way it does.

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

### ✅ Layer 3 Core

| Lab | Topic | Protocol / Method | Status |
|-----|-------|-------------------|--------|
| Lab 06 | Static Routing | IPv4, Next-Hop, Default Route | ✅ Complete |
| Lab 07 | Subnetting (VLSM) | Variable-Length Subnet Masks | ✅ Complete |

### ✅ VLANs & Inter-VLAN Routing

| Lab | Topic | Protocol / Method | Status |
|-----|-------|-------------------|--------|
| Lab 08 | VLANs Part 1 — Access Ports | 802.1Q, Access Ports | ✅ Complete |
| Lab 09 | VLANs Part 2 — Trunking & ROAS | 802.1Q Trunking, Router-on-a-Stick | ✅ Complete |
| Lab 10 | VLANs Part 3 — Multilayer Switch & SVIs | SVIs, Layer 3 Switching | ✅ Complete |

### 🔲 Dynamic Routing *(Coming Soon)*

| Lab | Topic | Status |
|-----|-------|--------|
| Lab 11 | OSPFv2 (Single Area) | 🔲 Planned |

### 🔲 Services & Security *(Coming Soon)*

| Lab | Topic | Status |
|-----|-------|--------|
| Lab 12 | DHCP & DNS | 🔲 Planned |
| Lab 13 | NAT / PAT | 🔲 Planned |
| Lab 14 | Access Control Lists (ACLs) | 🔲 Planned |
| Lab 15 | Wireless Fundamentals | 🔲 Planned |

---

## Tools & Resources

| Tool | Purpose |
|------|---------|
| Cisco Packet Tracer | Network simulation and lab environment |
| Cisco IOS / IOS-XE CLI | Configuration and verification |
| Jeremy's IT Lab | Primary CCNA study resource |
| Anki Flashcards | Daily review of key concepts |

---

## Certification Target

**Cisco Certified Network Associate (CCNA) 200-301**

Exam domains covered by this lab series:

- **1.0 Network Fundamentals** — OSI model, Ethernet, switching, subnetting
- **2.0 Network Access** — VLANs, trunking, STP, RSTP, EtherChannel, SVIs
- **3.0 IP Connectivity** — Routing tables, static routes, OSPF
- **4.0 IP Services** — DHCP, NAT, NTP
- **5.0 Security Fundamentals** — ACLs, port security
- **6.0 Automation & Programmability** — Coming later

---

## Repository Structure

```
CCNA-LABS/
├── Lab 01 Inter VLAN Routing/
│   ├── README.md
│   └── Inter-VLAN-Routing-Router-on-a-Stick.pkt
├── Lab02-STP_ANALYSIS/
│   ├── README.md
│   └── Analyzing_STP_Lab.pkt
├── LAB 03- STP/
│   ├── README.md
│   └── Configuring_Spanning_Tree.pkt
├── Lab 04- Rapid STP/
│   ├── README.md
│   └── Rapid_STP_Lab.pkt
├── LAB 05-EtherChannel/
│   ├── README.md
│   └── LAB05-EtherChannel.pkt
├── LAB 06-Static Routing/
│   ├── README.md
│   └── Lab06-Static-Routing.pkt
├── LAB 07- VLSM/
│   ├── README.md
│   └── Lab07-VLSM.pkt
├── LAB 08-VLANs PT1/
│   ├── README.md
│   └── Day16_VLANs_Part1.pkt
├── Lab 09-VLANs PT2/
│   ├── README.md
│   └── Lab09- VLANs Pt2.pkt
├── LAB 10-VLANs PT3/
│   ├── README.md
│   └── Lab18- VLANs Pt3.pkt
└── README.md  ← you are here
```

---

*Updated as new labs are completed. Each lab README contains full configuration details, CLI output, and exam alignment.*
