# CCNA-LABS
 
## Lab 20 – IPv6 Address Types & Configuration (Part 2)
 
### Objective
 
Configure IPv6 addresses using EUI-64, enable IPv6 on interfaces without explicitly configuring a global address, and implement IPv6 static routes using link-local next-hop addresses. Verify end-to-end IPv6 connectivity between PC1 and PC2.
 
---
 
### Topology
 
- 2x Cisco 2911 Routers (R1, R2) — connected via G0/0 (link-local only)
- 2x Cisco 2960-24TT Switches (SW1, SW2)
- 2x PCs (PC1, PC2)
- Pre-configured IPv4 dual-stack network
---
 
### IP Addressing
 
#### Router Interfaces
 
| Device | Interface | IPv6 Address | Type |
|--------|-----------|-------------|------|
| R1 | G0/1 | 2001:DB8::230:F2FF:FE36:4502/64 | EUI-64 generated |
| R1 | G0/0 | FE80::230:F2FF:FE36:4501 | Link-local only (ipv6 enable) |
| R2 | G0/1 | 2001:DB8:0:1:201:63FF:FEB0:B802/64 | EUI-64 generated |
| R2 | G0/0 | FE80::201:63FF:FEB0:B801 | Link-local only (ipv6 enable) |
 
#### End Hosts
 
| PC | IPv6 Address | IPv6 Gateway |
|----|-------------|-------------|
| PC1 | 2001:DB8::2/64 | 2001:DB8::230:F2FF:FE36:4502 (R1 G0/1) |
| PC2 | 2001:DB8:0:1::2/64 | 2001:DB8:0:1:201:63FF:FEB0:B802 (R2 G0/1) |
 
---
 
### Configuration Summary
 
#### Step 1 — EUI-64 on R1 and R2 G0/1
 
```
! R1
ipv6 unicast-routing
interface GigabitEthernet0/1
 ipv6 address 2001:DB8::/64 eui-64
 
! R2
ipv6 unicast-routing
interface GigabitEthernet0/1
 ipv6 address 2001:DB8:0:1::/64 eui-64
```
 
#### Step 2 — PC Configuration
 
- PC1: IPv6 address `2001:DB8::2/64`, gateway = R1's EUI-64 address
- PC2: IPv6 address `2001:DB8:0:1::2/64`, gateway = R2's EUI-64 address
#### Step 3 — Enable IPv6 on G0/0 (Link-Local Only)
 
```
! R1
interface GigabitEthernet0/0
 ipv6 enable
 
! R2
interface GigabitEthernet0/0
 ipv6 enable
```
 
#### Step 4 — IPv6 Static Routes Using Link-Local Next-Hop
 
```
! R1 — route to PC2's subnet via R2's link-local
ipv6 route 2001:DB8:0:1::/64 GigabitEthernet0/0 FE80::201:63FF:FEB0:B801
 
! R2 — route to PC1's subnet via R1's link-local
ipv6 route 2001:DB8::/64 GigabitEthernet0/0 FE80::230:F2FF:FE36:4501
```
 
---
 
### Verification
 
| Check | Command | Expected Result |
|-------|---------|-----------------|
| EUI-64 addresses | `show ipv6 interface brief` on R1/R2 | G0/1 shows EUI-64 generated address + link-local |
| Link-local only | `show ipv6 interface brief` on R1/R2 | G0/0 shows FE80 address only — no global unicast |
| Static routes | `show ipv6 route` | S routes to each subnet present |
| End-to-end | `ping 2001:DB8:0:1::2` from PC1 | 4/4 replies |
 
---
 
### Troubleshooting Encountered
 
| Issue | Cause | Resolution |
|-------|-------|------------|
| Static route rejected when only link-local next-hop specified | When using a link-local address as next-hop, IOS requires the exit interface to be specified as well | Added exit interface before the next-hop: `ipv6 route [dest] g0/0 [FE80 address]` |
| `show ip interface brief` run instead of `show ipv6 interface brief` | Habit from IPv4 | Used correct `show ipv6 interface brief` command |
 
---
 
### Key Observations
 
- EUI-64 automatically generates the 64-bit interface ID from the interface MAC address — the router does the 3-step conversion (split, insert FFFE, invert 7th bit) automatically
- The link-local address and the EUI-64 global unicast address share the same interface ID — both derived from the same MAC address
- `ipv6 enable` configures only a link-local address — no global unicast is assigned
- When using a link-local address as a static route next-hop, you MUST specify the exit interface — IOS cannot determine which interface to use from a link-local address alone
- Link-local addresses are valid next-hop addresses for static routes even though they are not routed between subnets
- `ipv6 unicast-routing` must be enabled before IPv6 routing between subnets will work
---
 
### Skills Demonstrated
 
- EUI-64 address configuration (`ipv6 address [prefix]/64 eui-64`)
- Link-local only interface enablement (`ipv6 enable`)
- IPv6 static route configuration with link-local next-hop
- Exit interface requirement for link-local next-hop routes
- `show ipv6 interface brief` verification
- End-to-end IPv6 connectivity verification
---
 
### CCNA Exam Alignment
 
| Exam Topic | Description |
|------------|-------------|
| 1.8 | Configure and verify IPv6 addressing and prefixes |
| 1.9 | Describe IPv6 address types |
| 1.9.c | Link-local addresses |
| 1.9.f | Modified EUI-64 |
| 3.3 | Configure and verify IPv4 and IPv6 static routing |
 
---
 
### Files
 
- `Lab20-IPv6_Configuration_Part2.pkt`
