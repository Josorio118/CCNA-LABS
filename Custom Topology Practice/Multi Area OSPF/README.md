# Multi-Area OSPF — Self-Built Hub-and-Spoke Topology

### Objective
Design and build a multi-area OSPF topology from scratch (not a pre-built lab file) to reinforce OSPF area design, router ID management, serial link configuration, and default route propagation ahead of exam sim practice. HQ acts as the ABR connecting three single-area branch routers, each serving its own LAN, with Internet access provided via a static default route redistributed into OSPF.

### Topology
```
                    ISP
                     |
              203.0.113.0/30
                     |
                HQ (ABR)  RID: 1.1.1.1
                  /   |   \
                 /    |    \
          10.10.12.0/30  10.10.13.0/30  10.10.14.0/30
             (Area 1)      (Area 2)      (Area 3)
                /             |              \
             BR1            BR2             BR3
          RID: 2.2.2.2   RID: 3.3.3.3    RID: 4.4.4.4
              |              |               |
        172.16.1.0/24  172.16.2.0/24   172.16.3.0/24
              |              |               |
            SW1            SW2             SW3
              |              |               |
            PC1            PC2             PC3
```

### IP Addressing
| Device | Interface | IP Address | Area |
|---|---|---|---|
| ISP | Se0/3/0 | 203.0.113.1/30 | — |
| HQ | Se0/3/0 | 203.0.113.2/30 | — |
| HQ | Loopback0 | 1.1.1.1/32 | 0 |
| HQ | Se0/3/1 (to BR1) | 10.10.12.1/30 | 1 |
| HQ | Se0/2/0 (to BR2) | 10.10.13.1/30 | 2 |
| HQ | Se0/2/1 (to BR3) | 10.10.14.1/30 | 3 |
| BR1 | Loopback0 | 2.2.2.2/32 | 1 |
| BR1 | Se0/3/0 | 10.10.12.2/30 | 1 |
| BR1 | Gi0/0 (LAN) | 172.16.1.1/24 | 1 |
| BR2 | Loopback0 | 3.3.3.3/32 | 2 |
| BR2 | Se0/3/0 | 10.10.13.2/30 | 2 |
| BR2 | Gi0/1 (LAN) | 172.16.2.1/24 | 2 |
| BR3 | Loopback0 | 4.4.4.4/32 | 3 |
| BR3 | Se0/3/0 | 10.10.14.2/30 | 3 |
| BR3 | Gi0/0 (LAN) | 172.16.3.1/24 | 3 |

### Configuration Summary
Each branch router runs single-area OSPF matching its topology position; HQ is the only multi-area ABR. Router IDs are manually set to match each router's Loopback0 address.

```
! HQ
router ospf 1
 router-id 1.1.1.1
 network 1.1.1.1 0.0.0.0 area 0
 network 10.10.12.0 0.0.0.3 area 1
 network 10.10.13.0 0.0.0.3 area 2
 network 10.10.14.0 0.0.0.3 area 3
 default-information originate
!
ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

```
! Branch router pattern (BR2 shown)
router ospf 1
 router-id 3.3.3.3
 network 3.3.3.3 0.0.0.0 area 2
 network 10.10.13.0 0.0.0.3 area 2
 network 172.16.2.0 0.0.0.255 area 2
```

DCE clock rates (`clock rate 64000`) configured on ISP (facing HQ) and on HQ (facing all three branch routers) — HQ is DTE toward ISP but DCE toward every branch, confirmed via the DCE clock icon in Packet Tracer's topology view before configuration.

### Verification
```
HQ#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           0   FULL/  -        00:00:37    10.10.12.2      Serial0/3/1
3.3.3.3           0   FULL/  -        00:00:39    10.10.13.2      Serial0/2/0
4.4.4.4           0   FULL/  -        00:00:37    10.10.14.2      Serial0/2/1
```

| Test | Result |
|---|---|
| PC1 → PC2 (Area 1 → Area 2, via HQ) | Success |
| PC1 → PC3 (Area 1 → Area 3, via HQ) | Success |
| HQ ↔ BR1/BR2/BR3 OSPF adjacencies | All FULL |
| HQ ↔ ISP link | Up, default route active |

### Troubleshooting Encountered
Two faults surfaced during initial build and were isolated through layered verification rather than re-reading the config line by line:

**1. BR2 OSPF neighbor table empty (no adjacency at all)**
`show ip interface brief` on BR2 confirmed the physical/L2 layer was fully healthy (Serial0/3/0 up/up, LAN interface up/up), which ruled out cabling or a missing `no shutdown`. Since OSPF wasn't attempting adjacency at all rather than getting stuck mid-negotiation, the next check was the `network` statements themselves. `show run | section ospf` revealed the serial link's wildcard mask was `0.0.0.0` instead of `0.0.0.3` — a `0.0.0.0` wildcard requires an exact address match, and no interface was configured with the literal network address `10.10.13.0`, so OSPF never activated on that interface. Comparing all three `network` statements side by side (the other two correctly used `0.0.0.3`) made the outlier obvious once checked directly instead of assumed correct.

**2. ISP-HQ serial link would not come up**
ISP's side showed a normal `no shutdown` with the link still reporting down. Reviewing the HQ configuration line by line showed `ip address 203.0.113.2 255.255.255.252` was applied to Se0/3/0, but the interface block moved directly to the next interface without ever issuing `no shutdown` — the only one of HQ's four serial interfaces missing it. The interface was left administratively down despite being fully addressed.

### Key Observations
- An empty OSPF neighbor table (vs. a neighbor stuck in Init/ExStart) is a useful diagnostic signal on its own: it usually points to OSPF not being enabled on the interface at all (wrong network statement/wildcard/area) rather than a negotiation-level problem like an MTU or hello/dead timer mismatch.
- A `0.0.0.0` wildcard mask is a silent failure — Cisco IOS accepts the command without error, but it only matches an exact host address, so it's easy to mistake for a working (if imprecise) statement rather than one that matches nothing at all.
- Configuring several interfaces back-to-back in one session increases the risk of skipping a step (like `no shutdown`) on exactly one of them — worth a full `show ip interface brief` sweep after a multi-interface configuration block, rather than trusting that a repeated pattern was applied consistently.
- Physical-layer status (`show ip interface brief`) should be checked before OSPF-layer troubleshooting begins on any suspect link — it immediately separates cabling/interface issues from routing protocol configuration issues and prevents wasted time investigating the wrong layer.

### Skills Demonstrated
- Multi-area OSPF design and ABR configuration from a blank topology
- Manual router ID assignment and Loopback0 advertisement
- DCE/DTE identification and clock rate configuration on serial WAN links
- Default route origination into OSPF (`default-information originate`)
- Layered fault isolation (physical -> OSPF neighbor state -> configuration review) without relying on pre-annotated lab objectives

### CCNA Exam Alignment
- IP Connectivity: OSPFv2 multi-area configuration, router ID, network statement/wildcard mask precision, default route propagation
- Network Fundamentals: serial WAN interface configuration, DCE/DTE clocking
- Troubleshooting methodology: systematic layer-by-layer verification

### Files
- `MultiArea_OSPF.pkt`
