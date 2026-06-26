# CCNA-LABS

## Lab 33 – Dynamic NAT and PAT (NAT Overload)

### Objective

Configure dynamic NAT on R1 using a pool of two public IP addresses, observe pool exhaustion when a third PC attempts to reach the Internet, then reconfigure to PAT using R1's interface IP to allow all three PCs to communicate simultaneously.

---

### Topology

Same topology as Lab 32 (Static NAT):

- 1x Cisco 2911 Router (R1)
- 1x Cisco 2960 Switch (SW1)
- 3x PC (PC1, PC2, PC3)
- 1x Internet cloud/router

```
PC1 (172.16.0.1) -|
PC2 (172.16.0.2) |- SW1 - R1 (G0/1: .254, G0/0: 203.0.113.1) - Internet - 8.8.8.8
PC3 (172.16.0.3) -|
```

---

### Configuration Summary

#### Step 1: Configure Inside/Outside Interfaces

```
R1(config)# interface g0/0
R1(config-if)# ip nat outside
R1(config)# interface g0/1
R1(config-if)# ip nat inside
```

#### Step 2: Configure Dynamic NAT with a Pool of Two Addresses

```
! ACL identifies which traffic to translate
R1(config)# access-list 1 permit 172.16.0.0 0.0.0.255

! Define pool of two public IPs
R1(config)# ip nat pool POOL1 100.0.0.1 100.0.0.2 netmask 255.255.255.0

! Map ACL to pool
R1(config)# ip nat inside source list 1 pool POOL1
```

#### Step 3: Observe Pool Exhaustion

Pinged google.com from PC1 and PC2: success.
Pinged google.com from PC3: failed; both pool addresses already in use.

#### Step 4: Change to PAT Using R1's Interface IP

```
! Clear existing translations first
R1# clear ip nat translation *

! New statement replaces pool statement automatically
R1(config)# ip nat inside source list 1 interface g0/0 overload
```

Pinged google.com from all three PCs: all succeeded.

---

### Verification

| Command | Expected Result |
|---------|-----------------|
| `show run \| include nat` | Shows inside/outside interface config, pool, and NAT statement |
| `show ip nat statistics` | Total translations, pool usage, allocated addresses |
| `show ip nat translations` | Dynamic entries showing inside local to inside global mappings with port numbers |
| `clear ip nat translation *` | Clears all dynamic entries |

#### PAT Translation Table (confirmed)

All three PCs mapped to the same inside global IP (203.0.113.1) with unique port numbers:

| Protocol | Inside Global | Inside Local | Outside |
|----------|--------------|--------------|---------|
| icmp | 203.0.113.1:13 | 172.16.0.1:13 | 172.217.175.238 |
| icmp | 203.0.113.1:1024 | 172.16.0.2:13 | 172.217.175.238 |
| icmp | 203.0.113.1:1 | 172.16.0.3:1 | 172.217.175.238 |
| udp | 203.0.113.1:1028 | 172.16.0.1:1028 | 8.8.8.8:53 |
| udp | 203.0.113.1:1024 | 172.16.0.2:1028 | 8.8.8.8:53 |
| udp | 203.0.113.1:1027 | 172.16.0.3:1027 | 8.8.8.8:53 |

---

### Troubleshooting Encountered

- `show run| include nat` without a space before the pipe fails; correct syntax requires a space: `show run | include nat`
- `do show ip nat translations` returned empty table because PCs had not generated traffic yet after clearing; pinged from PCs first, then re-ran the command
- `clear ip nat translation*` without a space before the asterisk fails; correct syntax is `clear ip nat translation *` with a space
- `show ip nat translations` and `show ip nat statistics` cannot be run from global config mode without the `do` prefix

---

### Key Observations

- Dynamic NAT with a pool of two addresses caused PC3 to fail; pool exhaustion drops the packet rather than queuing it
- Entering `ip nat inside source list 1 interface g0/0 overload` automatically replaces the existing pool statement; no need to remove old config first
- With PAT, all three PCs share 203.0.113.1 as the inside global address; R1 differentiates flows using unique port numbers per session
- Both ICMP and UDP entries appear in the translation table; UDP entries are for DNS queries (port 53 to 8.8.8.8)
- Outside local and outside global addresses remain identical since only source NAT is configured

---

### Skills Demonstrated

- Dynamic NAT configuration using ACL and address pool
- Pool exhaustion behavior observation
- PAT (NAT overload) configuration using interface IP
- NAT table verification and interpretation
- Understanding how PAT uses port numbers to track multiple simultaneous sessions from different inside hosts

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 4.1 | Configure and verify inside source NAT using static and pools |

---

### Files

- `Lab33-Dynamic-NAT-PAT.pkt`
