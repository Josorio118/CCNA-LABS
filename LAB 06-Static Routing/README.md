CCNA-LABS
Lab 06 – Static Routing
Objective
Build a multi-router topology from scratch and configure static routes to achieve end-to-end connectivity between two PCs across three routers. Includes a default route configuration and routing table verification.
Topology
PC1 --- R1 --- R2 --- R3 --- PC2

3x Cisco 2911 Routers
2x Generic PCs
Point-to-point /30 links between routers
/24 LAN segments on each end

Addressing
NetworkSubnetDevices192.168.1.0/24PC1 LANPC1, R1 G0/0192.168.12.0/30R1 ↔ R2R1 G0/1, R2 G0/0192.168.23.0/30R2 ↔ R3R2 G0/1, R3 G0/0192.168.3.0/24PC2 LANR3 G0/1, PC2
Interface IP Assignments
DeviceInterfaceIP AddressR1G0/0192.168.1.1/24R1G0/1192.168.12.1/30R2G0/0192.168.12.2/30R2G0/1192.168.23.1/30R3G0/0192.168.23.2/30R3G0/1192.168.3.1/24PC1Fa0192.168.1.10/24 — GW: 192.168.1.1PC2Fa0192.168.3.10/24 — GW: 192.168.3.1
Static Routes Configured
R1
ip route 192.168.23.0 255.255.255.252 192.168.12.2
ip route 192.168.3.0 255.255.255.0 192.168.12.2
ip route 0.0.0.0 0.0.0.0 192.168.12.2  ← default route
R2
ip route 192.168.1.0 255.255.255.0 192.168.12.1
ip route 192.168.3.0 255.255.255.0 192.168.23.2
R3
ip route 192.168.1.0 255.255.255.0 192.168.23.1
ip route 192.168.12.0 255.255.255.252 192.168.23.1
Key Configurations

IP addressing on all router interfaces via CLI
no shutdown on all router interfaces (administratively down by default)
Static routes using next-hop IP address format
Default route on R1 (0.0.0.0 0.0.0.0) pointing toward R2
PC default gateways mapped to directly connected router interfaces

Validation & Testing

Verified interface status with show ip interface brief
Confirmed static routes installed with show ip route (S and S* codes)
Verified Gateway of Last Resort set on R1
End-to-end ICMP ping from PC1 (192.168.1.10) to PC2 (192.168.3.10) — 4/4 success
Observed TTL=125 on replies (128 default − 3 router hops)

Routing Table Concepts Reinforced

C = Connected (directly attached network)
L = Local (router's own interface IP, /32)
S = Static (manually configured route)
S* = Candidate default route
Most specific route (longest prefix match) wins when multiple routes match
Gateway of last resort = where traffic goes when no specific route matches

Issues Encountered & Fixed

Router interfaces stayed down — forgot no shutdown. Router interfaces are administratively down by default, unlike switch interfaces. Must explicitly enable each one.
Duplicate IP address conflict — initially assigned 192.168.23.2 to R2 G0/1 instead of 192.168.23.1. Packet Tracer flagged %IP-4-DUPADDR. Fixed by correcting R2 G0/1 to 192.168.23.1.
Next-hop vs network confusion — initially mixing up interface IPs (e.g. 192.168.12.1) with destination networks (e.g. 192.168.12.0/30). Static route logic drilled: destination = the network you're missing, next hop = your neighbor's IP on the shared link.
Default route next hop — initially named R1's own interface (192.168.1.1) as the next hop. Corrected: next hop must always be a neighbor's IP, not your own. Default route next hop = 192.168.12.2 (R2).

Areas to Continue Drilling

Static route mental model: what networks am I missing? → which neighbor do I use? → what is that neighbor's IP on our shared link?
Distinguishing destination networks from interface IPs — destination always ends in .0/prefix
Default route placement and purpose — when to use it and what it replaces
CLI habits — use conf t, int g0/0, int range shortcuts; avoid GUI configuration

Skills Demonstrated

Multi-router topology build from scratch
Interface IP addressing and no shutdown via CLI
Static route configuration (next-hop format)
Default route configuration
Routing table interpretation (show ip route)
Real-time troubleshooting (duplicate IPs, down interfaces, wrong next-hops)
TTL analysis to count router hops

CCNA Exam Alignment
This lab directly supports CCNA 200-301 Objective 3.3:

Configure and verify IPv4 and IPv6 static routing — default route, network route, host route

Files

Lab06-Static-Routing.pkt
