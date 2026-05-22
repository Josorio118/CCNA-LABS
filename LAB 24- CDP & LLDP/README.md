# CCNA-LABS

## Lab 24 – CDP & LLDP

### Objective

Use CDP to discover and document unknown IP addresses and interface IDs in a network topology, then disable CDP on PC-facing switch interfaces, disable CDP globally on all devices, and replace it with LLDP on all network device interfaces.

---

### Topology

- 3x Cisco 2911 Routers (R1, R2, R3)
- 3x Cisco 2960-24TT Switches (SW1, SW2, SW3)
- 3x PCs (PC1, PC2, PC3)
- R1 connects to SW1 (G0/2), R2 (G0/1), and R3 (G0/0)
- R2 connects to SW2 (G0/1) and R3 (G0/2)
- R3 connects to SW3 (G0/0)

---

### Discovered IP Addresses (Step 1 — via CDP)

| Device | Interface | IP Address |
|--------|-----------|-----------|
| R1 | G0/0 | 10.0.13.1/30 |
| R1 | G0/1 | 10.0.12.1/30 |
| R1 | G0/2 | 192.168.1.254/24 |
| R2 | G0/0 | 10.0.12.2 |
| R2 | G0/2 | 10.0.23.1 |
| R3 | G0/1 | 10.0.13.2 |
| R3 | G0/2 | 10.0.23.2 |

### Discovered Interface Connections (Step 1 — via CDP)

| From | Local Int | To | Remote Int |
|------|-----------|----|------------|
| R1 | G0/2 | SW1 | G0/1 |
| R1 | G0/1 | R2 | G0/0 |
| R1 | G0/0 | R3 | G0/1 |
| R2 | G0/1 | SW2 | G0/2 |
| R2 | G0/2 | R3 | G0/2 |
| R3 | G0/0 | SW3 | G0/1 |

---

### Configuration Summary

#### Step 1 — CDP Discovery (no config needed)

```
! On R1, R2, R3 — CDP already enabled by default
do show cdp neighbors detail
```

Used physical view on each switch to identify PC-facing interfaces:
- SW1 → FastEthernet0/10 (PC1)
- SW2 → FastEthernet0/1 (PC2)
- SW3 → FastEthernet0/24 (PC3)

#### Step 2 — Disable CDP on PC-facing switch interfaces

```
! SW1
interface FastEthernet0/10
 no cdp enable

! SW2
interface FastEthernet0/1
 no cdp enable

! SW3
interface FastEthernet0/24
 no cdp enable
```

#### Step 3 — Disable CDP globally on all devices

```
! Applied on R1, R2, R3, SW1, SW2, SW3
no cdp run
```

#### Step 4 — Enable LLDP globally and on network-facing interfaces

```
! R1, R2, R3
lldp run
interface range GigabitEthernet0/0-2
 lldp transmit
 lldp receive

! SW1
lldp run
interface GigabitEthernet0/1
 lldp transmit
 lldp receive

! SW2
lldp run
interface GigabitEthernet0/2
 lldp transmit
 lldp receive

! SW3
lldp run
interface FastEthernet0/1
 lldp transmit
 lldp receive
interface FastEthernet0/24
 lldp transmit
 lldp receive
```

---

### Verification

| Check | Command | Expected Result |
|-------|---------|-----------------|
| CDP discovery | `show cdp neighbors detail` | All neighbors with IP addresses shown |
| CDP disabled | `show cdp` | CDP is not enabled |
| LLDP neighbors | `show lldp neighbors` | R1 sees R2, R3, SW1 as LLDP neighbors |
| LLDP interface status | `show lldp interface` | Not supported in Packet Tracer |

---

### Troubleshooting Encountered

| Issue | Cause | Resolution |
|-------|-------|------------|
| `show lldp interface` rejected in Packet Tracer | Command not supported in this version of Packet Tracer | Used `show lldp neighbors` to verify LLDP operation instead |

---

### Key Observations

- CDP is enabled by default on all Cisco devices — no configuration needed to use it for discovery
- `show cdp neighbors detail` reveals IP addresses, IOS version, native VLAN, and duplex of neighbors — the basic `show cdp neighbors` does not show IP addresses
- `no cdp run` disables CDP globally — `no cdp enable` on an interface disables it only on that interface
- CDP and LLDP messages are processed and discarded by the receiving device — never forwarded — so only directly connected neighbors appear
- LLDP requires two separate interface commands (`lldp transmit` and `lldp receive`) unlike CDP which uses a single `cdp enable`
- LLDP uses `B` for Bridge (switch) in capability codes — CDP uses `S` for Switch
- In Packet Tracer `show lldp interface` is not supported — use `show lldp neighbors` to verify

---

### Skills Demonstrated

- CDP-based network discovery (`show cdp neighbors detail`)
- Selective CDP disable on PC-facing interfaces (`no cdp enable`)
- Global CDP disable (`no cdp run`)
- LLDP global and interface configuration (`lldp run`, `lldp transmit`, `lldp receive`)
- LLDP neighbor verification (`show lldp neighbors`)
- Identifying active interfaces using physical device view

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 2.3 | Configure and verify Layer 2 discovery protocols (CDP and LLDP) |

---

### Files

- `Lab24-CDP_LLDP.pkt`
