# CCNA-LABS

## Lab 29 – Syslog (System Logging Protocol)

### Objective

Practice Syslog configurations on R1 including enabling timestamps, configuring logging to the buffer, enabling logging to an external Syslog server, and demonstrating the difference between console and VTY line logging behavior.

---

### Topology

- 1x Cisco 2911 Router (R1)
- 1x Cisco 2960 Switch (SW1)
- 2x PC (PC1, PC2)
- 1x Server-PT (SRV1) — Syslog server at 192.168.1.100

```
R1 (192.168.1.1) -- G0/1 -- SW1 -- PC1 (192.168.1.1)
                                 -- PC2 (console connection to R1)
                                 -- SRV1 (192.168.1.100)
```

---

### Configuration Summary

#### Step 1 — Console Logging (Default Behavior)

Connected to R1 via PC2's console terminal. Shut down and re-enabled G0/0 to generate Syslog messages — two messages displayed per interface state change (one for STATUS, one for PROTOCOL).

#### Step 2 — Enable Timestamps

```
R1(config)# service timestamps log datetime msec
```

Note: In real Cisco IOS, `msec` is optional. In Packet Tracer it is required.

#### Step 3 — VTY Line Logging via Telnet

Connected to R1 from PC1 using Telnet:
```
C:\> telnet 192.168.1.1
```

Syslog messages do NOT display by default over Telnet. Enabled for current session:
```
R1(config-if)# do terminal monitor
```

Shut down G0/1 — Syslog message now displayed over Telnet session.

#### Step 4 — Enable Buffer Logging

```
R1(config)# logging buffered 8192
```

Verified with:
```
R1# show logging
```

#### Step 5 — Configure External Syslog Server

```
R1(config)# logging host 192.168.1.100
R1(config)# logging trap debugging
```

Generated messages by toggling G0/1 up and down — verified on SRV1 under Services → Syslog.

---

### Verification

| Step | Command | Expected Result |
|------|---------|-----------------|
| Timestamps | `show logging` | Messages include datetime timestamps |
| Buffer | `show logging` | Buffer logging enabled, messages stored |
| Trap level | `show logging` | Trap logging: level debugging |
| Syslog server | SRV1 Services → Syslog | Messages from R1 visible |

---

### Troubleshooting Encountered

- `service timestamps log datetime` without `msec` failed in Packet Tracer — `msec` is required in PT even though it's optional in real IOS
- `show logging` typed as `show loggong` — typo, corrected immediately
- `terminal monitor` must be run from privileged EXEC mode — used `do terminal monitor` from interface config mode to apply during lab

---

### Key Observations

- Console line logging is enabled by default for all severity levels (0–7) — Syslog messages display automatically when connected via console
- VTY line (Telnet/SSH) logging is disabled by default — `terminal monitor` enables it for the current session only; it does not persist after logout
- Each interface state change generates two Syslog messages: one for the line status and one for the protocol status
- `logging buffered 8192` enables buffer logging with an 8192-byte buffer — view contents with `show logging`
- `logging trap debugging` sets severity level 7 (debugging) for messages sent to external server — all levels are sent
- `logging host` and `logging` (IP only) are equivalent commands — both point to the external Syslog server
- Syslog server listens on UDP port 514
- Packet Tracer's Syslog server functionality is primitive but functional for basic message verification

---

### Skills Demonstrated

- Console line Syslog behavior (default on)
- Enabling timestamps with `service timestamps log datetime msec`
- VTY line logging using `terminal monitor`
- Buffer logging configuration and verification
- External Syslog server configuration (`logging host`, `logging trap`)
- Understanding severity levels and which messages are sent at each level
- Generating Syslog messages via interface state changes

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 4.5 | Describe the use of syslog features including facilities and levels |

---

### Files

- `Lab29-Syslog.pkt`
