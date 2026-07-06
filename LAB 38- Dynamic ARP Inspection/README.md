# Lab 38 – Dynamic ARP Inspection (DAI)

## Objective

This lab focuses on securing a small switched network against DHCP-based and ARP-based attacks using Cisco Layer 2 security features. The goals were to:

- Configure R1 as a DHCP server for the LAN
- Configure DHCP snooping on both access layer switches
- Configure Dynamic ARP Inspection (DAI) on both switches, including all additional validation checks
- Correctly identify and trust uplink ports connecting to routers or other switches

## Topology

- 1x Cisco 2911 router (R1)
- 2x Cisco 2960-24TT switches (SW1, SW2)
- 3x PCs (PC1, PC2, PC3)
- Single subnet: 192.168.1.0/24
- R1 Gi0/0 (192.168.1.1) connects to SW1 Gi0/2
- SW1 Gi0/1 connects to SW2 Gi0/1
- SW2 Fa0/1, Fa0/2, Fa0/3 connect to PC1, PC2, PC3

## VLAN Design

Single VLAN (VLAN 1) spanning both switches. No VLAN segmentation was required for this lab; the focus was strictly on Layer 2 security features rather than Layer 2 segmentation.

## Configuration Summary

**R1 – DHCP Server**

```
ip dhcp excluded-address 192.168.1.1 192.168.1.9
ip dhcp pool POOL1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
```

**SW1**

```
ip dhcp snooping
ip dhcp snooping vlan 1
no ip dhcp snooping information option

ip arp inspection vlan 1
ip arp inspection validate src-mac dst-mac ip

interface GigabitEthernet0/1
 ip arp inspection trust

interface GigabitEthernet0/2
 ip arp inspection trust
```

**SW2**

```
ip dhcp snooping
ip dhcp snooping vlan 1
no ip dhcp snooping information option

ip arp inspection vlan 1
ip arp inspection validate src-mac dst-mac ip

interface GigabitEthernet0/1
 ip arp inspection trust
```

Trust ports were assigned only to uplinks connecting to another switch or router (SW1 Gi0/1 and Gi0/2, SW2 Gi0/1). Downlinks to end hosts (SW2 Fa0/1 through Fa0/3) were left untrusted by default, as required for both DHCP snooping and DAI to inspect host-facing traffic.

## Verification

| PC | IPv4 Address | Subnet Mask | Default Gateway | DHCP Status |
|----|-------------|-------------|------------------|-------------|
| PC1 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 | DHCP request successful |
| PC2 | 192.168.1.11 | 255.255.255.0 | 192.168.1.1 | DHCP request successful |
| PC3 | 192.168.1.12 | 255.255.255.0 | 192.168.1.1 | DHCP request successful |

All three clients leased addresses outside the excluded range (192.168.1.1 to 192.168.1.9), confirming the DHCP pool and exclusion range were configured correctly.

`show running-config` on both SW1 and SW2 confirmed `ip arp inspection validate src-mac dst-mac ip` was applied globally rather than under an interface, and that `ip dhcp snooping`, `ip dhcp snooping vlan 1`, and `no ip dhcp snooping information option` were all present at the global level.

Additional verification commands used:

```
show ip arp inspection interfaces
show ip arp inspection vlan 1
```

## Troubleshooting Encountered
- Typo on SW1: entered `no dhcp ip dhcp snooping information option`, which was rejected. Corrected to `no ip dhcp snooping information option`.
- Typo on SW2: entered `ip arp inspection vlaidate ip src-mac dst-mac`, which was rejected. Corrected to `ip arp inspection validate ip src-mac dst-mac`.

## Key Observations

- PCs in Packet Tracer are GUI-only devices. There is no CLI equivalent to `ip address dhcp` for a PC; DHCP must be enabled through the Desktop > IP Configuration window.
- `ip arp inspection validate` must include every desired check (ip, src-mac, dst-mac) in a single command. Entering them separately causes each new command to overwrite the previous one.
- DHCP snooping requires two commands to fully activate: the global `ip dhcp snooping` and the per-VLAN `ip dhcp snooping vlan 1`. DAI only requires the per-VLAN `ip arp inspection vlan 1`, with no separate global enable.
- Trust port placement follows the same logic for both DHCP snooping and DAI: uplinks toward routers or other switches are trusted, downlinks toward end hosts are left untrusted.

## Skills Demonstrated

- Configuring a Cisco router as a DHCP server with an excluded address range
- Configuring DHCP snooping on multiple switches, including disabling Option 82 insertion
- Configuring Dynamic ARP Inspection with all additional validation checks in a single command
- Correctly identifying and configuring trusted uplink ports
- Verifying configuration through `show running-config`, DHCP client status, and DAI-specific show commands

## CCNA Exam Alignment

- **5.7** Configure and verify Layer 2 security features (DHCP snooping, Dynamic ARP Inspection, port security)
- **4.6** Configure and verify DHCP client and relay (DHCP server-side configuration)

## Files

- `README.md` – this file
- `Lab 38- Dynamic ARP Inspection.pkt` – Packet Tracer topology and configuration file
