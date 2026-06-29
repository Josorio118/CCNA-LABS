# CCNA-LABS

## Lab 34 – Voice VLANs and ROAS

### Objective

Configure voice VLANs on SW1's access ports to separate PC data traffic (VLAN 10) from IP phone voice traffic (VLAN 20). Configure Router-on-a-Stick on R1 to provide inter-VLAN routing, then use simulation mode to verify that PC traffic is untagged and phone traffic is tagged with a VLAN ID.

---

### Topology

- 1x Cisco 3650-24PS Switch (SW1) — Layer 3 switch used as Layer 2 only; supports PoE
- 1x Cisco 2811 Router (R1) — ROAS with pre-configured telephony settings
- 2x IP Phone (PH1, PH2) — powered via PoE from SW1
- 2x PC (PC1, PC2) — connected through the IP phones to SW1

```
PC1 (.11) - PH1 - G1/0/2 - SW1 - G1/0/1 - F0/0 - R1
PC2 (.12) - PH2 - G1/0/3 - SW1
```

| VLAN | Purpose | Subnet |
|------|---------|--------|
| VLAN 10 | Data (PCs) | 192.168.10.0/24 |
| VLAN 20 | Voice (IP phones) | 192.168.20.0/24 |

---

### Configuration Summary

#### Step 1: Configure SW1 Interfaces

```
! Access ports for phones and PCs
SW1(config)# vlan 10
SW1(config)# interface g1/0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# switchport voice vlan 20

SW1(config)# interface g1/0/3
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# switchport voice vlan 20

! Trunk port toward R1
SW1(config)# interface g1/0/1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan 10,20
```

#### Step 2: Configure ROAS on R1

```
R1(config)# interface f0/0
R1(config-if)# no shutdown

R1(config)# interface f0/0.10
R1(config-subif)# encapsulation dot1q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0

R1(config)# interface f0/0.20
R1(config-subif)# encapsulation dot1q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
```

#### Steps 3 and 4: Simulation Mode Verification

- Pinged PC2 from PC1 and inspected outbound PDU details
- Called PH1 from PH2 and inspected outbound PDU details

---

### Verification

| Test | Expected Result |
|------|-----------------|
| PC1 ping PC2 (simulation mode) | No 802.1Q tag in Ethernet header; ICMP traffic is untagged |
| PH2 call to PH1 (simulation mode) | 802.1Q tag present; TPID 0x8100, TCI 0x0014 (VLAN 20 = decimal 20) |
| IP phone registration | Phones auto-register after ROAS configured; confirmed via IPPHONE-6-REGISTER syslog |

---

### Troubleshooting Encountered

- `switchport trunk encapsulation dot1q` rejected on SW1 (3650-24PS in Packet Tracer does not support the command); went straight to `switchport mode trunk` which worked
- `encapsulation dot 1q` (with space) rejected; correct syntax is `encapsulation dot1q` with no space
- `encapsulation dot1q` without VLAN number rejected as incomplete command; must specify VLAN ID

---

### Key Observations

- PC traffic sent over the access port is untagged; no 802.1Q header present in the Ethernet frame
- Phone traffic is tagged with VLAN 20; TPID 0x8100 and TCI 0x0014 (hex 14 = decimal 20) confirmed in PDU details
- The interface remains an ACCESS port even with a voice VLAN configured; `show interfaces trunk` shows it as not-trunking
- SW1 uses CDP to tell the IP phones to tag their traffic in VLAN 20
- IP phones auto-registered (ephone-1 and ephone-2) immediately after ROAS was configured, because the phones could now reach R1's telephony service on VLAN 20
- VLAN 20 was created automatically by Packet Tracer when `switchport voice vlan 20` was entered before VLAN 20 existed
- The 3650-24PS supports PoE, powering the IP phones over the Ethernet cable without a separate power cable

---

### Skills Demonstrated

- Voice VLAN configuration using `switchport voice vlan`
- Data VLAN and voice VLAN separation on a single access port
- Trunk configuration allowing specific VLANs on the uplink
- ROAS subinterface configuration on a router for inter-VLAN routing
- Simulation mode PDU analysis to confirm tagging behavior
- Understanding of how IP phones use an internal 3-port switch to share a switchport with a PC

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 4.6 | Configure and verify DHCP client and relay |
| 2.2 | Configure and verify VLANs spanning multiple switches |
| 4.7 | Explain the forwarding per-hop behavior (PHB) for QoS |

---

### Files

- `Lab34-Voice-VLANs.pkt`
