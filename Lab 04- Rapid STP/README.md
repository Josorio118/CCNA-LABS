LAB 04 – Rapid Spanning Tree Protocol (RSTP) Analysis
Objective
This lab focuses on analyzing Rapid Spanning Tree Protocol (RSTP) behavior in a multi-switch topology.
The goals were to:
Identify the root bridge
Determine port roles and states without using the CLI
Verify results using show spanning-tree
Configure appropriate RSTP link types
Configure edge ports using PortFast
Topology Overview
4x Cisco 2960 switches
1x hub (shared collision domain)
Multiple PCs connected to access ports
Single VLAN (VLAN 1)
All switches priority: 32769 (32768 + VLAN 1)
Because all switch priorities were equal, root election was determined by the lowest MAC address.
Step 1 – Root Bridge Identification
Command used:
show spanning-tree
Root Bridge: SW1
Root ID Priority 32769
Address 0005.5E4E.714B
This bridge is the root
Important Observation
On SW1:
Fa0/3  Back  BLK  Shr
This is a Backup port in RSTP.
In classic STP, this would have been a non-designated port.
Because SW1 has two interfaces connected to the same hub (same collision domain), only one interface can be designated. The other becomes a backup port.
Key refinement:
The root bridge has one designated port per collision domain.
Step 2 – Port Role and State Analysis
SW2
Fa0/1 → Root Port
Fa0/2 → Designated
Fa0/23–24 → Designated (PCs)
Gi0/1 → Alternate (Discarding)
Verified:
Gi0/1  Altn BLK  P2p
SW3
Fa0/2 → Root Port
Fa0/1 → Designated
Gi0/1 → Designated
Fa0/24 → Designated (PC)
Verified:
Fa0/2 Root FWD Shr
SW4
Fa0/1 → Root Port
Fa0/2 → Alternate
Fa0/24 → Designated (PC)
Verified:
Fa0/2 Altn BLK P2p
Step 3 – RSTP Link Type Configuration
RSTP Link Types
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
Fa0/2 was automatically detected as shared due to hub connection.
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
But also an Edge port (PortFast enabled)
In Packet Tracer, only Shr appears in the Type column.
On a real Cisco switch, it would display both:
Edge Shr
This demonstrates:
P2p / Shr = duplex-based link type
Edge = PortFast behavior
A port can be both Shared and Edge
Key RSTP Concepts Reinforced
RSTP uses protocol version 2
All switches generate BPDUs
Port states: Discarding, Learning, Forwarding
Non-designated role split into Alternate and Backup
PortFast, UplinkFast, and BackboneFast behavior built into RSTP
Verification Commands Used
show spanning-tree
spanning-tree mode rapid-pvst
spanning-tree link-type point-to-point
spanning-tree portfast
Final Outcome
Correct root bridge identified
Root, designated, alternate, and backup ports mapped accurately
RSTP link types configured where appropriate
Edge ports configured using PortFast
All configurations verified via CLI
This lab strengthened practical understanding of RSTP behavior and its improvements over classic STP.
