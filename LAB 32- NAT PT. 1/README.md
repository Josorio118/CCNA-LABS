# CCNA-LABS

## Lab 32 – Static NAT

### Objective

Configure static NAT on R1 to allow three PCs with private IP addresses to reach the Internet. Map each PC's private IP to a unique public IP, verify translations, test connectivity, and clear the NAT translation table.

---

### Topology

- 1x Cisco 2911 Router (R1)
- 1x Cisco 2960 Switch (SW1)
- 3x PC (PC1, PC2, PC3)
- 1x Internet cloud/router

```
PC1 (172.16.0.1) -|
PC2 (172.16.0.2) |- SW1 - R1 (G0/1: .254, G0/0: 203.0.113.x) - Internet - 8.8.8.8
PC3 (172.16.0.3) -|
```

| Device | Interface | IP Address |
|--------|-----------|------------|
| R1 | G0/1 | 172.16.0.254/24 |
| R1 | G0/0 | 203.0.113.x/30 |
| PC1 | NIC | 172.16.0.1/24 |
| PC2 | NIC | 172.16.0.2/24 |
| PC3 | NIC | 172.16.0.3/24 |

---

### Configuration Summary

#### Step 1: Verify Ping Fails Before NAT

```
PC1> ping 8.8.8.8    ! All 4 packets timeout; ISP drops private source IPs
```

#### Step 2: Configure Static NAT on R1

```
! Configure inside/outside interfaces
R1(config)# interface g0/0
R1(config-if)# ip nat outside
R1(config)# interface g0/1
R1(config-if)# ip nat inside

! Configure static one-to-one mappings
R1(config)# ip nat inside source static 172.16.0.1 100.0.0.1
R1(config)# ip nat inside source static 172.16.0.2 100.0.0.2
R1(config)# ip nat inside source static 172.16.0.3 100.0.0.3
```

#### Step 3: Verify and Test

```
R1# show ip nat translations    ! View static mappings in NAT table
R1# show ip nat statistics      ! View total translations and interfaces
```

#### Step 4: Ping from each PC

```
PC1> ping 8.8.8.8       ! Success
PC1> ping google.com    ! DNS resolves then pings succeed
PC2> ping google.com    ! Success
PC3> ping google.com    ! Success
```

#### Step 5: Clear Dynamic NAT Entries

```
R1# do clear ip nat translation *    ! Clears dynamic entries only
R1# do show ip nat translations      ! Static entries remain; dynamic entries gone
```

---

### Verification

| Test | Expected Result |
|------|-----------------|
| `show ip nat translations` | Three static entries mapping 172.16.0.x to 100.0.0.x |
| `show ip nat statistics` | 3 total translations; inside/outside interfaces listed |
| Ping 8.8.8.8 from PC1 | Success after NAT configured |
| Ping google.com from each PC | Success; DNS entries appear as dynamic translations |
| After `clear ip nat translation *` | Only static entries remain in table |

---

### Troubleshooting Encountered

- `clear ip nat translation *` cannot be run from global config mode; must use `do clear ip nat translation *` or exit to privileged EXEC first
- Several syntax variations attempted before correct command accepted

---

### Key Observations

- Before NAT was configured, pings to 8.8.8.8 failed because the ISP drops packets with private source IP addresses
- Static NAT creates permanent one-to-one mappings; entries always present in the translation table regardless of traffic
- When PCs ping google.com, additional dynamic entries appear for the DNS queries (UDP port 53 to 8.8.8.8) alongside the ICMP ping entries
- Outside local and outside global addresses are identical because only source NAT is configured; destination NAT is not in use
- `clear ip nat translation *` removes dynamic entries only; static mappings created with `ip nat inside source static` are permanent and stay in the table
- Static NAT is bidirectional; an outside host could initiate a connection to the inside global IP and R1 would forward it to the corresponding inside local address

---

### Skills Demonstrated

- Static NAT configuration (inside/outside interfaces + one-to-one mappings)
- NAT table verification using `show ip nat translations` and `show ip nat statistics`
- Understanding of inside local vs inside global address terminology
- Dynamic NAT entry clearing with `clear ip nat translation *`
- DNS resolution behavior visible in NAT translation table

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 4.1 | Configure and verify inside source NAT using static and pools |

---

### Files

- `Lab32-Static-NAT.pkt`
