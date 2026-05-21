# CCNA-LABS

## Lab 23 – Extended ACLs

### Objective

Configure extended ACLs to enforce three network access policies using both named extended ACLs on R1 and R2. Practice protocol and port matching (IP, TCP, UDP), correct ACL placement close to the source, and editing ACL entries using named ACL config mode.

---

### Topology

- 2x Cisco 2911 Routers (R1, R2) — connected via Serial0/0/0 (203.0.113.0/30)
- 4x Cisco 2960-24TT Switches
- 4x PCs (PC1–PC4), 2x Servers (SRV1, SRV2)
- R1 LANs: 172.16.1.0/24 (G0/0 — PC1), 172.16.2.0/24 (G0/1 — PC2, PC3, PC4)
- R2 LANs: 192.168.1.0/24 (G0/0 — SRV1 at .100), 192.168.2.0/24 (G0/1 — SRV2 at .100)

---

### Network Policies

| # | Policy |
|---|--------|
| 1 | Hosts in 172.16.2.0/24 can't communicate with PC1 (172.16.1.1) |
| 2 | Hosts in 172.16.1.0/24 can't access DNS service on SRV1 (192.168.1.100) |
| 3 | Hosts in 172.16.2.0/24 can't access HTTP or HTTPS services on SRV2 (192.168.2.100) |

---

### Configuration Summary

#### R1 — Extended Named ACL (policies 1 and 3 combined)

```
ip access-list extended BLOCK_FROM_172.16.2.0
 deny ip 172.16.2.0 0.0.0.255 host 172.16.1.1
 deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 80
 deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 443
 permit ip any any

interface GigabitEthernet0/1
 ip access-group BLOCK_FROM_172.16.2.0 in
```

#### R2 — Extended Named ACL (policy 2)

```
ip access-list extended BLOCK_DNS_SRV1
 deny udp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53
 permit ip any any

interface GigabitEthernet0/0
 ip access-group BLOCK_DNS_SRV1 out
```

---

### Verification

| Check | Command | Expected Result |
|-------|---------|-----------------|
| ACL entries R1 | `show access-lists` on R1 | BLOCK_FROM_172.16.2.0 with 4 entries, permit at bottom |
| ACL entries R2 | `show access-lists` on R2 | BLOCK_DNS_SRV1 with 2 entries |
| Interface check R1 | `show ip interface g0/1` | Inbound ACL = BLOCK_FROM_172.16.2.0 |
| Interface check R2 | `show ip interface g0/0` | Outbound ACL = BLOCK_DNS_SRV1 |
| Policy 1 test | PC2/PC3 ping 172.16.1.1 | Fail — denied by entry 10 |
| Policy 3 test | PC2/PC3 HTTP/HTTPS to SRV2 | Fail — denied by entries 20/30 |
| Policy 2 test | PC1 DNS query to SRV1 port 53 | Fail — denied by BLOCK_DNS_SRV1 |

---

### Troubleshooting Encountered

| Issue | Cause | Resolution |
|-------|-------|------------|
| `permit ip any any` inserted at sequence 20, before HTTP/HTTPS deny entries | Two separate ACLs were merged — permit was added to first ACL before HTTP/HTTPS entries were moved in | Used named ACL config mode: `no 20` to delete the permit, then re-added it — appended at sequence 50 after all deny entries |
| Two ACLs created (BLOCK_FROM_172.16.2.0 and BLOCK_HTTP/S) for same interface/direction | Only one ACL allowed per interface per direction | Merged HTTP/HTTPS entries into BLOCK_FROM_172.16.2.0, deleted BLOCK_HTTP/S with `no ip access-list extended BLOCK_HTTP/S` |
| `ip access-group BLOCK_FROM_172.16.2.0` rejected — incomplete command | Missing direction keyword | Added `in` — full command: `ip access-group BLOCK_FROM_172.16.2.0 in` |
| DNS ACL placed on R2 outbound G0/0 rather than R1 inbound G0/0 | Source (172.16.1.0/24) is on R1 but applying close to destination (SRV1) was chosen due to single interface constraint | Valid placement — outbound on R2 G0/0 correctly filters traffic before it reaches SRV1 |

---

### Key Observations

- Extended ACLs match on protocol, source IP, destination IP, and optionally source/destination port — much more precise than standard ACLs
- Only **one ACL per interface per direction** — when two policies target the same source subnet, they must be combined into a single ACL
- The `permit ip any any` entry **must be at the end** of the ACL — if placed before deny entries those denies will never be reached
- Named ACL config mode allows deleting individual entries with `no [sequence-number]` and appending new entries — critical for fixing order issues without deleting the whole ACL
- IOS automatically converts port numbers to protocol keywords: eq 80 → eq www, eq 53 → eq domain
- Extended ACL placement = close to source. DNS policy placed outbound on R2 G0/0 (close to destination) was still valid given the topology constraints

---

### Skills Demonstrated

- Extended named ACL configuration (ip, tcp, udp protocols)
- Port number matching (eq 80, eq 443, eq 53)
- ACL placement rule for extended ACLs (close to source)
- Editing ACL entries using named ACL config mode (no [seq#])
- Merging two ACLs into one to comply with one-ACL-per-direction rule
- `show access-lists` and `show ip interface` verification

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 5.6 | Configure and verify access control lists |

---

### Files

- `Lab23-Extended_ACLs.pkt`
