# CCNA-LABS
 
## Lab 08 – VLANs Part 1 (Access Ports & Inter-VLAN Routing)
 
### Objective
Configure VLANs on a Cisco switch using access ports, connect a router with one physical interface per VLAN, and verify inter-VLAN routing and broadcast containment.
 
---
 
### Topology
- 1x Cisco 2911 Router (R1)
- 1x Cisco Switch (SW1)
- 6x End Hosts (PC1–PC6)
- Three separate physical connections between SW1 and R1 — one per VLAN
 
---
 
### VLAN Design
 
| VLAN | Name | Subnet | Gateway |
|-----:|------|--------|---------|
| 10 | ENGINEERING | 10.0.0.0/26 | 10.0.0.62 |
| 20 | HR | 10.0.0.64/26 | 10.0.0.126 |
| 30 | SALES | 10.0.0.128/26 | 10.0.0.190 |
 
---
 
### Host Addressing
 
| Host | VLAN | IP Address | Subnet Mask | Gateway |
|------|------|-----------|-------------|---------|
| PC1 | 10 | 10.0.0.1 | 255.255.255.192 | 10.0.0.62 |
| PC2 | 10 | 10.0.0.2 | 255.255.255.192 | 10.0.0.62 |
| PC3 | 20 | 10.0.0.65 | 255.255.255.192 | 10.0.0.126 |
| PC4 | 20 | 10.0.0.66 | 255.255.255.192 | 10.0.0.126 |
| PC5 | 30 | 10.0.0.129 | 255.255.255.192 | 10.0.0.190 |
| PC6 | 30 | 10.0.0.130 | 255.255.255.192 | 10.0.0.190 |
 
---
 
### Configuration Summary
 
**SW1 — VLAN Creation and Access Port Assignment:**
```
vlan 10
 name ENGINEERING
vlan 20
 name HR
vlan 30
 name SALES
 
interface fa3/1
 switchport mode access
 switchport access vlan 10
interface fa4/1
 switchport mode access
 switchport access vlan 10
interface fa5/1
 switchport mode access
 switchport access vlan 20
interface fa6/1
 switchport mode access
 switchport access vlan 20
interface fa7/1
 switchport mode access
 switchport access vlan 30
interface fa8/1
 switchport mode access
 switchport access vlan 30
 
interface gig0/1
 switchport mode access
 switchport access vlan 10
interface gig1/1
 switchport mode access
 switchport access vlan 20
interface gig2/1
 switchport mode access
 switchport access vlan 30
```
 
**R1 — One Interface Per VLAN:**
```
interface g0/0
 ip address 10.0.0.62 255.255.255.192
 no shutdown
interface g0/1
 ip address 10.0.0.126 255.255.255.192
 no shutdown
interface g0/2
 ip address 10.0.0.190 255.255.255.192
 no shutdown
```
 
---
 
### Verification
 
| Command | Purpose |
|---------|---------|
| `show vlan brief` | Confirm VLAN creation, names, and access port assignments |
| `show ip interface brief` | Verify router interface IPs and up/up status |
| `ping` | Validate inter-VLAN connectivity between hosts |
 
---
 
### Troubleshooting Encountered
 
**Issue:** Inter-VLAN pings failing — 100% packet loss from PC1 to PC3
 
**Root Cause:** The switch uplink ports connecting to R1 (Gig0/1, Gig1/1, Gig2/1) were left in the default VLAN 1. Traffic from VLAN 10 PCs could not reach R1's G0/0 interface because the uplink was in a different VLAN.
 
**Fix:** Assigned each uplink port to its corresponding VLAN using `switchport access vlan X`.
 
**Key Lesson:** In a legacy inter-VLAN routing design (one physical interface per VLAN), the switch port connecting to each router interface must be in the same VLAN as the hosts that router interface serves. Leaving uplinks in VLAN 1 breaks routing even if the router IPs and host configs are correct.
 
---
 
### Key Observations
- Assigning a port to a VLAN that doesn't exist automatically creates it
- `show vlan brief` only shows access ports — trunk ports do not appear here
- VLANs 1 and 1002–1005 exist by default and cannot be deleted
- First ping often fails in Packet Tracer due to ARP resolution — second ping succeeds
- Broadcast traffic was confined to each VLAN — confirmed via Simulation Mode
- `interface range` with a dash only works for interfaces on the same module (e.g. fa0/1 - fa0/5). For interfaces on different modules, use commas (e.g. fa3/1, fa4/1, fa5/1)
 
---
 
### Skills Demonstrated
- VLAN creation and naming on a Cisco switch
- Access port configuration and VLAN assignment
- Legacy inter-VLAN routing (one physical interface per VLAN)
- Static IP addressing and default gateway configuration
- Troubleshooting VLAN membership issues on uplink ports
- Broadcast containment verification using Simulation Mode
 
---
 
### CCNA Exam Alignment
- **2.1** – Configure and verify VLANs spanning multiple switches
- **2.1a** – Access ports (data and voice)
- **2.1c** – InterVLAN connectivity
 
---
 
### Files
- `Day16_VLANs_Part1.pkt`
