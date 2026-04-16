# CCNA-LABS

## Lab 14 – OSPFv2 Single Area (Part 1)

### Objective

Configure single-area OSPFv2 across a four-router topology. Enable OSPF on all appropriate interfaces, configure loopback interfaces as stable router IDs, set passive interfaces where appropriate, and configure R1 as an ASBR that advertises a default route into the OSPF domain.

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
| R1 | G3/0 | 203.0.113.1/30 | Link to ISPR1 |
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

#### Loopback Interfaces (all routers)

```
interface loopback 0
 ip address X.X.X.X 255.255.255.255
```

#### OSPF Configuration — R1

```
router ospf 1
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.13.0 0.0.0.3 area 0
 network 1.1.1.1 0.0.0.0 area 0
 passive-interface loopback0
 default-information originate
```

#### OSPF Configuration — R2

```
router ospf 1
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.24.0 0.0.0.3 area 0
 network 2.2.2.2 0.0.0.0 area 0
 passive-interface loopback0
```

#### OSPF Configuration — R3

```
router ospf 1
 network 10.0.13.0 0.0.0.3 area 0
 network 10.0.34.0 0.0.0.3 area 0
 network 3.3.3.3 0.0.0.0 area 0
 passive-interface loopback0
```

#### OSPF Configuration — R4

```
router ospf 1
 network 10.0.24.0 0.0.0.3 area 0
 network 10.0.34.0 0.0.0.3 area 0
 network 4.4.4.4 0.0.0.0 area 0
 passive-interface loopback0
 passive-interface gigabitEthernet0/0
```

#### Default Route and ASBR — R1

```
ip route 0.0.0.0 0.0.0.0 203.0.113.2

router ospf 1
 default-information originate
```

---

### Verification

| Check | Command | Expected Result |
|-------|---------|-----------------|
| OSPF neighbors | `show ip ospf neighbor` | R2 via G0/0, R3 via F1/0 on R1 |
| Routing table | `show ip route` | OSPF routes to all loopbacks and subnets |
| Default route on R2 | `show ip route` | `O*E2 0.0.0.0/0 [110/1] via 10.0.12.1` |
| Default route on R3 | `show ip route` | `O*E2 0.0.0.0/0 [110/1] via 10.0.13.1` |
| Default route on R4 | `show ip route` | `O*E2 0.0.0.0/0` via both R2 and R3 (ECMP) |
| LSDB | `show ip ospf database` | Router, Net, and Type-5 AS External LSAs present |
| Protocol status | `show ip protocols` | Process ID, router ID 1.1.1.1, ASBR noted on R1 |

---

### Troubleshooting Encountered

| Issue | Cause | Resolution |
|-------|-------|------------|
| `network` command returned incomplete command error | Area number missing | Added `area 0` at end of each network statement |
| `passive-interface` rejected from global config mode | Must be entered from OSPF config mode | Re-entered `router ospf 1` then ran command |
| R1 G3/0 had duplicate IP (203.0.113.2) | Copied ISPR1's address by mistake | Corrected to 203.0.113.1 on R1 G3/0 |
| Wrong wildcard mask on R4 loopback network statement | Used 0.0.0.3 instead of 0.0.0.0 for /32 | Wildcard for /32 = 0.0.0.0 — all bits must match |

---

### Key Observations

- OSPF process ID is locally significant — routers with different process IDs can still become OSPF neighbors. Different from EIGRP where AS number must match.
- The NETWORK command in OSPF requires the area number at the end — omitting it returns an incomplete command error
- Loopback interfaces and LAN-facing interfaces (like R4 G0/0) should be passive — no need to send hello messages where there are no OSPF neighbors
- OSPF is not enabled on R1's G3/0 (Internet link) — other routers only need to know to send traffic to R1, not the ISP subnet details
- `default-information originate` makes R1 an ASBR and floods the default route as `O*E2` to all OSPF neighbors
- R4 received the default route via both R2 and R3 with equal cost — OSPF ECMP load-balancing across equal-metric FastEthernet paths

---

### Skills Demonstrated

- Single-area OSPFv2 configuration and verification
- OSPF process ID behavior (locally significant)
- Wildcard mask usage in OSPF network commands
- Loopback interface configuration as stable router ID
- Passive interface configuration in OSPF
- ASBR configuration using `default-information originate`
- OSPF routing table analysis including external routes (O*E2)

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 3.4 | Configure and verify single area OSPFv2 |
| 3.4a | Neighbor adjacencies |
| 3.4d | Router ID |
| 3.1 | Interpret components of routing table |
| 3.1e | Administrative distance |

---

### Files

- `Lab14-OSPF_Part1.pkt`
