# CCNA-LABS

## Lab 21 – IPv6 Static Routes (Part 3)

### Objective

Configure IPv6 static routes to enable PC1 and PC2 to reach each other via a primary path through R3 and a floating static backup path through R2. Use SLAAC to automatically configure IPv6 addresses on the PCs. Verify failover by removing the primary link and confirming the backup route takes over.

---

### Topology

- 3x Cisco 2911 Routers (R1, R2, R3)
- 2x Cisco 2960-24TT Switches (SW1, SW3)
- 2x PCs (PC1, PC2)
- R1 ↔ R3: GigabitEthernet (primary path)
- R1 ↔ R2 ↔ R3: Serial (backup path)
- IPv6 addresses pre-configured on routers; PCs configured via SLAAC

---

### IP Addressing

| Device | Interface | IPv6 Address |
|--------|-----------|-------------|
| R1 | G0/1 (LAN) | 2001:DB8:0:1::/64 gateway |
| R1 | G0/1 (to R3) | 2001:DB8:0:13::1/64 |
| R3 | G0/1 (to R1) | 2001:DB8:0:13::2/64 |
| R3 | G0/0 (LAN) | 2001:DB8:0:3::/64 gateway |
| R1/R2 | S0/0/0 | Link-local only (ipv6 enable) |
| R2/R3 | S0/0/x | Link-local only (ipv6 enable) |
| PC1 | FastEthernet0 | SLAAC — auto-generated from R1's prefix |
| PC2 | FastEthernet0 | SLAAC — auto-generated from R3's prefix |

---

### Configuration Summary

#### Step 1 — Enable IPv6 Routing on All Routers

```
! R1, R2, R3
ipv6 unicast-routing
```

> Required before SLAAC works — routers must send NDP Router Advertisements for PCs to auto-configure addresses.

#### Step 2 — SLAAC on PC1 and PC2

- Set Gateway/DNS IPv6 to **Automatic** in PC Config tab
- Set FastEthernet0 IPv6 to **Automatic**
- PC learns network prefix from router's RA message and generates interface ID using EUI-64

#### Step 3 — IPv6 Static Routes

**R1 — Primary route to R3's LAN (fully specified via G0/1):**
```
ipv6 route 2001:db8:0:3::/64 GigabitEthernet0/1 2001:db8:0:13::2
```

**R1 — Floating static backup via R2's serial link-local (AD 5):**
```
ipv6 route 2001:db8:0:3::/64 Serial0/0/0 [R2's link-local FE80 address] 5
```

**R2 — Route to R1's LAN:**
```
ipv6 route 2001:db8:0:1::/64 Serial0/0/0 [R1's S0/0/0 link-local]
```

**R2 — Route to R3's LAN:**
```
ipv6 route 2001:db8:0:3::/64 Serial0/0/1 [R3's S0/0/0 link-local]
```

**R3 — Primary route to R1's LAN (fully specified via G0/1):**
```
ipv6 route 2001:db8:0:1::/64 GigabitEthernet0/1 2001:db8:0:13::1
```

**R3 — Floating static backup via R2's serial link-local (AD 5):**
```
ipv6 route 2001:db8:0:1::/64 Serial0/0/0 [R2's S0/0/1 link-local] 5
```

---

### Verification

| Check | Command | Expected Result |
|-------|---------|-----------------|
| IPv6 routing table | `show ipv6 route` | Static routes present; floating static not shown (only in running-config) |
| Floating static confirmation | `show run | include ipv6 route` | Both primary and floating routes visible |
| Link-local addresses | `show ipv6 interface brief` | FE80 addresses on serial interfaces |
| PC SLAAC address | `ipconfig` on PC | IPv6 address with prefix learned from router |
| Primary path ping | `ping [PC2 IPv6]` from PC1 | 4/4 success |
| Primary path traceroute | `tracert [PC2 IPv6]` from PC1 | Hops: R1 → R3 → PC2 |
| Failover test | Delete R1-R3 cable, ping PC2 | Still 4/4 — backup route via R2 activates |
| Failover traceroute | `tracert [PC2 IPv6]` after cable removed | Path goes through R2; R2 may show * (link-local not routable) |

---

### Troubleshooting Encountered

| Issue | Cause | Resolution |
|-------|-------|------------|
| SLAAC not working on PCs | ipv6 unicast-routing not yet enabled on routers — no RA messages being sent | Enabled ipv6 unicast-routing on all three routers first |
| Floating static not visible in show ipv6 route | Normal behavior — floating static only appears when primary route is down | Confirmed with show run \| include ipv6 route |
| Traceroute shows * at R2 hop | R2 only has link-local addresses on serial interfaces — link-local addresses are not routable | Expected behavior — ping still works end-to-end |
| Link-local next-hop rejected without exit interface | When using FE80 address as next-hop, IOS requires exit interface to be specified | Used fully specified route: ipv6 route [dest] [interface] [FE80 address] |

---

### Key Observations

- `ipv6 unicast-routing` is required not just for routing — without it routers do not send RA messages, so SLAAC cannot work on connected PCs
- SLAAC uses NDP Router Advertisement messages to learn the network prefix, then generates the interface ID using EUI-64
- Floating static routes do not appear in `show ipv6 route` while the primary route is active — verify with `show run | include ipv6 route`
- Link-local addresses work as next-hop addresses for static routes but are not routable between subnets — traceroute shows * at those hops, which is normal
- Directly attached static routes do not work on Ethernet interfaces in IPv6 — fully specified routes were used throughout
- After removing the primary link, the backup floating static route activates automatically — same failover behavior as IPv4 floating static

---

### Skills Demonstrated

- SLAAC configuration via NDP RA/RS exchange
- IPv6 static route configuration (fully specified and recursive)
- IPv6 floating static route configuration (AD manipulation)
- Link-local address as next-hop in fully specified static routes
- `show ipv6 route` and `show run | include ipv6 route` verification
- IPv6 failover testing with link removal and traceroute analysis

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 1.8 | Configure and verify IPv6 addressing and prefixes |
| 3.3 | Configure and verify IPv4 and IPv6 static routing |
| 3.3.a | Default route |
| 3.3.b | Network route |
| 3.3.d | Floating static |

---

### Files

- `Lab21-IPv6_Static_Routes.pkt`
