# CCNA-LABS

## Lab 15 – OSPFv2 Single Area (Part 2)

### Objective

Configure OSPFv2 using the `ip ospf` interface command instead of the `network` command. Configure passive interfaces using both standard and default methods. Adjust the OSPF reference bandwidth so FastEthernet interfaces have a cost of 100. Configure R1 as an ASBR advertising a default route into the OSPF domain. Verify routing tables and observe OSPF Hello messages in simulation mode.

---

### Topology

- 4x Routers (R1, R2, R3, R4)
- 1x Cisco 2960-24TT Switch (SW1)
- 1x PC (PC1) — 192.168.4.0/24
- 1x ISP Router (ISPR1) — 203.0.113.0/30
- R1 ↔ R2: GigabitEthernet (10.0.12.0/30)
- R1 ↔ R3: FastEthernet (10.0.13.0/30)
- R2 ↔ R4: FastEthernet (10.0.24.0/30)
- R3 ↔ R4: FastEthernet (10.0.34.0/30)
- R1 ↔ ISPR1: GigabitEthernet (203.0.113.0/30)
- R4 ↔ SW1 ↔ PC1: GigabitEthernet (192.168.4.0/24)
- OSPF Area: 0 (single area)

---

### Network Design

| Device | Interface | IP Address | Network |
|--------|-----------|------------|---------|
| R1 | G0/0 | 10.0.12.1/30 | Link to R2 |
| R1 | F1/0 | 10.0.13.1/30 | Link to R3 |
| R1 | G3/0 | 203.0.113.1/30 | Link to ISPR1 (no OSPF) |
| R1 | Loopback0 | 1.1.1.1/32 | Router ID |
| R2 | G0/0 | 10.0.12.2/30 | Link to R1 |
| R2 | F1/0 | 10.0.24.1/30 | Link to R4 |
| R2 | Loopback0 | 2.2.2.2/32 | Router ID |
| R3 | F1/0 | 10.0.13.2/30 | Link to R1 |
| R3 | F2/0 | 10.0.34.1/30 | Link to R4 |
| R3 | Loopback0 | 3.3.3.3/32 | Router ID |
| R4 | F1/0 | 10.0.24.2/30 | Link to R2 |
| R4 | F2/0 | 10.0.34.2/30 | Link to R3 |
| R4 | G0/0 | 192.168.4.254/24 | LAN |
| R4 | Loopback0 | 4.4.4.4/32 | Router ID |
| ISPR1 | G0/0 | 203.0.113.2/30 | Link to R1 |

---

### Configuration Summary

#### OSPF — Enable Directly on Interfaces (R1)

```
interface g0/0
 ip ospf 1 area 0
interface f1/0
 ip ospf 1 area 0
interface loopback 0
 ip ospf 1 area 0
 ip ospf cost 1

router ospf 1
 passive-interface loopback0
```

#### OSPF — Enable Directly on Interfaces (R2)

```
interface g0/0
 ip ospf 1 area 0
interface f1/0
 ip ospf 1 area 0

router ospf 1
 passive-interface default
 no passive-interface g0/0
 no passive-interface f1/0
```

#### OSPF — Enable Directly on Interfaces (R3)

```
interface f1/0
 ip ospf 1 area 0
interface f2/0
 ip ospf 1 area 0

router ospf 1
 passive-interface default
 no passive-interface f1/0
 no passive-interface f2/0
```

#### OSPF — Enable Directly on Interfaces (R4)

```
interface f1/0
 ip ospf 1 area 0
interface f2/0
 ip ospf 1 area 0

router ospf 1
 passive-interface default
 no passive-interface f1/0
 no passive-interface f2/0
```

#### Reference Bandwidth (all routers)

```
router ospf 1
 auto-cost reference-bandwidth 10000
```

#### Default Route and ASBR (R1)

```
ip route 0.0.0.0 0.0.0.0 203.0.113.2

router ospf 1
 default-information originate
```

---

### Verification

| Check | Command | Expected Result |
|-------|---------|-----------------|
| OSPF interfaces and cost | `show ip ospf interface brief` | GigabitEthernet = 100, FastEthernet = 1000 |
| OSPF neighbors | `show ip ospf neighbor` | All expected adjacencies in Full state |
| Routing table R4 | `show ip route` | `O*E2 0.0.0.0/0` via R2 present |
| Loopback cost | `show ip ospf interface loopback 0` | Cost = 1 after manual override |
| Hello message fields | Simulation mode | Destination 224.0.0.5, Type 1, Protocol 0x59 (89 decimal) |

---

### Troubleshooting Encountered

| Issue | Cause | Resolution |
|-------|-------|------------|
| `ip add 203.0.113.0` rejected | 203.0.113.0 is the network address, not a host address | Used 203.0.113.1 on R1 G3/0 |
| Loopback0 showing cost 12 instead of 1 | Packet Tracer quirk — incorrect auto-calculated cost on loopback | Applied `ip ospf cost 1` directly on loopback interface |
| `passive-interface` rejected from global config mode | Must be entered from OSPF config mode | Entered `router ospf 1` first |
| R4 routing table shows only one default route path | Packet Tracer bug — E2 external metric should make both paths equal cost | Expected behavior on real equipment — both R2 and R3 paths would appear |
| R1 loopback initially configured with /30 mask | Typed 255.255.255.252 instead of 255.255.255.255 | Re-entered correct /32 mask |

---

### Key Observations

- `ip ospf [process-id] area [area]` entered directly on an interface achieves the same result as the `network` command — it's more precise and requires no wildcard mask math
- `passive-interface default` followed by `no passive-interface` on active interfaces is efficient when most interfaces need to be passive
- Reference bandwidth of 10,000 Mbps gives FastEthernet a cost of 100 and GigabitEthernet a cost of 10 — properly differentiating link speeds
- Reference bandwidth must be consistent across all OSPF routers in the network
- The default route is advertised as `O*E2` (OSPF External Type 2) — the E2 metric means the internal OSPF path cost to R1 is not added, so the cost stays at 1 regardless of path
- OSPF Hello messages are sent to multicast 224.0.0.5, identified as Type 1, and encapsulated in IP with protocol number 89 (0x59 hex)
- OSPF is not enabled on R1's G3/0 (Internet link) — other routers don't need to know the ISP subnet, only that R1 is the gateway to the Internet

---

### Skills Demonstrated

- Alternative OSPF interface activation using `ip ospf` command
- `passive-interface default` method for bulk passive configuration
- OSPF reference bandwidth adjustment and cost verification
- Manual OSPF cost override with `ip ospf cost`
- ASBR configuration and default route advertisement
- OSPF Hello message analysis in simulation mode
- OSPF routing table interpretation including O*E2 external routes

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 3.4 | Configure and verify single area OSPFv2 |
| 3.4a | Neighbor adjacencies |
| 3.4b | Point-to-point |
| 3.4d | Router ID |
| 3.1 | Interpret components of routing table |
| 3.1f | Metric |

---

### Files

- `Lab15-OSPF_Part2.pkt`
