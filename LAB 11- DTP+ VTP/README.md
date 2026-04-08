# CCNA-LABS
 
## Lab 11 – DTP & VTP (Dynamic Trunking Protocol & VLAN Trunking Protocol)
 
### Objective
 
Configure and verify Dynamic Trunking Protocol (DTP) and VLAN Trunking Protocol (VTP) behavior across a three-switch topology. Demonstrate VTP server, transparent, and client modes; observe VLAN propagation and isolation; and confirm DTP behavior on access ports.
 
---
 
### Topology
 
- 3x Cisco 2960-24TT Switches (SW1, SW2, SW3)
- 9x End Hosts (PC1–PC9)
- Inter-switch links: G0/1 (SW1↔SW2), G0/2 (SW2↔SW3), G0/1 (SW2↔SW3)
 
```
PC1 ─┐                          ┌─ PC5
PC2 ─┤  SW1 ──G0/1── SW2 ──G0/1── SW3 ─┤
     │         G0/2─────────────┘       └─ PC6
     └─ PC8 (VLAN20)                       PC7
```
 
---
 
### VLAN Design
 
| VLAN | Name   | Subnet          | Switch Scope         |
|-----:|--------|-----------------|----------------------|
| 10   | VLAN10 | 10.0.0.0/26     | SW1, SW2, SW3        |
| 20   | VLAN20 | 10.0.0.64/26    | SW1, SW2, SW3        |
| 30   | VLAN30 | 10.0.0.128/26   | SW1, SW2, SW3        |
| 40   | VLAN40 | 10.0.0.192/26   | SW2 only (transparent) |
 
---
 
### Configuration Summary
 
#### Step 1 – Configure Trunk Ports and Disable DTP
 
```
! SW1
interface GigabitEthernet0/1
 switchport mode trunk
 switchport nonegotiate
 
! SW2
interface range GigabitEthernet0/1 - 2
 switchport mode trunk
 switchport nonegotiate
 
! SW3
interface GigabitEthernet0/1
 switchport mode trunk
 switchport nonegotiate
```
 
#### Step 2 – Configure VTP Domain and Create VLANs on SW1
 
```
! SW1
vtp domain CCNA
 
vlan 10
 name VLAN10
vlan 20
 name VLAN20
vlan 30
 name VLAN30
```
 
#### Step 3 – Configure SW2 as VTP Transparent and Add VLAN40
 
```
! SW2
vtp mode transparent
 
vlan 40
 name VLAN40
```
 
#### Step 4 – Configure SW3 as VTP Client
 
```
! SW3
vtp mode client
```
 
#### Step 5 – Configure Access Ports
 
```
! SW1
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 10
 
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 20
 
! SW2
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 40
 
! SW3
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 
interface range FastEthernet0/2 - 3
 switchport mode access
 switchport access vlan 30
 
interface FastEthernet0/4
 switchport mode access
 switchport access vlan 20
```
 
---
 
### Verification
 
| Check | Command | Expected Result |
|-------|---------|-----------------|
| Trunk status | `show interfaces g0/1 switchport` | Admin: trunk, Oper: trunk, Negotiation: Off |
| VTP status | `show vtp status` | Domain: CCNA, Mode per switch |
| VLAN database | `show vlan brief` | VLANs 10/20/30 on SW1 and SW3; VLAN40 on SW2 only |
| DTP on access port | `show interfaces f0/1 switchport` | Negotiation of trunking: Off |
 
---
 
### Troubleshooting Encountered
 
| Issue | Cause | Resolution |
|-------|-------|------------|
| `write memory` rejected | Command entered while still in interface config mode | Exited to privileged EXEC mode first |
| VLAN name with space rejected | `name VLAN 10` — spaces not allowed in VLAN names | Used `name VLAN10` |
| `switchport nonegotiate` typo | Typed `nonegitiate` on SW3 | Re-entered correct spelling |
| PVST BPDU error on SW2 during trunk config | One side of trunk configured before the other | Resolved automatically once both sides were set to trunk |
 
---
 
### Key Observations
 
- `switchport mode trunk` alone does **not** disable DTP — `switchport nonegotiate` is required as a second command
- VTP server (SW1) propagated VLANs 10, 20, and 30 to SW2 and SW3 automatically via trunk links — no manual configuration required on those switches
- VTP transparent mode (SW2) does **not** advertise its own VLANs — VLAN40 was visible only on SW2; SW1 and SW3 never received it
- VTP client mode (SW3) prevents local VLAN creation — the switch rejects `vlan 50` with: `VTP VLAN configuration not allowed when device is in CLIENT mode`
- Manually configuring `switchport mode access` on a port automatically disables DTP on that port
 
---
 
### Skills Demonstrated
 
- DTP mode configuration and disabling DTP with `switchport nonegotiate`
- VTP domain configuration and VLAN propagation verification
- VTP mode differences: server, transparent, and client behavior
- Access port assignment per VLAN across multiple switches
- CLI verification of trunk and DTP status
 
---
 
### CCNA Exam Alignment
 
| Exam Topic | Description |
|------------|-------------|
| 2.1 | Configure and verify VLANs spanning multiple switches |
| 2.2 | Configure and verify interswitch connectivity (trunk ports, 802.1Q, native VLAN) |
| 2.2a | Trunk port configuration and verification |
 
---
 
### Files
 
- `Lab11-DTP_VTP.pkt`
