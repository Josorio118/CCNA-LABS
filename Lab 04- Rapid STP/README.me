LAB 04 – Rapid Spanning Tree Protocol (RSTP) Analysis
Objective
This lab focuses on analyzing Rapid Spanning Tree Protocol (RSTP) behavior in a multi-switch topology. The goal was to:
Identify the root bridge
Determine port roles and states without using the CLI
Verify results using show spanning-tree
Manually configure appropriate RSTP link types
Configure edge ports using PortFast
This lab reinforces understanding of:
RSTP port roles (Root, Designated, Alternate, Backup)
RSTP port states (Forwarding, Discarding)
RSTP link types (Point-to-Point, Shared, Edge)
Differences between classic STP and RSTP
Topology Overview
4x Cisco 2960 switches
1x hub connected to SW1 and SW3
Multiple PCs connected to access ports
Single VLAN (VLAN 1)
All switches priority: 32769 (32768 + VLAN 1)
Because all priorities were equal, root election was determined by lowest MAC address.
Step 1 – Root Bridge Identification
Using:
show spanning-tree
Root Bridge: SW1
SW1 had the lowest MAC address:
Root ID Priority 32769
Address 0005.5E4E.714B
This bridge is the root
Important Observation
On SW1:
Fa0/3  Back  BLK  Shr
This is a Backup port in RSTP.
In classic STP, this would have been a non-designated port.
In RSTP, backup ports occur when two interfaces connect to the same collision domain (hub scenario).
Key correction to the common rule:
The root bridge has one designated port per collision domain.
Because SW1 had multiple connections into the same hub, only one interface could be designated — the other became a backup port.
Step 2 – Port Role and State Analysis (Without CLI First)
Using RSTP rules:
SW2
Fa0/1 → Root Port (lowest cost to root)
Fa0/2 → Designated
Fa0/23–24 → Designated (PC connections)
Gi0/1 → Alternate (discarding)
Verified:
Gi0/1  Altn BLK  P2p
SW3
Fa0/2 → Root Port (hub does not add cost)
Fa0/1 → Designated
Gi0/1 → Designated (lower bridge ID vs SW2)
Fa0/24 → Designated (PC)
Verified:
Fa0/2 Root FWD Shr
SW4
Fa0/1 → Root Port (toward SW3, lower bridge ID)
Fa0/2 → Alternate (discarding)
Fa0/24 → Designated (PC)
Verified:
Fa0/2 Altn BLK P2p
Step 3 – RSTP Link Type Configuration
RSTP link types:
Point-to-Point → Full-duplex switch-to-switch links
Shared → Hub connections (half-duplex)
Edge → PortFast-enabled access ports
SW4 Configuration
conf t
interface range f0/1 - 2
spanning-tree link-type point-to-point

interface f0/24
spanning-tree portfast
SW3 Configuration
conf t
interface f0/24
spanning-tree portfast
Note:
Fa0/2 connected to hub was automatically detected as shared.
SW2 Configuration
conf t
interface range f0/23 - 24
spanning-tree portfast
SW1 Configuration
conf t
interface f0/24
spanning-tree portfast
Important detail:
SW1 Fa0/24 is:
Connected to a hub
Therefore Shared
But also an Edge port (because it connects to end hosts)
In Packet Tracer, the Type column may only display Shr, but on a real Cisco switch it would show both:
Edge Shr
This demonstrates that:
Link type (P2p / Shr) reflects duplex
Edge reflects PortFast behavior
A port can be both Shared and Edge
Key RSTP Concepts Reinforced
RSTP uses protocol version 2
All switches generate BPDUs (unlike classic STP)
Port states reduced to:
Discarding
Learning
Forwarding
Non-designated role split into:
Alternate
Backup
PortFast, UplinkFast, and BackboneFast behavior built into RSTP
Verification Commands Used
show spanning-tree
spanning-tree mode rapid-pvst
spanning-tree link-type point-to-point
spanning-tree portfast
Final Outcome
Correct root bridge identified
All root, designated, alternate, and backup ports correctly mapped
RSTP link types manually configured where appropriate
Edge ports configured using PortFast
All configurations verified via CLI
This lab strengthened practical understanding of RSTP behavior beyond theory and highlighted how RSTP improves convergence compared to classic STP.
