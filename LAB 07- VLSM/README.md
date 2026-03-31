# Lab 07 – VLSM (Variable Length Subnet Masking)
 
## Objective
Subnet a single Class C network using VLSM to provide efficient addressing for five subnets of different sizes, configure router interfaces, and establish full end-to-end connectivity using static routes.
 
## Topology
- 2x Cisco 2911 Routers (R1, R2)
- 4x Cisco 2960-24TT Switches (SW1, SW2, SW3, SW4)
- 4x PCs (PC1, PC2, PC3, PC4)
- Point-to-point link between R1 and R2
 
## Base Network
`192.168.5.0/24`
 
## VLSM Subnet Design
 
Subnets assigned largest-first to minimize address waste:
 
| Subnet | LAN | Hosts Required | Prefix | Network Address | Broadcast | First Usable | Last Usable |
|--------|-----|---------------|--------|----------------|-----------|--------------|-------------|
| 1 | LAN2 | 64 | /25 | 192.168.5.0 | 192.168.5.127 | 192.168.5.1 | 192.168.5.126 |
| 2 | LAN1 | 45 | /26 | 192.168.5.128 | 192.168.5.191 | 192.168.5.129 | 192.168.5.190 |
| 3 | LAN3 | 14 | /28 | 192.168.5.192 | 192.168.5.207 | 192.168.5.193 | 192.168.5.206 |
| 4 | LAN4 | 9 | /28 | 192.168.5.208 | 192.168.5.223 | 192.168.5.209 | 192.168.5.222 |
| 5 | P2P | 2 | /30 | 192.168.5.224 | 192.168.5.227 | 192.168.5.225 | 192.168.5.226 |
 
## IP Address Assignments
 
| Device | Interface | IP Address | Subnet Mask |
|--------|-----------|-----------|-------------|
| R1 | G0/0 | 192.168.5.190 | 255.255.255.192 |
| R1 | G0/1 | 192.168.5.126 | 255.255.255.128 |
| R1 | G0/0/0 | 192.168.5.225 | 255.255.255.252 |
| R2 | G0/0 | 192.168.5.206 | 255.255.255.240 |
| R2 | G0/1 | 192.168.5.222 | 255.255.255.240 |
| R2 | G0/0/0 | 192.168.5.226 | 255.255.255.252 |
| PC1 | Fa0 | 192.168.5.129 | 255.255.255.192 |
| PC2 | Fa0 | 192.168.5.1 | 255.255.255.128 |
| PC3 | Fa0 | 192.168.5.193 | 255.255.255.240 |
| PC4 | Fa0 | 192.168.5.209 | 255.255.255.240 |
 
## Static Routes Configured
 
**R1** (routes to R2's LANs via 192.168.5.226):
```
ip route 192.168.5.192 255.255.255.240 192.168.5.226
ip route 192.168.5.208 255.255.255.240 192.168.5.226
```
 
**R2** (routes to R1's LANs via 192.168.5.225):
```
ip route 192.168.5.128 255.255.255.192 192.168.5.225
ip route 192.168.5.0 255.255.255.128 192.168.5.225
```
 
## Validation & Testing
- Verified all router interfaces came up (up/up) after `no shutdown`
- Confirmed connected and local routes in `show ip route` on both routers
- Verified static routes appeared in routing table with `S` code
- Confirmed end-to-end connectivity with ICMP ping from PC1 to PC4 across both routers
- Initial 50% packet loss on first ping is expected due to ARP resolution — subsequent pings 100% success
 
## Key Concepts Demonstrated
- VLSM subnetting (largest subnet first)
- Block size method for calculating subnet boundaries
- Router interface IP assignment (last usable per subnet)
- PC IP assignment (first usable per subnet)
- Default gateway configuration on end hosts
- Static route configuration with next-hop IP
- Multi-router static routing (two hops between LANs)
- `show ip route` verification and routing table interpretation
 
## Lessons Learned
- Block size determines the next network address, not the broadcast — adding block size to the network address lands on the next network
- Default gateway must be set on end hosts — without it, traffic destined outside the local subnet never leaves the PC
- Router interfaces are administratively down by default — `no shutdown` is required (unlike switch ports)
- First ping failures across routers are normal — ARP needs to resolve before ICMP succeeds
 
## CCNA Exam Alignment
- **1.6** Configure and verify IPv4 addressing and subnetting
- **3.3** Configure and verify IPv4 static routing
- **3.2** Determine how a router makes a forwarding decision (longest prefix match)
- **3.1** Interpret the components of a routing table
 
## Files
- `Lab_07_-_VLSM.pkt`
 
