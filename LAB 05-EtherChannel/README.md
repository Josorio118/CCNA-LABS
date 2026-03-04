# CCNA-LABS

## Lab 05 – EtherChannel (Layer 2 & Layer 3)

### Objective
Configure and verify Layer 2 and Layer 3 EtherChannels across a multi-switch topology using LACP, PAgP, and Static EtherChannel. The goal was to eliminate STP-blocked redundant links and enable full bandwidth utilization across all physical connections.

---

### Topology
- 2x Access Layer Switches (ASW1, ASW2) — Cisco 2960
- 2x Distribution Layer Switches (DSW1, DSW2) — Cisco 3650
- 2x End Hosts (PC1, PC2)
- 1x Server (SRV1)

---

### EtherChannel Design

| Port-Channel | Switches | Protocol | Type | Purpose |
|-------------|----------|----------|------|---------|
| Po1 | ASW1 ↔ DSW1 | LACP (active/active) | Layer 2 | Access to distribution uplink |
| Po1 | ASW2 ↔ DSW2 | PAgP (desirable/desirable) | Layer 2 | Access to distribution uplink |
| Po2 | DSW1 ↔ DSW2 | Static (on/on) | Layer 3 | Distribution interconnect |

---

### IP Addressing

| Interface | Device | IP Address |
|-----------|--------|------------|
| Port-channel2 | DSW1 | 10.0.0.1/30 |
| Port-channel2 | DSW2 | 10.0.0.2/30 |
| VLAN1 SVI | DSW1 | 172.16.1.254/24 |
| VLAN1 SVI | DSW2 | 172.16.2.254/24 |

---

### Configuration Summary

**Step 1 – Layer 2 EtherChannel: ASW1 ↔ DSW1 (LACP)**

ASW1:
```
interface range g0/1 - 2
 channel-group 1 mode active

interface port-channel 1
 switchport mode trunk
```

DSW1:
```
interface range g1/0/3 - 4
 channel-group 1 mode active

interface port-channel 1
 switchport trunk encapsulation dot1q
 switchport mode trunk
```

---

**Step 2 – Layer 2 EtherChannel: ASW2 ↔ DSW2 (PAgP)**

ASW2:
```
interface range g0/1 - 2
 channel-group 1 mode desirable

interface port-channel 1
 switchport mode trunk
```

DSW2:
```
interface range g1/0/3 - 4
 channel-group 1 mode desirable

interface port-channel 1
 switchport trunk encapsulation dot1q
 switchport mode trunk
```

---

**Step 3 – Layer 3 EtherChannel: DSW1 ↔ DSW2 (Static)**

DSW1:
```
interface range g1/0/1 - 2
 no switchport
 channel-group 2 mode on

interface port-channel 2
 ip address 10.0.0.1 255.255.255.252
```

DSW2:
```
interface range g1/0/1 - 2
 no switchport
 channel-group 2 mode on

interface port-channel 2
 ip address 10.0.0.2 255.255.255.252
```

---

**Step 4 – Static Routes & IP Routing**

DSW1:
```
ip routing
ip route 172.16.2.0 255.255.255.0 10.0.0.2
```

DSW2:
```
ip routing
ip route 172.16.1.0 255.255.255.0 10.0.0.1
```

---

**Step 5 & 6 – Load Balancing**

Default method on both switch models: `src-mac`

Changed to source + destination IP on all switches:
```
port-channel load-balance src-dst-ip
```

Verified with:
```
show etherchannel load-balance
```

---

### Verification Commands

| Command | Purpose |
|---------|---------|
| `show etherchannel summary` | Primary verification — flags, protocol, member ports |
| `show etherchannel load-balance` | View current load-balancing method |
| `show interfaces trunk` | Confirm port-channel trunk status |
| `show spanning-tree` | Verify STP sees port-channel as single logical link |
| `show ip route` | Confirm routing table after enabling ip routing |

---

### Key Observations
- STP blocked redundant links before EtherChannel — all extra ports were orange
- After EtherChannel all member ports showed green — STP sees one logical link
- Layer 3 EtherChannel required `no switchport` on physical interfaces AND the port-channel interface before IP assignment
- `ip routing` must be enabled on multilayer switches before routing table is built
- IOS-XE 16.x shows additional Local (L) routes — normal behavior, not an error

---

### Troubleshooting Encountered
**Issue:** Po2 showing `SD` (Layer 2, Down) instead of `RU` (Layer 3, In Use)

**Cause:** Port-channel interface was created as Layer 2 by default before `no switchport` was applied

**Fix:**
```
interface po2
 no switchport

interface range g1/0/1 - 2
 no switchport
 channel-group 2 mode on
```

> Key lesson: For Layer 3 EtherChannel, `no switchport` must be applied to BOTH the physical member interfaces AND the port-channel interface before assigning an IP address.

---

### Skills Demonstrated
- Layer 2 EtherChannel with LACP and PAgP
- Layer 3 EtherChannel with static mode
- EtherChannel load-balancing configuration and verification
- Static routing on multilayer switches
- Real-world troubleshooting of L2/L3 mismatch errors

---

### CCNA Exam Alignment
- **2.4** – Configure and verify Layer 2 and Layer 3 EtherChannel (LACP)

---

### Files
- `LAB05-EtherChannel.pkt`
