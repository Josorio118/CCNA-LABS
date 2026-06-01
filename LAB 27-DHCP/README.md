# CCNA-LABS

## Lab 27 – DHCP (Dynamic Host Configuration Protocol)

### Objective

Configure R2 as a DHCP server with three pools, R1's G0/0 interface as a DHCP client, and R1 as a DHCP relay agent for the 192.168.1.0/24 subnet. Verify PC1 and PC2 receive IP addresses dynamically from R2.

---

### Topology

- 2x Cisco 2911 Routers (R1, R2)
- 2x Cisco 2960 Switches (SW1, SW2)
- 2x PC (PC1, PC2)

```
SW2 (192.168.2.0/24) -- R2 (G0/1: .1) -- G0/0 (203.0.113.0/30) -- R1 (G0/0: .2) -- G0/1: 192.168.1.1 -- SW1 -- PC1
                                                                                                     PC2 connected to SW2
```

| Device | Interface | IP Address |
|--------|-----------|------------|
| R2 | G0/1 | 192.168.2.1/24 |
| R2 | G0/0 | 203.0.113.1/30 |
| R1 | G0/0 | 203.0.113.2/30 (DHCP client) |
| R1 | G0/1 | 192.168.1.1/24 |
| PC1 | NIC | 192.168.1.12/24 (DHCP) |
| PC2 | NIC | 192.168.2.11/24 (DHCP) |

---

### Configuration Summary

#### Step 1 — Configure DHCP Pools on R2

```
! Exclude reserved addresses before creating pools (global config mode)
R2(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
R2(config)# ip dhcp excluded-address 192.168.2.1 192.168.2.10
R2(config)# ip dhcp excluded-address 203.0.113.1

! POOL1 — for 192.168.1.0/24 (PC1 subnet, served via relay)
R2(config)# ip dhcp pool POOL1
R2(dhcp-config)# network 192.168.1.0 255.255.255.0
R2(dhcp-config)# dns-server 8.8.8.8
R2(dhcp-config)# domain-name jeremysitlab.com
R2(dhcp-config)# default-router 192.168.1.1

! POOL2 — for 192.168.2.0/24 (PC2 subnet, directly connected to R2)
R2(config)# ip dhcp pool POOL2
R2(dhcp-config)# network 192.168.2.0 255.255.255.0
R2(dhcp-config)# dns-server 8.8.8.8
R2(dhcp-config)# domain-name jeremysitlab.com
R2(dhcp-config)# default-router 192.168.2.1

! POOL3 — for 203.0.113.0/30 (R1's G0/0 DHCP client link)
R2(config)# ip dhcp pool POOL3
R2(dhcp-config)# network 203.0.113.0 255.255.255.252
```

Verification:
```
R2(config)# do show run | section dhcp
```

#### Step 2 — Configure R1's G0/0 as a DHCP Client

```
R1(config)# interface g0/0
R1(config-if)# ip address dhcp
R1(config-if)# no shutdown
```

R1 was assigned 203.0.113.2/30 from R2's POOL3.

#### Step 3 — Configure R1 as DHCP Relay Agent

```
R1(config)# interface g0/1          ! Client-facing interface
R1(config-if)# ip helper-address 203.0.113.1    ! R2's IP (DHCP server)
```

Verification:
```
R1# show ip interface g0/1          ! Confirms "Helper address is 203.0.113.1"
```

#### Step 4 — Verify PC1 and PC2 Get IP Addresses

```
! PC2 (directly connected to R2)
C:\> ipconfig /renew                ! Received 192.168.2.11

! PC1 (behind R1 relay agent)
C:\> ipconfig /renew                ! May need multiple attempts in Packet Tracer
                                    ! Received 192.168.1.12
```

---

### Verification

| Device | Command | Expected Result |
|--------|---------|-----------------|
| R2 | `show run \| section dhcp` | All three pools configured, excluded ranges present |
| R2 | `show ip dhcp binding` | PC1 and PC2 listed with assigned IPs |
| R1 | `show ip interface g0/0` | Address determined by DHCP (203.0.113.2) |
| R1 | `show ip interface g0/1` | Helper address 203.0.113.1 listed |
| PC1 | `ipconfig /renew` | 192.168.1.12, gateway 192.168.1.1, DNS 8.8.8.8 |
| PC2 | `ipconfig /renew` | 192.168.2.11, gateway 192.168.2.1, DNS 8.8.8.8 |

---

### Troubleshooting Encountered

- `ip dhcp pool POOL 2` (with a space) fails — pool names cannot contain spaces
- `network 192.168.1.0` without subnet mask gives "incomplete command" — mask is required
- PC1's first `ipconfig /renew` attempts failed — ARP process is slow in Packet Tracer; ran the command multiple times until successful
- `ip helper-address` requires the DHCP **server** IP (203.0.113.1), not the client subnet address

---

### Key Observations

- `ip dhcp excluded-address` is configured from **global config mode** — not inside the DHCP pool. Entering it from DHCP config mode will fail
- POOL3 for the 203.0.113.0/30 link between R1 and R2 required no DNS, domain-name, or default-router — only the network statement
- PC2 got .11 (first available after .1–.10 excluded). PC1 got .12 — .11 may have been attempted and released during failed DHCP requests before succeeding
- R1's G0/0 went through the full DORA process to get its IP from R2's POOL3 — confirmed by the `%DHCP-6-ADDRESS_ASSIGN` message in the console
- The relay agent (R1) forwards PC1's broadcast DHCP Discover as a **unicast** message to R2 at 203.0.113.1 — broadcasts don't cross router boundaries without a relay

---

### Skills Demonstrated

- DHCP server configuration on Cisco IOS (pools, excluded addresses, network, DNS, domain, default-router)
- DHCP client configuration on a router interface using `ip address dhcp`
- DHCP relay agent configuration using `ip helper-address`
- DHCP verification using `show run | section dhcp`, `show ip dhcp binding`, `show ip interface`
- Understanding of the DORA process and why relay agents are needed in multi-subnet environments

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 4.3 | Explain the role of DHCP and DNS within the network |
| 4.6 | Configure and verify DHCP client and relay |

---

### Files

- `Lab27-DHCP.pkt`
