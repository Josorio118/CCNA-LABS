# Lab 17 – Integrated Network Design

## Objective

Design and implement a fully integrated enterprise network from scratch, incorporating VLANs, EtherChannel, VTP, STP, OSPF, static routing, and a simulated ISP connection. This lab serves as a cumulative capstone covering all major topics completed through OSPFv2.

---

## Topology

| Device | Type | Role |
|--------|------|------|
| R1 | Cisco 2911 | VLAN 10 gateway, OSPF ABR, ISP-facing router |
| R2 | Cisco 2911 | VLAN 20 gateway, OSPF internal router |
| R3 | Cisco 2911 | VLAN 30 gateway, OSPF internal router |
| ISP | Cisco 2911 | Simulated ISP (serial connection to R1) |
| SW1 | Cisco 2960 | Layer 2 access switch, VTP Server, STP Root |
| SW2 | Cisco 2960 | Layer 2 access switch, VTP Client |
| SW3 | Cisco 2960 | Layer 2 access switch, VTP Client |
| PC0–PC3 | PC-PT | VLAN 10 end hosts (Corporate) |
| PC4–PC7 | PC-PT | VLAN 20 end hosts (Engineering) |
| PC8–PC11 | PC-PT | VLAN 30 end hosts (Management) |

---

## VLAN Design

| VLAN | Name | Subnet | Hosts | Gateway |
|------|------|--------|-------|---------|
| 10 | CORPORATE | 192.168.10.0/23 | 510 usable | 192.168.10.1 (R1) |
| 20 | ENGINEERING | 192.168.20.0/25 | 126 usable | 192.168.20.1 (R2) |
| 30 | MANAGEMENT | 192.168.30.0/26 | 62 usable | 192.168.30.1 (R3) |

---

## IP Addressing

### Point-to-Point Links

| Network | Subnet | R1 | R2 | R3 |
|---------|--------|----|----|----|
| R1 ↔ R2 | 10.0.0.0/30 | 10.0.0.1 (G0/0) | 10.0.0.2 (G0/0) | — |
| R1 ↔ R3 | 10.0.0.8/30 | 10.0.0.10 (G0/2) | — | 10.0.0.9 (G0/2) |
| R2 ↔ R3 | 10.0.0.4/30 | — | 10.0.0.5 (G0/1) | 10.0.0.6 (G0/1) |
| ISP ↔ R1 | 203.0.113.0/30 | 203.0.113.2 (S0/0/0) | — | — |

### Loopbacks

| Device | Loopback0 |
|--------|-----------|
| R1 | 1.1.1.1/32 |
| R2 | 2.2.2.2/32 |
| R3 | 3.3.3.3/32 |

---

## EtherChannel Design

| Port-Channel | SW1 Ports | SW2/SW3 Ports | Protocol | Type |
|-------------|-----------|---------------|----------|------|
| SW1 ↔ SW2 Po1 | Fa0/7-8 | Fa0/7-8 | LACP (active/active) | Layer 2 Trunk |
| SW1 ↔ SW3 Po2 | Fa0/9-10 | Fa0/9-10 | LACP (active/active) | Layer 2 Trunk |
| SW2 ↔ SW3 Po1/Po2 | — | Fa0/9-10 ↔ Fa0/7-8 | LACP (active/active) | Layer 2 Trunk |

---

## Configuration Summary

### EtherChannel (All Switches)

```
interface range fa0/7-8
 channel-group 1 mode active

interface range fa0/9-10
 channel-group 2 mode active

interface port-channel 1
 switchport mode trunk

interface port-channel 2
 switchport mode trunk
```

### VTP

```
! SW1 — Server
vtp mode server
vtp domain CCNA

! SW2, SW3 — Clients
vtp mode client
```

### VLAN Creation (SW1 only — propagated via VTP)

```
vlan 10
 name CORPORATE
vlan 20
 name ENGINEERING
vlan 30
 name MANAGEMENT
```

### Access Port Assignment

```
! SW1 — VLAN 10
interface range fa0/2-5
 switchport mode access
 switchport access vlan 10

! SW2 — VLAN 20
interface range fa0/3-6
 switchport mode access
 switchport access vlan 20

! SW3 — VLAN 30
interface range fa0/11-14
 switchport mode access
 switchport access vlan 30
```

### STP Root Bridge (SW1)

```
spanning-tree vlan 1,10,20,30 root primary
```

### PortFast & BPDU Guard

```
! SW1
interface range fa0/2-5
 spanning-tree portfast
 spanning-tree bpduguard enable

! SW2
interface range fa0/3-6
 spanning-tree portfast
 spanning-tree bpduguard enable

! SW3
interface range fa0/11-14
 spanning-tree portfast
 spanning-tree bpduguard enable
```

### Router IP Addressing

```
! R1
interface g0/1
 ip address 192.168.10.1 255.255.254.0
 no shutdown

interface g0/0
 ip address 10.0.0.1 255.255.255.252
 no shutdown

interface g0/2
 ip address 10.0.0.10 255.255.255.252
 no shutdown

interface s0/0/0
 ip address 203.0.113.2 255.255.255.252
 no shutdown

interface loopback0
 ip address 1.1.1.1 255.255.255.255

! R2
interface g0/2
 ip address 192.168.20.1 255.255.255.128
 no shutdown

interface g0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown

interface g0/1
 ip address 10.0.0.5 255.255.255.252
 no shutdown

interface loopback0
 ip address 2.2.2.2 255.255.255.255

! R3
interface g0/0
 ip address 192.168.30.1 255.255.255.192
 no shutdown

interface g0/1
 ip address 10.0.0.6 255.255.255.252
 no shutdown

interface g0/2
 ip address 10.0.0.9 255.255.255.252
 no shutdown

interface loopback0
 ip address 3.3.3.3 255.255.255.255
```

### OSPF (All Routers — Area 0)

```
! R1
router ospf 1
 router-id 1.1.1.1
 auto-cost reference-bandwidth 10000
 passive-interface default
 no passive-interface g0/0
 no passive-interface g0/2
 no passive-interface s0/0/0
 network 192.168.10.0 0.0.1.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
 network 10.0.0.8 0.0.0.3 area 0
 network 1.1.1.1 0.0.0.0 area 0
 default-information originate

! R2
router ospf 1
 router-id 2.2.2.2
 auto-cost reference-bandwidth 10000
 passive-interface default
 no passive-interface g0/0
 no passive-interface g0/1
 network 192.168.20.0 0.0.0.127 area 0
 network 10.0.0.0 0.0.0.3 area 0
 network 10.0.0.4 0.0.0.3 area 0
 network 2.2.2.2 0.0.0.0 area 0

! R3
router ospf 1
 router-id 3.3.3.3
 auto-cost reference-bandwidth 10000
 passive-interface default
 no passive-interface g0/1
 no passive-interface g0/2
 network 192.168.30.0 0.0.0.63 area 0
 network 10.0.0.4 0.0.0.3 area 0
 network 10.0.0.8 0.0.0.3 area 0
 network 3.3.3.3 0.0.0.0 area 0
```

### Default Route & Floating Static (R1)

```
ip route 0.0.0.0 0.0.0.0 203.0.113.1
ip route 0.0.0.0 0.0.0.0 203.0.113.5 200
```

### ISP Configuration

```
interface s0/0/0
 ip address 203.0.113.1 255.255.255.252
 clock rate 64000
 no shutdown

interface loopback0
 ip address 203.0.113.5 255.255.255.252
```

---

## Verification

| Test | Command | Expected Result |
|------|---------|-----------------|
| EtherChannel status | `show etherchannel summary` | All port-channels SU, member ports P |
| Trunk ports | `show interfaces trunk` | Po1, Po2 trunking on all switches |
| VTP propagation | `show vlan brief` on SW2/SW3 | VLANs 10, 20, 30 present |
| STP root | `show spanning-tree` on SW1 | "This bridge is the root" all VLANs |
| OSPF neighbors | `show ip ospf neighbor` | All neighbors FULL state |
| OSPF routes | `show ip route ospf` | All remote networks learned via OSPF |
| End-to-end ping | PC0 → PC4 (VLAN 10 → 20) | Success after ARP |
| End-to-end ping | PC0 → PC8 (VLAN 10 → 30) | Success after ARP |
| Default route | `show ip route` on R1 | S* 0.0.0.0/0 via 203.0.113.1 |
| Floating static | `show running-config` on R1 | Two static routes, AD 1 and AD 200 |

---

## Troubleshooting Encountered

| Issue | Symptom | Resolution |
|-------|---------|------------|
| Router-facing switch ports set as trunks | Inter-VLAN routing not needed since each router serves one VLAN | Changed to access ports assigned to respective VLANs |
| `channel group` (no hyphen) | `% Ambiguous command` error on SW3 | Corrected to `channel-group` |
| Floating static typed in OSPF config mode | Route not applied | Re-entered in global config mode |
| Serial IP typed as /32 | `Bad mask` error | Corrected to 255.255.255.252 |

---

## Key Observations

- OSPF adjacencies formed automatically once network statements matched the point-to-point subnets on both ends
- First ICMP packet in each ping timed out due to ARP resolution — expected behavior
- VTP propagated VLANs 10, 20, 30 to SW2 and SW3 immediately after trunks were established
- STP elected SW1 as root bridge automatically based on lowest MAC — manually locked in with `root primary`
- Floating static at AD 200 does not appear in `show ip route` while primary AD 1 route is active — confirmed in `show running-config`
- BPDU Guard warning messages on PortFast ports are normal — Packet Tracer reminder only

---

## Skills Demonstrated

- VLAN design with VLSM subnetting (/23, /25, /26)
- EtherChannel (LACP) configuration and verification
- VTP server/client deployment and VLAN propagation
- STP root bridge manipulation
- PortFast and BPDU Guard on access ports
- OSPFv2 single-area configuration with passive interfaces and reference bandwidth
- Default route advertisement into OSPF via `default-information originate`
- Floating static route as backup path (AD 200)
- Serial WAN link configuration with clock rate
- End-to-end connectivity verification across three VLANs

---

## CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 2.1 | Configure and verify VLANs spanning multiple switches |
| 2.2 | Configure and verify interswitch connectivity (trunking, 802.1Q) |
| 2.4 | Configure and verify EtherChannel (LACP) |
| 2.5 | Interpret basic operations of Rapid PVST+ STP |
| 3.2 | Determine how a router makes a forwarding decision |
| 3.3 | Configure and verify IPv4 static routing (default, floating) |
| 3.4 | Configure and verify single area OSPFv2 |
| 1.6 | Configure and verify IPv4 addressing and subnetting |

---

## Files

- `Lab17-Integrated_Network_Design.pkt`
