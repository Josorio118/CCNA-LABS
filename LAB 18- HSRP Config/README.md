# CCNA-LABS

## Lab 18 – HSRP Configuration (First Hop Redundancy)

### Objective

Configure HSRP version 2 on R1 and R2 to provide a redundant default gateway for the 10.0.1.0/24 subnet. Verify failover behavior when R1 goes down and confirm preemption causes R1 to reclaim the active role when it comes back online.

---

### Topology

- 2x Cisco 2911 Routers (R1, R2) — redundant default gateways
- 1x Cisco 2911 Router (R3) — simulated ISP (8.8.8.8 loopback)
- 4x Cisco 2960-24TT Switches (SW1, SW2, SW3, SW4)
- 2x PCs (PC1, PC2) — 10.0.1.0/24
- HSRP Virtual IP: 10.0.1.254
- R1 G0/0: 10.0.1.253/24 (Active router)
- R2 G0/0: 10.0.1.252/24 (Standby router)

---

### Configuration Summary

#### R1 — Active Router (Priority 200, Preempt enabled)

```
interface GigabitEthernet0/0
 standby version 2
 standby 1 ip 10.0.1.254
 standby 1 priority 200
 standby 1 preempt
```

#### R2 — Standby Router (Default Priority 100)

```
interface GigabitEthernet0/0
 standby version 2
 standby 1 ip 10.0.1.254
 standby 1 priority 100
```

#### PC Default Gateway Update

- PC1 Default Gateway: 10.0.1.254 (HSRP VIP)
- PC2 Default Gateway: 10.0.1.254 (HSRP VIP)

---

### Lab Steps and Results

#### Step 1 — Verify Initial Connectivity
- PC1 and PC2 pinged 8.8.8.8 successfully
- Default gateway was 10.0.1.253 (R1's actual interface IP)

#### Step 2 — Configure HSRP v2 on R1 and R2
- R1 configured as active with priority 200 and preemption enabled
- R2 configured as standby with priority 100
- HSRP version must match — version mismatch causes both routers to claim active role simultaneously

#### Step 3 — Update PC Default Gateways to VIP
- Both PCs updated to use 10.0.1.254 as default gateway
- Pings to 8.8.8.8 successful
- ARP table on PC1 confirmed VIP mapped to HSRP v2 virtual MAC: `0000.0c9f.f001`

#### Step 4 — Simulate R1 Failure
- R1 powered off via Physical tab (after saving config with `write memory`)
- First ping packet dropped during HSRP failover — expected behavior
- Subsequent pings successful via R2 (10.0.1.252)
- R2 became the active router automatically

#### Step 5 — R1 Recovery with Preemption
- R1 powered back on
- Preemption caused R1 to reclaim the active role from R2
- `show standby` confirmed R1 state = Active, priority 200, standby = R2 at 10.0.1.252

---

### Verification

| Check | Command | Expected Result |
|-------|---------|-----------------|
| HSRP state | `show standby` | R1 = Active, R2 = Standby |
| Virtual MAC | `arp -a` on PC | 10.0.1.254 → 0000.0c9f.f001 |
| Connectivity | `ping 8.8.8.8` from PC | Successful |
| Failover | Power off R1, ping 8.8.8.8 | 1 dropped packet then success via R2 |
| Preemption | Power R1 back on, `show standby` | R1 reclaims Active role |

---

### Troubleshooting Encountered

| Issue | Cause | Resolution |
|-------|-------|------------|
| `write memory` rejected from interface config mode | Must be run from privileged EXEC mode | Typed `end` to exit to `#` prompt first |
| Both routers claimed Active simultaneously | HSRP version mismatch — R1 on v2, R2 on v1 | Configured `standby version 2` on R2 |
| Priority 300 briefly configured on R1 | Typo — entered twice before correct value | Corrected to `standby 1 priority 200` |

---

### Key Observations

- HSRP uses a **virtual IP** and **virtual MAC** — end hosts use the VIP as their default gateway, not the router's actual IP
- HSRP version 1 and version 2 are **not compatible** — both routers must use the same version or both will claim Active
- HSRP v2 virtual MAC format: `0000.0c9f.fXXX` (XXX = group number in hex) — group 1 = `0000.0c9f.f001`
- When standby becomes active, it sends **gratuitous ARP** to update switch MAC address tables
- **Preemption** must be configured on the router that should reclaim the active role after recovery — the standby does not need it
- Traceroute shows the router's **actual interface IP**, not the virtual IP — useful for verifying which router is currently active
- FHRPs are **non-preemptive by default** — without `standby preempt`, R1 would remain standby even after recovering
- One packet dropped during failover is normal — HSRP takes a few seconds to detect failure and complete the transition

---

### Skills Demonstrated

- HSRP v2 configuration on router interfaces
- Active/standby election using priority
- Preemption configuration and verification
- Virtual IP and virtual MAC behavior
- HSRP failover simulation and verification
- ARP table analysis for FHRP MAC identification
- `show standby` output interpretation

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 3.5 | Describe the purpose of first hop redundancy protocols |

---

### Files

- `Lab18-HSRP_Configuration.pkt`
