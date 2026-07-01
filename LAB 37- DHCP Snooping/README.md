# CCNA-LABS

## Lab 37 – DHCP Snooping

### Objective

Configure R1 as a DHCP server for the 192.168.1.0/24 subnet, then enable DHCP snooping on SW1 and SW2 with trusted uplink interfaces. Verify that PC1 can successfully obtain an IP address via DHCP through the snooping-enabled switches.

---

### Topology

- 1x Cisco 2911 Router (R1) — DHCP server at 192.168.1.1
- 2x Cisco 2960-24TT Switches (SW1, SW2)
- 3x PC (PC1, PC2, PC3)

```
R1 (.1) - G0/0 - G0/2 - SW1 - G0/1 - G0/1 - SW2 - F0/1 - PC1
                                                   - F0/2 - PC2
                                                   - F0/3 - PC3
```

Network: 192.168.1.0/24

---

### Configuration Summary

#### Step 1: Configure R1 as DHCP Server

```
R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.9
R1(config)# ip dhcp pool POOL1
R1(dhcp-config)# network 192.168.1.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.1.1
```

#### Step 2: Configure DHCP Snooping on SW1

```
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 1
SW1(config)# no ip dhcp snooping information option
SW1(config)# interface g0/2
SW1(config-if)# ip dhcp snooping trust
```

#### Step 3: Configure DHCP Snooping on SW2

```
SW2(config)# ip dhcp snooping
SW2(config)# ip dhcp snooping vlan 1
SW2(config)# no ip dhcp snooping information option
SW2(config)# interface g0/1
SW2(config-if)# ip dhcp snooping trust
```

#### Step 4: Verify PC1 Gets IP Address

```
C:\> ipconfig /renew
```

PC1 received 192.168.1.10 (first available address after .1-.9 excluded).

---

### Verification

| Device | Command | Expected Result |
|--------|---------|-----------------|
| SW1 | `do show ip dhcp snooping` | Snooping enabled, VLAN 1 active, Option 82 disabled, G0/2 trusted |
| SW2 | `do show ip dhcp snooping` | Snooping enabled, VLAN 1 active, Option 82 disabled, G0/1 trusted |
| PC1 | `ipconfig /renew` | IP 192.168.1.10, gateway 192.168.1.1 |

---

### Troubleshooting Encountered

- `ip dhcp snooping` cannot be run from privileged EXEC mode; must be in global config
- `no ip dhcp snooping information option` accidentally entered as `ip dhcp snooping information option` first on SW2; corrected immediately with the `no` form
- `show ip dhcp snooping` must be run with `do` prefix from config mode

---

### Key Observations

- Two commands are required to activate DHCP snooping: `ip dhcp snooping` (global enable) AND `ip dhcp snooping vlan 1` (per VLAN); global enable alone is not sufficient
- `no ip dhcp snooping information option` is critical on switches NOT acting as DHCP relay agents; without it, switches insert Option 82 into DHCP messages which causes downstream switches to drop them on untrusted ports
- PC1's DHCP request worked on the first attempt because Option 82 was correctly disabled; Jeremy's lab intentionally omits this step first to demonstrate the failure, then fixes it
- The trusted interface must face toward the DHCP server; SW1 trusts G0/2 (toward R1) and SW2 trusts G0/1 (toward SW1/R1); all other ports remain untrusted
- All untrusted ports drop DHCP server messages (OFFER, ACK, NAK) automatically; only trusted ports forward them
- `ip dhcp excluded-address` is configured in global config mode, NOT inside the pool; the `network` command inside the pool defines the subnet, not the excluded range

---

### Skills Demonstrated

- DHCP server configuration on Cisco IOS
- DHCP snooping global enable and per-VLAN activation
- Trusted port configuration for uplink interfaces
- Option 82 disabling on non-relay switches
- DHCP snooping verification using `show ip dhcp snooping`
- Understanding of trusted vs untrusted port behavior

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 5.7 | Configure and verify Layer 2 security features (DHCP snooping, dynamic ARP inspection, and port security) |

---

### Files

- `Lab37-DHCP-Snooping.pkt`
