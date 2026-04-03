# CCNA-LABS
 
## Lab 09 – VLANs Part 2 (Trunking & Router-on-a-Stick)
 
### Objective
Configure VLANs across two switches using a trunk link, then enable inter-VLAN routing using Router-on-a-Stick (ROAS) with subinterfaces on a single router interface.
 
---
 
### Topology
- 1x Cisco 2911 Router (R1)
- 2x Cisco 2960-24TT Switches (SW1, SW2)
- 7x End Hosts (PC1–PC7)
- Trunk link between SW1 and SW2
- Trunk link between SW2 and R1 (ROAS)
 
---
 
### VLAN Design
 
| VLAN | Subnet | Gateway (Last Usable) |
|-----:|--------|----------------------|
| 10 | 10.0.0.0/26 | 10.0.0.62 |
| 20 | 10.0.0.64/26 | 10.0.0.126 |
| 30 | 10.0.0.128/26 | 10.0.0.190 |
 
---
 
### Configuration Summary
 
**SW1 — Access Ports:**
```
interface range fa0/1-2
 switchport mode access
 switchport access vlan 10
 
interface range fa0/3-4
 switchport mode access
 switchport access vlan 30
 
vlan 20
```
 
**SW1 — Trunk to SW2:**
```
interface gig0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 switchport trunk native vlan 27
```
 
**SW2 — Access Ports:**
```
interface fa0/1
 switchport mode access
 switchport access vlan 20
 
interface range fa0/2-3
 switchport mode access
 switchport access vlan 10
 
vlan 30
```
 
**SW2 — Trunk to SW1:**
```
interface gig0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 switchport trunk native vlan 27
```
 
**SW2 — Trunk to R1:**
```
interface gig0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
```
 
**R1 — Router-on-a-Stick:**
```
interface g0/0
 no shutdown
 
interface g0/0.10
 encapsulation dot1q 10
 ip address 10.0.0.62 255.255.255.192
 
interface g0/0.20
 encapsulation dot1q 20
 ip address 10.0.0.126 255.255.255.192
 
interface g0/0.30
 encapsulation dot1q 30
 ip address 10.0.0.190 255.255.255.192
```
 
---
 
### Verification
 
| Command | Purpose |
|---------|---------|
| `show vlan brief` | Confirm VLAN existence and access port assignments on each switch |
| `show interfaces trunk` | Verify trunk ports, encapsulation, native VLAN, allowed VLANs |
| `show ip interface brief` | Confirm subinterface IPs and up/up status on R1 |
| `ping` | Validate inter-VLAN connectivity between hosts |
 
---
 
### Troubleshooting Encountered
 
**Issue 1 — Native VLAN mismatch warning**
 
CDP reported a native VLAN mismatch when SW1 trunk was configured before SW2. Both sides were set to native VLAN 27 to resolve. The mismatch warning disappeared once both ends matched.
 
**Issue 2 — Pings timing out after ROAS configured**
 
After configuring R1 subinterfaces, pings from PC1 to its own default gateway were timing out.
 
**Root Cause:** The trunk between SW2 and R1 (G0/2 on SW2) was never configured. SW2 only had one trunk — to SW1. R1 was receiving no tagged frames because the SW2 uplink to R1 was still an access port in VLAN 1.
 
**Fix:** Configured G0/2 on SW2 as a trunk with `switchport mode trunk` and allowed VLANs 10, 20, 30.
 
**Key Lesson:** In a ROAS topology, the switch port connecting to the router must be configured as a trunk — not just the switch-to-switch link. Forgetting the router-facing trunk is a common misconfiguration that completely breaks inter-VLAN routing even when everything else is correct.
 
---
 
### Key Observations
- `switchport trunk encapsulation dot1q` was rejected on SW1 — it only supports dot1q so the command is not needed, just use `switchport mode trunk`
- All necessary VLANs must exist on both switches even if no hosts in that VLAN are directly connected — required for trunk to carry them correctly
- TTL=127 on successful inter-VLAN pings confirms traffic is passing through the router (started at 128, decremented once)
- First ping in each new flow drops due to ARP resolution — normal behavior in Packet Tracer
 
---
 
### Skills Demonstrated
- VLAN configuration across multiple switches
- 802.1Q trunk configuration with allowed VLANs and native VLAN
- Router-on-a-Stick subinterface configuration
- Multi-switch inter-VLAN routing verification
- Troubleshooting missing trunk on router-facing switch port
 
---
 
### CCNA Exam Alignment
- **2.1** – Configure and verify VLANs spanning multiple switches
- **2.2** – Configure and verify interswitch connectivity (trunk ports, 802.1Q, native VLAN)
- **2.1c** – InterVLAN connectivity
 
---
 
### Files
- `Lab09- VLANs Pt2.pkt`
