# CCNA-LABS

## Lab 26 – DNS (Domain Name System)

### Objective

Configure DNS on a three-PC LAN topology. Configure a default route on R1 to reach the Internet, set 1.1.1.1 (Cloudflare) as the DNS server for all hosts and R1, build a static host table on R1, and use Packet Tracer simulation mode to observe DNS and ICMP traffic flow when pinging youtube.com by name.

---

### Topology

- 1x Cisco 2911 Router (R1)
- 1x Cisco 2960 Switch (SW1)
- 3x PC (PC1, PC2, PC3)
- 1x Server-PT (1.1.1.1 — Cloudflare DNS)
- 1x Server-PT (youtube.com — 172.217.6.78)
- 1x Internet cloud device (router)

```
1.1.1.1 (DNS) ─┐
                INTERNET (203.0.113.0/30) ── R1 (G0/0: .1) ── G0/1: 192.168.0.254
youtube.com ───┘                                                      |
                                                                    SW1
                                                               /     |     \
                                                            PC1    PC2    PC3
                                                          (.1)   (.2)   (.3)
```

---

### Configuration Summary

#### Step 1 — Configure Default Route on R1

```
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

Verification:
```
R1(config)# do ping 1.1.1.1    ! First two packets may fail (ARP) — 3/5 success is normal
```

#### Step 2 — Configure DNS Server on PC1, PC2, PC3

Configured via GUI: Config tab → DNS Server → 1.1.1.1 on each PC.

#### Step 3 — Configure R1 DNS Settings and Host Table

```
R1(config)# ip name-server 1.1.1.1            ! R1 uses Cloudflare as its DNS server
R1(config)# ip host R1  192.168.0.254         ! Static host entries for local devices
R1(config)# ip host PC1 192.168.0.1
R1(config)# ip host PC2 192.168.0.2
R1(config)# ip host PC3 192.168.0.3
```

Verification:
```
R1# show hosts                  ! Confirms perm entries for R1, PC1, PC2, PC3
R1# ping PC1                    ! Case-sensitive in Packet Tracer — use uppercase
```

#### Step 4 — Simulation Mode: Observe DNS + ICMP Flow

From PC1 CLI in simulation mode:
```
C:\> ping youtube.com
```

---

### Verification

| Device | Command | Expected Result |
|--------|---------|-----------------|
| R1 | `ping 1.1.1.1` | Reachable (3/5 minimum — first packets may fail due to ARP) |
| R1 | `show hosts` | R1, PC1, PC2, PC3 listed as perm entries |
| R1 | `ping PC1` | Resolves to 192.168.0.1, 5/5 success |
| PC1 | `ping 1.1.1.1` | 4/4 success — confirms DNS server is reachable |

---

### Traffic Flow Observed (Simulation Mode)

When PC1 pings youtube.com by name, two distinct exchanges occur:

**DNS Query (Trip 1):**
PC1 → SW1 → R1 → Internet → 1.1.1.1 (DNS server)
1.1.1.1 replies with youtube.com = 172.217.6.78
Response travels back: Internet → R1 → SW1 → PC1

**ICMP Ping (Trip 2):**
PC1 now has the IP — sends ping to 172.217.6.78
PC1 → SW1 → R1 → Internet → youtube.com server
youtube.com replies, same path in reverse

Key observation: the DNS query and the ICMP ping are two separate exchanges. The ping cannot be sent until the DNS resolution completes.

---

### Troubleshooting Encountered

- `sh hosts` and `ip host` cannot be run from privileged EXEC without `do` — must be in global config or use `do` prefix
- `ping pc1` (lowercase) fails in Packet Tracer — host table entries are case-sensitive; must use `ping PC1`
- `conft` typo triggered a DNS lookup attempt since `ip name-server` was already configured — Packet Tracer tried to resolve "conft" as a hostname
- `ip name-server` must be entered from global config mode, not privileged EXEC

---

### Key Observations

- Routers require **zero DNS configuration** to forward DNS traffic — R1 forwarded DNS packets between PC1 and 1.1.1.1 before any DNS config was applied
- `ip name-server` makes the router a DNS **client** — it can resolve names itself but does NOT serve queries to other hosts
- `ip dns server` (not available in Packet Tracer) would be needed to make R1 serve DNS queries to internal hosts
- `show hosts` flags: **perm** = manually configured via `ip host`; **temp** = learned via DNS query, will expire
- The 1.1.1.1 server in Packet Tracer has pre-configured A records for one.one.one.one and youtube.com (172.217.6.78)
- DNS uses UDP — no TCP connection is established between PC1 and the DNS server during the query

---

### Skills Demonstrated

- Default route configuration and verification
- DNS client configuration on Cisco IOS using `ip name-server`
- Static host table configuration using `ip host`
- DNS verification using `show hosts` and name-based pings
- Packet Tracer simulation mode traffic analysis
- Understanding of DNS query/response flow separate from ICMP

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 4.3 | Explain the role of DHCP and DNS within the network |

---

### Files

- `Lab26-DNS.pkt`
