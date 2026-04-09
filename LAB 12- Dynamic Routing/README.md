# CCNA-LABS
 
## Lab 12 – Floating Static Routes
 
### Objective
 
Configure and verify floating static routes on R1 and R2 to provide a backup path to each Enterprise A LAN via ISP A, activated automatically when the direct link between R1 and R2 fails.
 
---
 
### Topology
 
- 2x Cisco 2911 Routers (R1, R2) — Enterprise A
- 2x Cisco 2911 ISP Routers (SPR1, SPR2) — ISP A
- 2x Cisco 2911 ISP Routers (ISPBR1, ISPBR2) — ISP B
- 2x Cisco 2960-24TT Switches (SW1, SW2)
- 1x PC (PC1) — 10.0.1.0/24
- 1x Server (SRV1) — 10.0.2.0/24
- Direct link between R1 and R2: 10.0.0.0/30
- ISP A backup path: R1 G0/0/0 → SPR1 → SPR2 → R2 G0/0/0
 
---
 
### Network Design
 
| Device | Interface | IP Address | Network |
|--------|-----------|------------|---------|
| R1 | G0/1 | 10.0.1.254/24 | Enterprise LAN 1 |
| R1 | G0/2/0 | 10.0.0.1/30 | Direct link to R2 |
| R1 | G0/0/0 | 203.0.113.2/30 | ISP A (SPR1) |
| R1 | G0/1/0 | 203.0.113.10/30 | ISP B (ISPBR1) |
| R2 | G0/1 | 10.0.2.254/24 | Enterprise LAN 2 |
| R2 | G0/2/0 | 10.0.0.2/30 | Direct link to R1 |
| R2 | G0/0/0 | 203.0.113.6/30 | ISP A (SPR2) |
| R2 | G0/1/0 | 203.0.113.14/30 | ISP B (ISPBR2) |
| SPR1 | G0/0/0 | 203.0.113.1/30 | ISP A next-hop for R1 |
| SPR2 | G0/0/0 | 203.0.113.5/30 | ISP A next-hop for R2 |
 
---
 
### Configuration Summary
 
#### Floating Static Routes
 
```
! R1 — backup route to 10.0.2.0/24 via ISP A (AD 115 > OSPF AD 110)
ip route 10.0.2.0 255.255.255.0 203.0.113.1 115
 
! R2 — backup route to 10.0.1.0/24 via ISP A (AD 115 > OSPF AD 110)
ip route 10.0.1.0 255.255.255.0 203.0.113.5 115
```
 
#### Routing Table — Normal Operation (OSPF active)
 
```
R1# show ip route
O    10.0.2.0/24 [110/2] via 10.0.0.2, GigabitEthernet0/2/0
! Floating static route does NOT appear — OSPF preferred
```
 
#### Routing Table — After Link Failure (G0/2/0 shutdown on R1)
 
```
R1# show ip route
S    10.0.2.0/24 [115/0] via 203.0.113.1
 
R2# show ip route
S    10.0.1.0/24 [115/0] via 203.0.113.5
```
 
---
 
### Verification
 
| Check | Command | Expected Result |
|-------|---------|-----------------|
| Routing table — normal | `show ip route` | OSPF route present, floating static not visible |
| Routing table — after failure | `show ip route` | Static route `[115/0]` appears on both R1 and R2 |
| Connectivity test | `ping 10.0.2.1` from PC1 | Successful via ISP A backup path |
| Path verification | `traceroute 10.0.2.1` from R1 | Path shows SPR1 → SPR2 → R2 |
 
---
 
### Troubleshooting Encountered
 
| Issue | Cause | Resolution |
|-------|-------|------------|
| Floating static routes not appearing after shutdown | Shutdown applied to R2's G0/2/0 — R1 still saw its own side as up | Shut down G0/2/0 on R1 instead |
| OSPF routes still showing briefly after shutdown | Packet Tracer slow to converge | Waited a few seconds and re-ran `show ip route` |
| First ping packet dropped after failover | ARP cache stale on devices in new path | Normal behavior — subsequent packets succeeded |
 
---
 
### Key Observations
 
- Floating static routes are configured identically to regular static routes — the only difference is the AD value appended at the end of the command
- AD set to 115 places it above OSPF's AD of 110 — OSPF route is always preferred when the direct link is up
- The floating static route is completely invisible in the routing table while the OSPF route exists — it only activates when the OSPF route is removed
- Both R1 and R2 need floating static routes — R1 for the forward path, R2 for the return path. Without R2's route, SRV1 reply traffic has no backup
- `traceroute` is a useful real-world verification tool — `traceroute` in Cisco IOS, `tracert` on Windows
 
---
 
### Skills Demonstrated
 
- Floating static route configuration with custom administrative distance
- AD-based route selection and understanding of when metric vs AD applies
- OSPF route verification and comparison with static backup routes
- Simulating link failure and verifying automatic failover behavior
- Path verification using traceroute
 
---
 
### CCNA Exam Alignment
 
| Exam Topic | Description |
|------------|-------------|
| 3.3 | Configure and verify IPv4 static routing |
| 3.3d | Floating static routes |
| 3.1e | Administrative distance |
| 3.2b | Administrative distance in forwarding decisions |
 
---
 
### Files
 
- `Lab12-Floating_Static_Routes.pkt`
