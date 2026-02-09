# CCNA-LABS

## Lab 01 – Inter-VLAN Routing (Router-on-a-Stick)

### Objective
Design and validate a segmented Layer 2 network using multiple VLANs and enable Layer 3 communication using a router-on-a-stick architecture.

### Topology
- Cisco 2960 Switch
- Cisco 2911 Router
- Two end hosts in separate VLANs
- 802.1Q trunk between switch and router

### VLAN Design
| VLAN | Name  | Subnet |
|-----:|------|--------|
| 10 | USER | 192.168.10.0/24 |
| 20 | TEST | 192.168.20.0/24 |
| 99 | MGMT | 192.168.99.0/24 |

### Key Configurations
- VLAN creation and access port assignment
- 802.1Q trunk with allowed VLANs
- Router subinterfaces with dot1Q encapsulation
- Static IP addressing on hosts
- Default gateways mapped to router subinterfaces

### Validation & Testing
- Verified trunk status and VLAN propagation
- Confirmed intra-VLAN connectivity
- Observed expected inter-VLAN behavior
- Used ICMP (ping) testing to validate routing

### Skills Demonstrated
- VLAN segmentation
- 802.1Q trunking
- Inter-VLAN routing
- Network troubleshooting
- CCNA-level network design principles

### Files
- `Inter-VLAN-Routing-Router-on-a-Stick.pkt`


Spanning Tree Protocol (STP) – Packet Tracer Lab
Overview
This lab is based on a guided Spanning Tree Protocol (STP) exercise from Jeremy’s IT Lab, part of a free and comprehensive CCNA (200-301) preparation series. The goal of the lab was to analyze and verify STP behavior in a multi-switch topology using Cisco Packet Tracer, with a focus on understanding root bridge election, port roles, and STP verification via the CLI.
This lab emphasizes observation and reasoning rather than configuration, which aligns with the CCNA exam objectives for STP.
Objectives
By completing this lab, I was able to:
Identify the root bridge based on bridge ID (priority + MAC address)
Determine root ports, designated ports, and non-designated (blocking) ports
Apply STP tie-breaker rules (root cost, neighbor bridge ID, neighbor port ID)
Verify spanning tree behavior using Cisco IOS CLI commands
Interpret STP output rather than relying on visual indicators (link lights)
Lab Environment
Tool: Cisco Packet Tracer
Topology: Four interconnected Layer 2 switches
VLANs: Single VLAN (VLAN 1)
STP Mode: IEEE / PVST (classic spanning tree)
To prevent visual bias, link lights were disabled so that forwarding vs. blocking states had to be determined logically and confirmed via CLI.
STP Analysis Process
1. Root Bridge Identification
The root bridge was determined by comparing bridge IDs, which consist of:
Bridge priority
MAC address (used as a tiebreaker)
In this topology, SW3 had the lowest bridge priority (24577) and was therefore elected as the root bridge.
Key rule applied:
The switch with the lowest bridge ID becomes the root bridge.
Because SW3 is the root bridge, all of its interfaces are designated ports and are in a forwarding state.
2. Root Port Selection
Each non-root switch must have exactly one root port, defined as the interface with the lowest total path cost to the root bridge.
When multiple paths had equal cost, STP tie-breakers were applied in this order:
Lowest root path cost
Lowest neighbor bridge ID
Lowest neighbor port ID
Using these rules:
SW1 selected F0/4 as its root port
SW2 selected G0/1 as its root port
SW4 selected G0/2 as its root port
3. Designated vs. Non-Designated Ports
For each remaining shared segment (collision domain), STP determines:
One designated port (forwards traffic)
One non-designated port (blocks to prevent loops)
The switch with the lower root path cost wins the designated role.
As a result:
SW2’s F0/1 and F0/2 were designated
SW1’s F0/1 and F0/2 became non-designated (blocking)
Blocking ports were not errors — they were intentional loop-prevention mechanisms.
CLI Verification
The following commands were used to confirm STP behavior:
show spanning-tree
show spanning-tree vlan 1
show spanning-tree detail
show spanning-tree summary
Key Observations
The root bridge correctly identified itself as the root in the Root ID section
Port roles matched the calculated results (root, designated, alternate/blocking)
The show spanning-tree detail command confirmed total root path cost
Default STP timers were observed:
Hello Time: 2 seconds
Max Age: 20 seconds
Forward Delay: 15 seconds
CCNA Exam Relevance
This lab directly supports CCNA 200-301 Objective 2.5:
Describe the need for and basic operations of Rapid PVST+ spanning tree protocol.
Although STP configuration is not required for the CCNA exam, the ability to:
Identify root bridges
Interpret port roles
Read and understand show spanning-tree output
is essential.
Key Takeaways
STP prevents Layer 2 loops by creating a logical tree topology
Blocking ports are a normal and healthy part of a switched network
Root port and designated port selection follows strict, predictable rules
CLI verification is critical — visual indicators alone are not reliable
This knowledge becomes foundational for advanced topics (RSTP, EtherChannel, CCNP)
Credits
Jeremy’s IT Lab – CCNA 200-301 Free Course
Cisco Packet Tracer
Cisco IOS CLI
