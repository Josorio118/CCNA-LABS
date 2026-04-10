# CCNA-LABS

## Lab 13 – EIGRP Configuration & Unequal-Cost Load-Balancing

### Objective

Configure EIGRP AS100 across a four-router topology. Enable EIGRP on all appropriate interfaces using wildcard masks, configure loopback interfaces for router IDs, set passive interfaces, disable auto-summary, and configure unequal-cost load-balancing on R1 using the variance command.

---

### Topology

- 4x Routers (R1, R2, R3, R4)
- 1x Cisco 2960-24TT Switch (SW1)
- 1x PC (PC1) — 192.168.4.0/24
- R1 ↔ R2: GigabitEthernet (10.0.12.0/30)
- R1 ↔ R3: FastEthernet (10.0.13.0/30)
- R2 ↔ R4: FastEthernet (10.0.24.0/30)
- R3 ↔ R4: FastEthernet (10.0.34.0/30)
- R4 ↔ SW1 ↔ PC1: GigabitEthernet (192.168.4.0/24)
- EIGRP AS: 100

---

### Network Design

| Device | Interface | IP Address | Network |
|--------|-----------|------------|---------|
| R1 | G0/0 | 10.0.12.1/30 | Link to R2 |
| R1 | F1/0 | 10.0.13.1/30 | Link to R3 |
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

---

### Configuration Summary

#### Loopback Interfaces (all routers)

```
interface loopback 0
 ip address X.X.X.X 255.255.255.255
```

#### EIGRP Configuration — R1

```
router eigrp 100
 network 10.0.12.0 0.0.0.3
 network 10.0.13.0 0.0.0.3
 network 1.1.1.1 0.0.0.0
 no auto-summary
 passive-interface loopback0
```

#### EIGRP Configuration — R2

```
router eigrp 100
 network 10.0.12.0 0.0.0.3
 network 10.0.24.0 0.0.0.3
 network 2.2.2.2 0.0.0.0
 no auto-summary
 passive-interface loopback0
```

#### EIGRP Configuration — R3

```
router eigrp 100
 network 10.0.13.0 0.0.0.3
 network 10.0.34.0 0.0.0.3
 network 3.3.3.3 0.0.0.0
 no auto-summary
 passive-interface loopback0
```

#### EIGRP Configuration — R4 (shortcut network command)

```
router eigrp 100
 network 0.0.0.0 255.255.255.255
 no auto-summary
 passive-interface gigabitEthernet0/0
 passive-interface loopback0
```

#### Unequal-Cost Load-Balancing — R1

```
router eigrp 100
 variance 2
```

---

### Verification

| Check | Command | Expected Result |
|-------|---------|-----------------|
| EIGRP neighbors | `show ip eigrp neighbors` | R2 via G0/0, R3 via F1/0 |
| EIGRP routes | `show ip route eigrp` | Routes to all loopbacks, inter-router links, 192.168.4.0/24 |
| Topology table | `show ip eigrp topology` | Two paths to 192.168.4.0/24 — successor via R2, feasible successor via R3 |
| Load-balancing | `show ip route eigrp` after variance 2 | Both paths to 192.168.4.0/24 installed |
| Protocol status | `show ip protocols` | AS 100, auto-summary off, passive interfaces listed |

---

### Troubleshooting Encountered

| Issue | Cause | Resolution |
|-------|-------|------------|
| 192.168.4.0/24 missing from topology table | R4's G0/0 unassigned and administratively down | Configured 192.168.4.254/24 and `no shutdown` on R4 G0/0 |
| Loopback interfaces on wrong numbers (L1, L2, L3) | All routers should use `loopback 0` | Used `no interface loopback X` to delete, then configured `interface loopback 0` |
| Incomplete command error on `ip add` | Subnet mask omitted | Re-entered full command with subnet mask |

---

### Key Observations

- EIGRP AS number must match on all routers — mismatched AS = no adjacency, no route sharing
- `network 0.0.0.0 255.255.255.255` activates EIGRP on all interfaces at once — convenient for labs, not recommended in production
- Loopback interfaces are always up/up unless manually shut down — ideal for stable router IDs
- Auto-summary must be explicitly disabled — when enabled it advertises classful networks which breaks routing for specific subnets
- EIGRP routes show as D in the routing table
- Feasibility condition: a route is a feasible successor only if its reported distance is less than the successor's feasible distance — loop-prevention mechanism
- Unequal-cost load-balancing only works over feasible successor routes — variance alone is not sufficient if feasibility condition is not met
- With `variance 2`, R1 sends more traffic via R2 (lower metric) and less via R3 (higher metric) — proportional to path speed

---

### EIGRP Terminology Reference

| Term | Definition |
|------|-----------|
| Feasible Distance (FD) | This router's total metric to the destination |
| Reported Distance (RD) | The neighbor's metric to the destination (also called Advertised Distance) |
| Successor | Route with the lowest FD — best route, installed in routing table |
| Feasible Successor | Backup route where RD < successor's FD — guaranteed loop-free |

---

### Skills Demonstrated

- EIGRP AS configuration and neighbor adjacency verification
- Wildcard mask usage in EIGRP network commands
- Loopback interface configuration and use as router ID
- Passive interface configuration on loopback and LAN interfaces
- EIGRP topology table analysis (FD, RD, successor, feasible successor)
- Unequal-cost load-balancing using the variance command

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 3.0 | IP Connectivity — dynamic routing overview |
| 3.2 | How a router makes forwarding decisions (metric, AD) |

---

### Files

- `Lab13-EIGRP_Configuration.pkt`
