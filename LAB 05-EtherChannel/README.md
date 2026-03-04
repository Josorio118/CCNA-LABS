Lab 05 – EtherChannel (Layer 2 & Layer 3)
Overview
This lab configures Layer 2 and Layer 3 EtherChannels across a multi-switch topology using LACP, PAgP, and Static EtherChannel. The goal was to replace STP-blocked redundant links with active, load-balanced port-channels — providing both redundancy and increased bandwidth.

Topology

2x Access Layer Switches (ASW1, ASW2) — Cisco 2960
2x Distribution Layer Switches (DSW1, DSW2) — Cisco 3650
2x End Hosts (PC1, PC2)
1x Server (SRV1)


EtherChannel Design
Port-ChannelSwitchesProtocolTypePurposePo1ASW1 ↔ DSW1LACP (active/active)Layer 2Access to distribution uplinkPo1ASW2 ↔ DSW2PAgP (desirable/desirable)Layer 2Access to distribution uplinkPo2DSW1 ↔ DSW2Static (on/on)Layer 3Distribution interconnect

IP Addressing
InterfaceDeviceIP AddressPort-channel2DSW110.0.0.1/30Port-channel2DSW210.0.0.2/30VLAN1 SVIDSW1172.16.1.254/24VLAN1 SVIDSW2172.16.2.254/24

Configuration Summary
Step 1 — Layer 2 EtherChannel: ASW1 ↔ DSW1 (LACP)
ASW1:
interface range g0/1 - 2
 channel-group 1 mode active

interface port-channel 1
 switchport mode trunk
DSW1:
interface range g1/0/3 - 4
 channel-group 1 mode active

interface port-channel 1
 switchport trunk encapsulation dot1q
 switchport mode trunk

Step 2 — Layer 2 EtherChannel: ASW2 ↔ DSW2 (PAgP)
ASW2:
interface range g0/1 - 2
 channel-group 1 mode desirable

interface port-channel 1
 switchport mode trunk
DSW2:
interface range g1/0/3 - 4
 channel-group 1 mode desirable

interface port-channel 1
 switchport trunk encapsulation dot1q
 switchport mode trunk

Step 3 — Layer 3 EtherChannel: DSW1 ↔ DSW2 (Static)
Critical: no switchport must be applied to BOTH the port-channel interface AND the physical member interfaces before assigning the channel-group. Failing to do this causes the L2/L3 mismatch error.
DSW1:
interface port-channel 2
 no switchport

interface range g1/0/1 - 2
 no switchport
 channel-group 2 mode on

interface port-channel 2
 ip address 10.0.0.1 255.255.255.252
DSW2:
interface range g1/0/1 - 2
 no switchport
 channel-group 2 mode on

interface port-channel 2
 ip address 10.0.0.2 255.255.255.252

Step 4 — IP Routing and Static Routes
! On both DSW1 and DSW2
ip routing

! DSW1 — route to SRV1's subnet
ip route 172.16.2.0 255.255.255.0 10.0.0.2

! DSW2 — route to PC subnet
ip route 172.16.1.0 255.255.255.0 10.0.0.1

Note: ip routing must be enabled on multilayer switches before the routing table is built. Without it, static routes have no effect even if configured.


Step 5 & 6 — Load Balancing
Default load-balancing method on both switch models: src-mac
Changed to src-dst-ip on all switches:
port-channel load-balance src-dst-ip
Verified with:
show etherchannel load-balance

Verification
Expected show etherchannel summary output (DSW1)
1    Po1(SU)    LACP    Gig1/0/3(P) Gig1/0/4(P)
2    Po2(RU)    -       Gig1/0/1(P) Gig1/0/2(P)

SU = Layer 2, in use ✅
RU = Layer 3, in use ✅
P = properly bundled ✅

Verification Commands Used
show etherchannel summary
show etherchannel load-balance
show interfaces trunk
show spanning-tree
show ip route
ping 10.0.0.2
ping 172.16.2.1      (PC1 to SRV1)
STP Verification (ASW1)
Before EtherChannel: G0/1 = Root Port, G0/2 = Alternate (blocking)
After EtherChannel: Po1 = Root Port — both physical links active, no blocking

Troubleshooting Encountered
Error: Command rejected (Port-channel): Either port is L2 and port-channel is L3, or vice-versa
Cause: The port-channel interface was created as Layer 2 by default before no switchport was applied to the physical interfaces. When attempting to add routed (L3) interfaces to a Layer 2 port-channel, the command is rejected.
Fix: Run no switchport on the port-channel interface itself first, then on the physical member interfaces, then assign the channel-group.
Key rule learned: For a Layer 3 EtherChannel — no switchport must be applied to both the port-channel interface AND the physical member interfaces before adding an IP address.

Key Takeaways

EtherChannel bundles physical links into one logical interface — STP sees one link, all physical links forward traffic
LACP (active/passive) is the preferred method — IEEE standard, works with any vendor
PAgP (desirable/auto) is Cisco proprietary — only between Cisco switches
Static (on/on) requires both sides to be on — mixing with LACP or PAgP modes fails
Member interfaces must match: speed, duplex, switchport mode, allowed VLANs
Layer 3 EtherChannel requires no switchport before the channel-group command
ip routing must be explicitly enabled on multilayer switches
Load balancing is per-flow, not per-frame — guarantees in-order delivery


CCNA Exam Alignment
This lab covers CCNA 200-301 Exam Topic 2.4:

Configure and verify Layer 2 and Layer 3 EtherChannel (LACP)

Skills demonstrated:

LACP, PAgP, and Static EtherChannel configuration
Layer 2 and Layer 3 EtherChannel distinction
EtherChannel verification using show commands
STP behavior with EtherChannel
Static routing on multilayer switches
Real troubleshooting of a common EtherChannel misconfiguration
