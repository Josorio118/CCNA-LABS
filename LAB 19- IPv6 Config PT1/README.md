# CCNA-LABS

## Lab 19 – IPv6 Configuration Part 1 (Dual-Stack)

### Objective

Configure IPv6 addressing on a Cisco router running IPv4, creating a dual-stack network where both IPv4 and IPv6 operate simultaneously. Verify end-to-end connectivity between hosts across all three subnets using both protocols.

---

### Topology

- 1x Cisco 2911 Router (R1) — IPv4 + IPv6 dual-stack gateway
- 3x Cisco 2960-24TT Switches (SW1, SW2, SW3)
- 3x PCs (PC1, PC2, PC3) — one per subnet
- Pre-configured IPv4 network — IPv6 added on top

---

### Subnet Design

| Interface | IPv4 Subnet | IPv6 Subnet | PC |
|-----------|-------------|-------------|-----|
| R1 G0/0 | 192.168.1.0/24 | 2001:DB8:0:1::/64 | PC1 |
| R1 G0/1 | 192.168.2.0/24 | 2001:DB8:0:2::/64 | PC2 |
| R1 G0/2 | 192.168.3.0/24 | 2001:DB8:0:3::/64 | PC3 |

---

### IP Addressing

#### Router R1

| Interface | IPv4 Address | IPv6 Address |
|-----------|-------------|-------------|
| G0/0 | 192.168.1.1/24 | 2001:DB8:0:1::1/64 |
| G0/1 | 192.168.2.1/24 | 2001:DB8:0:2::1/64 |
| G0/2 | 192.168.3.1/24 | 2001:DB8:0:3::1/64 |

#### End Hosts

| PC | IPv4 Address | IPv4 Gateway | IPv6 Address | IPv6 Gateway |
|----|-------------|-------------|-------------|-------------|
| PC1 | 192.168.1.2/24 | 192.168.1.1 | 2001:DB8:0:1::2/64 | 2001:DB8:0:1::1 |
| PC2 | 192.168.2.2/24 | 192.168.2.1 | 2001:DB8:0:2::2/64 | 2001:DB8:0:2::1 |
| PC3 | 192.168.3.2/24 | 192.168.3.1 | 2001:DB8:0:3::2/64 | 2001:DB8:0:3::1 |

---

### Configuration Summary

#### Enable IPv6 Routing (Required)

```
ipv6 unicast-routing
```

#### R1 IPv6 Interface Configuration

```
interface GigabitEthernet0/0
 ipv6 address 2001:DB8:0:1::1/64

interface GigabitEthernet0/1
 ipv6 address 2001:DB8:0:2::1/64

interface GigabitEthernet0/2
 ipv6 address 2001:DB8:0:3::1/64
```

> No `no shutdown` needed — interfaces already up from existing IPv4 configuration.

---

### Verification

| Check | Command | Expected Result |
|-------|---------|-----------------|
| IPv6 interface status | `show ipv6 interface brief` | All three interfaces up with configured + link-local addresses |
| IPv4 connectivity | `ping 192.168.X.X` from PC | Success (first packet may timeout — ARP) |
| IPv6 connectivity | `ping 2001:DB8:0:X::2` from PC | Success |
| Dual-stack cross-subnet | PC1 → PC2 and PC1 → PC3 both protocols | All successful |

---

### Troubleshooting Encountered

| Issue | Cause | Resolution |
|-------|-------|------------|
| Wrong IPv6 address assigned to G0/1 | Accidentally configured G0/3 address on G0/1 before moving to G0/2 | Used `no ipv6 address 2001:DB8:0:3::1/64` on G0/1 to remove it, then configured G0/2 correctly |
| Overlapping address error on G0/2 | 2001:DB8:0:3::/64 still assigned to G0/1 when trying to add to G0/2 | Removed wrong address from G0/1 first |
| IPv6 address disappearing in PT GUI | Address entered without /64 prefix length | PT has two separate fields — address in the left box, prefix number (64) in the small box after the / |
| Invalid IPv6 gateway error | Entered /64 in the gateway field | Gateway field takes the IPv6 address only — no prefix length |

---

### Key Observations

- `ipv6 unicast-routing` is mandatory — without it the router accepts IPv6 address configuration but will NOT forward packets between subnets
- Dual-stack means IPv4 and IPv6 run simultaneously on the same interfaces with no disruption to existing IPv4 traffic
- Each interface automatically receives a link-local address (FE80::/10) when IPv6 is enabled — these do not need to be configured manually
- IPv6 addresses in Packet Tracer GUI are split across two fields — the address and the prefix length after the /
- The gateway field accepts only the IPv6 address — no prefix length
- First IPv4 ping packet timeout is normal — ARP resolution delay
- IPv6 uses NDP (Neighbor Discovery Protocol) instead of ARP — covered in IPv6 Part 2
- 2001:DB8::/32 is reserved for documentation and examples only

---

### Skills Demonstrated

- IPv6 unicast routing enablement
- IPv6 address configuration on router interfaces
- Dual-stack (IPv4 + IPv6) network design
- IPv6 host configuration in Packet Tracer
- End-to-end IPv6 connectivity verification across multiple subnets
- `show ipv6 interface brief` output interpretation
- IPv6 address troubleshooting (overlapping subnet, wrong interface assignment)

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 1.8 | Configure and verify IPv6 addressing and prefixes |
| 1.9 | Describe IPv6 address types |

---

### Files

- `Lab19-IPv6_Configuration_Part1.pkt`
