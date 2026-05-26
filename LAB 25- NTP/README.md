# CCNA-LABS

## Lab XX – NTP (Network Time Protocol)

### Objective

Configure and verify Network Time Protocol (NTP) across a three-router topology. Synchronize R1 to an external NTP server, configure R1 as a stratum 8 NTP master backup, and synchronize R2 and R3 to R1 using NTP authentication.

---

### Topology

- 3x Cisco 2911 Routers (R1, R2, R3)
- 1x Server-PT (SRV1) — external NTP server at 1.1.1.1
- 1x Internet cloud device
- OSPF pre-configured across all routers
- Default route pre-configured on R1

```
SRV1 (1.1.1.1) -- INTERNET -- R1 (G0/0: 203.0.113.1)
                                  |
                    G0/1: 192.168.12.1 --- R2 (192.168.12.2)
                    G0/2: 192.168.13.1 --- R3 (192.168.13.2)
                                              |
                                    R2 G0/1: 192.168.23.1
                                    R3 G0/1: 192.168.23.2
```

---

### Configuration Summary

#### Step 1 — Set Software Clock on All Routers (Privileged EXEC)

```
R1# clock set 12:00:00 30 DEC 2020
R2# clock set 12:00:00 30 DEC 2020
R3# clock set 12:00:00 30 DEC 2020
```

#### Step 2 — Configure Time Zone (Global Config)

```
R1(config)# clock timezone EST -5
R2(config)# clock timezone EST -5
R3(config)# clock timezone EST -5
```

#### Step 3 — Configure R1 as NTP Client to SRV1

```
R1(config)# ntp server 1.1.1.1
```

Verification:
```
R1# show ntp associations     ! Wait for asterisk (*) to appear next to 1.1.1.1
R1# show ntp status           ! Confirms: synchronized, stratum 2, reference 1.1.1.1
R1# show clock detail         ! Confirms: Time source is NTP
```

#### Step 4 — Configure R1 as Stratum 8 NTP Master + Authentication

```
! R1 — master + authentication setup
R1(config)# ntp master
R1(config)# ntp authenticate
R1(config)# ntp authentication-key 1 md5 PASSWORD
R1(config)# ntp trusted-key 1

! R2 — full auth + point to R1's G0/1 interface
R2(config)# ntp authenticate
R2(config)# ntp authentication-key 1 md5 PASSWORD
R2(config)# ntp trusted-key 1
R2(config)# ntp server 192.168.12.1 key 1

! R3 — full auth + point to R1's G0/2 interface
R3(config)# ntp authenticate
R3(config)# ntp authentication-key 1 md5 PASSWORD
R3(config)# ntp trusted-key 1
R3(config)# ntp server 192.168.13.1 key 1
```

#### Step 5 — Configure NTP to Update Hardware Calendar

```
R1(config)# ntp update-calendar
R2(config)# ntp update-calendar
R3(config)# ntp update-calendar
```

Note: `show calendar` is not available in Packet Tracer — `ntp update-calendar` was configured but cannot be visually verified in this environment.

---

### Verification

| Device | Command | Expected Result |
|--------|---------|-----------------|
| R1 | `show ntp status` | Synchronized, stratum 2, reference 1.1.1.1 |
| R1 | `show ntp associations` | Asterisk on 1.1.1.1 (stratum 1) and 127.127.1.1 (ntp master) |
| R1 | `show clock detail` | Time source is NTP |
| R2 | `show ntp status` | Synchronized, stratum 3, reference 192.168.12.1 |
| R2 | `show ntp associations` | Asterisk on 192.168.12.1 |
| R2 | `show clock detail` | Time source is NTP |
| R3 | `show ntp status` | Synchronized, stratum 3, reference 192.168.13.1 |
| R3 | `show ntp associations` | Asterisk on 192.168.13.1 |
| R3 | `show clock detail` | Time source is NTP |

---

### Troubleshooting Encountered

- `show clock details` is not valid — correct command is `show clock detail` (no 's')
- `ntp server` and `ntp authenticate` must be entered from global config mode — not privileged EXEC
- NTP synchronization takes time in Packet Tracer — used the fast-forward simulation button; `show ntp status` initially showed stratum 16 (unsynchronized) before syncing
- `show calendar` is unavailable in Packet Tracer — `ntp update-calendar` was configured but cannot be verified visually
- `ntp source` command is also unavailable in Packet Tracer — used physical interface IPs instead of a loopback address for R2/R3 NTP server configuration

---

### Key Observations

- `clock set` is run from **privileged EXEC** and is not saved to running-config; `clock timezone` is run from **global config** and is saved
- After NTP syncs, `show clock detail` changes from `Time source is user configuration` to `Time source is NTP`
- R1 stratum = 2 (synced to stratum 1 server). R2 and R3 stratum = 3 (synced to stratum 2 R1)
- `ntp master` with no stratum specified defaults to stratum 8 — provides a fallback clock if R1 loses contact with SRV1
- `show ntp associations` on R1 shows two entries: `*~1.1.1.1` (selected upstream server) and `~127.127.1.1` (the ntp master loopback reference)
- NTP authentication requires four steps in order: `ntp authenticate` → `ntp authentication-key` → `ntp trusted-key` → `ntp server ... key X` (server command only on clients)
- The `ntp source` command (for specifying loopback as NTP source) is not available in Packet Tracer — in real networks, loopback interfaces are preferred as NTP source for stability

---

### Skills Demonstrated

- Manual software clock and time zone configuration on Cisco IOS
- NTP client configuration using `ntp server`
- NTP master clock configuration using `ntp master`
- NTP authentication setup (authenticate, key creation, trusted-key, server key reference)
- Hardware calendar synchronization via `ntp update-calendar`
- NTP verification using `show ntp status`, `show ntp associations`, and `show clock detail`
- Understanding of NTP stratum hierarchy and how stratum propagates through a network

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 4.2 | Configure and verify NTP operating in client and server mode |

---

### Files

- `LabXX-NTP.pkt`
