# CCNA-LABS

## Lab 22 – Standard ACLs (Numbered & Named)

### Objective

Configure OSPF to enable full connectivity between all devices, then implement standard ACLs to enforce four network access policies. Practice both standard numbered ACLs (on R1) and standard named ACLs (on R2), applying the rule that standard ACLs should be placed as close to the destination as possible.

---

### Topology

- 2x Cisco 2911 Routers (R1, R2) — connected via Serial0/0/0 (203.0.113.0/30)
- 4x Cisco 2960-24TT Switches (SW1, SW2, SW3, SW4)
- 4x PCs (PC1–PC4) and 2x Servers (SRV1, SRV2)
- R1 LANs: 172.16.1.0/24 (G0/0), 172.16.2.0/24 (G0/1)
- R2 LANs: 192.168.1.0/24 (G0/0), 192.168.2.0/24 (G0/1)

---

### Network Policies (Requirements)

| # | Policy |
|---|--------|
| 1 | Only PC1 and PC3 can access 192.168.1.0/24 |
| 2 | Hosts in 172.16.2.0/24 cannot access 192.168.2.0/24 |
| 3 | 172.16.1.0/24 cannot access 172.16.2.0/24 |
| 4 | 172.16.2.0/24 cannot access 172.16.1.0/24 |

---

### Configuration Summary

#### Step 1 — OSPF Configuration

```
! R1
router ospf 1
 network 203.0.113.0 0.0.0.3 area 0

interface range GigabitEthernet0/0-1
 ip ospf 1 area 0

! R2
router ospf 1
 network 203.0.113.0 0.0.0.3 area 0

interface range GigabitEthernet0/0-1
 ip ospf 1 area 0
```

#### Step 2 — Standard Named ACLs on R2 (close to destination)

**ACL: TO_192.168.1.0/24 — applied outbound on R2 G0/0**
```
ip access-list standard TO_192.168.1.0/24
 permit 172.16.1.1
 permit 172.16.2.1
 deny any

interface GigabitEthernet0/0
 ip access-group TO_192.168.1.0/24 out
```

**ACL: TO_192.168.2.0/24 — applied outbound on R2 G0/1**
```
ip access-list standard TO_192.168.2.0/24
 deny 172.16.2.0 0.0.0.255
 permit any

interface GigabitEthernet0/1
 ip access-group TO_192.168.2.0/24 out
```

#### Step 3 — Standard Numbered ACLs on R1 (close to destination)

**ACL 1 — blocks 172.16.1.0/24 from reaching 172.16.2.0/24, applied outbound on G0/1**
```
access-list 1 deny 172.16.1.0 0.0.0.255
access-list 1 permit any

interface GigabitEthernet0/1
 ip access-group 1 out
```

**ACL 2 — blocks 172.16.2.0/24 from reaching 172.16.1.0/24, applied outbound on G0/0**
```
access-list 2 deny 172.16.2.0 0.0.0.255
access-list 2 permit any

interface GigabitEthernet0/0
 ip access-group 2 out
```

---

### Verification

| Check | Command | Expected Result |
|-------|---------|-----------------|
| OSPF adjacency | `show ip ospf neighbor` | R1 and R2 FULL state via Serial0/0/0 |
| Routing table | `show ip route` | R1 learns 192.168.x.x, R2 learns 172.16.x.x via OSPF |
| ACL status | `show access-lists` | All four ACLs present with match counts |
| Policy 1 test | PC1 ping SRV1 (192.168.1.100) | Success |
| Policy 1 test | PC2 ping SRV1 | Fail — denied by TO_192.168.1.0/24 |
| Policy 2 test | PC3 ping SRV2 (192.168.2.100) | Fail — denied by TO_192.168.2.0/24 |
| Policy 3/4 test | PC1 ping PC3 | Fail — denied by ACL 1 on R1 |

---

### Troubleshooting Encountered

| Issue | Cause | Resolution |
|-------|-------|------------|
| Wrong network statement entered (203.0.113.254) | Typo while clearing previous command | Used `no network 203.0.113.254 0.0.0.3 area 0` to remove, then entered correct statement |
| `ospf router 1` rejected | Command syntax wrong — correct is `router ospf 1` | Corrected command order |
| `sh acc-lists` rejected | Abbreviation not recognized | Used full command `show access-lists` |

---

### Key Observations

- Standard ACLs match on **source IP only** — placement close to the destination is critical to avoid over-blocking traffic to other networks
- Named ACLs use `ip access-list standard [name]` to enter config mode — numbered ACLs use `access-list [number]` in global config
- The `ip access-group` command applies the ACL to an interface — `ACCESS-GROUP` not `ACCESS-LIST`
- Match counts in `show access-lists` confirm the ACL is actively filtering traffic — hit counts increment in real time
- The `deny any` at the end of TO_192.168.1.0/24 is explicit but technically redundant — the implicit deny would cover it. Including it is good practice for clarity.
- OSPF serial link uses point-to-point network type — no DR/BDR election (shown as dash in `show ip ospf neighbor`)
- Using `ip ospf 1 area 0` directly on interfaces instead of the `network` command keeps config cleaner — both methods work

---

### Skills Demonstrated

- OSPF configuration using `ip ospf area` interface command
- Standard numbered ACL configuration and application
- Standard named ACL configuration and application
- ACL placement rule (standard = close to destination)
- Implicit deny awareness
- `show access-lists` verification with match count analysis
- Multi-ACL policy enforcement on a single router

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 5.6 | Configure and verify access control lists |
| 3.4 | Configure and verify single area OSPFv2 |

---

### Files

- `Lab22-Standard_ACLs.pkt`
