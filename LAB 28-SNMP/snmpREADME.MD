# CCNA-LABS

## Lab 28 – SNMP (Simple Network Management Protocol)

### Objective

Configure SNMP community strings on R1 and use the MIB Browser on PC1 (acting as the NMS) to send SNMP Get messages to retrieve device information, and an SNMP Set message to change R1's hostname remotely.

> **Note:** SNMP functionality is very limited in Packet Tracer. The `snmp-server host` command is unavailable, so trap destinations cannot be configured. This lab focuses on community string configuration and MIB browser interaction only.

---

### Topology

- 1x Cisco 2911 Router (R1)
- 1x Cisco 2960 Switch (SW1)
- 1x PC (PC1) — acts as NMS

```
R1 (192.168.1.254) -- G0/0 -- SW1 -- PC1 (192.168.1.1)
```

---

### Configuration Summary

#### Step 1 — Configure SNMP Communities on R1

```
R1(config)# snmp-server community Cisco1 ro    ! Read-only community string
R1(config)# snmp-server community Cisco2 rw    ! Read/write community string
```

#### Step 2 — Configure MIB Browser on PC1

- Desktop tab → MIB Browser
- Address: 192.168.1.254
- Click Advanced → Read Community: Cisco1 | Write Community: Cisco2 | SNMP Version: v1

#### Step 3 — SNMP Get Queries from PC1

| OID Path | Variable | Result |
|----------|----------|--------|
| iso → org → dod → internet → mgmt → mib-2 → system → sysUpTime | System uptime | 10 hours 10 minutes 57 seconds |
| iso → org → dod → internet → mgmt → mib-2 → system → sysName | Hostname | R1 |
| iso → org → dod → internet → mgmt → mib-2 → interfaces → ifNumber | Interface count | 4 |
| iso → org → dod → internet → mgmt → mib-2 → interfaces → ifTable → ifEntry → ifDescr | Interface names | Vlan1, GigabitEthernet0/0, G0/1, G0/2 |

#### Step 4 — SNMP Set to Change Hostname

- Select sysName OID
- Switch operation to Set
- Data type: OctetString
- Value: ROUTER
- Click GO

Verification on R1 CLI:
```
ROUTER# do show run | include hostname
hostname ROUTER
```

---

### Verification

| Action | Expected Result |
|--------|-----------------|
| Get sysUpTime | Returns uptime in hours/minutes/seconds |
| Get sysName | Returns current hostname (R1) |
| Get ifNumber | Returns 4 |
| Get ifDescr | Returns Vlan1, G0/0, G0/1, G0/2 |
| Set sysName to ROUTER | R1 CLI prompt changes to ROUTER, confirmed in running-config |

---

### Troubleshooting Encountered

- `snmp-server community` requires the `community` keyword — `snmp-server Cisco1 ro` fails
- `sh run` cannot be run from config mode without `do` prefix
- ifDescr is nested inside ifTable → ifEntry — clicking ifTable itself triggers a Set dialog; must expand to ifEntry → ifDescr
- Accidentally switched operation to Set while navigating ifTable — cancelled and corrected

---

### Key Observations

- SNMP uses **UDP port 161** for agent queries (NMS → managed device) and **UDP port 162** for trap/inform messages (managed device → NMS)
- Community strings in SNMPv1 are sent in **plain text** — insecure; SNMPv3 adds encryption
- RO community (Cisco1) allows Get operations only. RW community (Cisco2) allows both Get and Set
- SNMP Set messages can change live device configuration — the hostname change via sysName OID was immediately reflected in R1's running-config
- MIB variables are organized hierarchically by OID — iso.org.dod.internet.mgmt.mib-2 is the base path for standard interface and system variables
- `snmp-server host` (to configure trap destination) is not available in Packet Tracer

---

### Skills Demonstrated

- SNMP community string configuration on Cisco IOS
- MIB browser usage to send SNMP Get messages and retrieve OID values
- SNMP Set operation to remotely modify device configuration
- Understanding of MIB hierarchy and OID navigation
- Identifying the difference between RO and RW community access

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 4.4 | Explain the function of SNMP in network operations |

> SNMP configuration commands are **not** on the CCNA exam topics list. This lab covers the conceptual operation of SNMP only.

---

### Files

- `Lab28-SNMP.pkt`
