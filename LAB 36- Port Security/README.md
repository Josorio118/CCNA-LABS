# CCNA-LABS

## Lab 36 – Port Security

### Objective

Configure port security on SW1's access ports (F0/1, F0/2, F0/3) with shutdown violation mode, and on SW2's G0/1 uplink with restrict violation mode and sticky MAC learning. Trigger violations on both switches and observe the different behaviors of each violation mode.

---

### Topology

- 2x Cisco 2960-24TT Switches (SW1, SW2)
- 1x Cisco 2911 Router (R1) — 10.0.0.254
- 3x PC (PC1, PC2, PC3) connected to SW1 F0/1, F0/2, F0/3
- 1x PC (PC4) connected to SW2

```
PC1 (.1) - F0/1 -|
PC2 (.2) - F0/2 |- SW1 - G0/1 - G0/1 - SW2 - G0/2 - G0/0 - R1 (.254)
PC3 (.3) - F0/3 -|
```

Network: 10.0.0.0/24

---

### Configuration Summary

#### Step 1: Port Security on SW1 (F0/1, F0/2, F0/3)

```
SW1(config)# interface f0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport port-security
SW1(config-if)# switchport port-security maximum 1
SW1(config-if)# switchport port-security violation shutdown
SW1(config-if)# switchport port-security aging time 60

! Repeat for F0/2 and F0/3 with same settings
```

Default violation mode is already shutdown and default maximum is already 1 — these were configured explicitly to match the lab instructions.

#### Step 2: Port Security on SW2 (G0/1)

```
SW2(config)# interface g0/1
SW2(config-if)# switchport mode access
SW2(config-if)# switchport port-security
SW2(config-if)# switchport port-security maximum 4
SW2(config-if)# switchport port-security violation restrict
SW2(config-if)# switchport port-security mac-address sticky
```

After pinging from each PC to R1, SW2 learned 4 sticky MACs: PC1, PC2, PC3, and SW1's physical G0/1 MAC.

---

### Triggering Violations

#### SW1 Shutdown Violation

Configured SW1's VLAN1 SVI with an IP address:
```
SW1(config)# interface vlan 1
SW1(config-if)# ip address 10.0.0.10 255.255.255.0
SW1(config-if)# no shutdown
```

Pinged R1 from SW1. The SVI MAC (0001.0001.000A) was seen by SW2 as a 5th address, triggering SW2's restrict violation. Simultaneously, a new PC connected to F0/1 on SW1 caused a shutdown violation on F0/1.

#### SW2 Restrict Violation

When SW1's SVI sent traffic through G0/1, SW2 saw a 5th source MAC exceeding the maximum of 4. Port stayed up, traffic was blocked, syslog messages were generated, and violation counter incremented.

---

### Verification

| Device | Command | Expected Result |
|--------|---------|-----------------|
| SW1 | `show port-security` | F0/1, F0/2, F0/3 listed; max 1, action shutdown |
| SW1 | `show port-security interface f0/1` | Secure-shutdown after violation; violation count 1 |
| SW2 | `show port-security interface g0/1` | Secure-up; restrict mode; 4 sticky MACs learned |
| SW2 | `show port-security interface g0/1` (after violation) | Violation count incremented; port still up |
| SW2 | `show mac-address` | 4 sticky MACs shown as STATIC type |

#### Post-Violation Results

| Switch | Interface | Status | Violation Count | Unauthorized MAC |
|--------|-----------|--------|-----------------|-----------------|
| SW1 | F0/1 | Secure-shutdown | 1 | 0001.0001.000A |
| SW2 | G0/1 | Secure-up | 6 | 0060.7024.2366 |

---

### Troubleshooting Encountered

- `switcport mode trunk` typo rejected; corrected to `switchport mode trunk`
- `sh port-security int f0/1` rejected from global config mode without `do` prefix
- `int v;lan 1` typo (semicolon) rejected; corrected to `int vlan 1`
- `sh mac-address table` rejected in Packet Tracer; correct command is `show mac-address` (no "table")
- SW2's G0/1 was initially configured as trunk; changed to access since port security requires static access or trunk mode and the lab uses access

---

### Key Observations

- Shutdown mode: interface goes err-disabled immediately upon violation; syslog message generated; violation counter set to 1; traffic completely stopped
- Restrict mode: port stays up; unauthorized frames are dropped; syslog messages generated per violation (6 messages for 6 frames); violation counter incremented with each blocked frame
- Sticky MACs appear as type STATIC in the MAC address table even though they were dynamically learned
- Sticky MACs are saved to running-config automatically; `write memory` is required to preserve them through a reboot
- SW2 learned 4 sticky MACs from the three PCs and SW1's physical interface MAC; the 5th MAC (SW1's SVI) triggered the restrict violation
- Pings from SW1's SVI (10.0.0.10) to R1 failed because SW2 blocked the traffic in restrict mode

---

### Skills Demonstrated

- Port security configuration with shutdown and restrict violation modes
- Sticky MAC address learning and verification
- MAC address aging configuration
- Triggering and observing port security violations
- Identifying secure-shutdown vs secure-up port states
- Understanding how sticky MACs are stored in running-config as static entries

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 5.7 | Configure and verify Layer 2 security features (DHCP snooping, dynamic ARP inspection, and port security) |

---

### Files

- `Lab36-Port-Security.pkt`
