# CCNA-LABS

## Lab 03 – Spanning Tree Protocol (STP) Root Manipulation & Port Protection

### Objective
Demonstrate IEEE 802.1D STP behavior in a multi-switch topology with redundant links. Manually manipulate root bridge election, influence path selection, and configure edge port protection features.

---

### Topology
- 4x Cisco Catalyst 2960 Switches (redundant Layer 2 links)
- VLANs: VLAN 1 and VLAN 2
- Protocol: IEEE 802.1D STP
- Platform: Cisco Packet Tracer

---

### Step 1 – Initial STP Topology Verification

**Command used:** `show spanning-tree`

**Initial Root Bridge:** SW2 (MAC: 0001.4301.4B81) — lowest Bridge ID

| Switch | Interface | Role | State |
|--------|-----------|------|-------|
| SW1 | Fa0/1 | Alternate | Blocking |
| SW1 | Fa0/2 | Designated | Forwarding |
| SW1 | Fa0/3 | Root | Forwarding |
| SW2 (Root) | Fa0/1–3 | Designated | Forwarding |
| SW3 | Fa0/1 | Designated | Forwarding |
| SW3 | Fa0/2 | Root | Forwarding |
| SW4 | Fa0/1 | Root | Forwarding |
| SW4 | Fa0/2 | Alternate | Blocking |

---

### Step 2 – Root Bridge Manipulation Per VLAN

**Configuration:**
```
SW1(config)# spanning-tree vlan 1 root primary
SW1(config)# spanning-tree vlan 2 root secondary

SW2(config)# spanning-tree vlan 2 root primary
SW2(config)# spanning-tree vlan 1 root secondary
```

**Result:**

| VLAN | Root Bridge |
|------|-------------|
| VLAN 1 | SW1 |
| VLAN 2 | SW2 |

- Per-VLAN load balancing achieved
- STP recalculated — SW3 Fa0/2 became Alternate Blocking to prevent loop

---

### Step 3 – Path Cost Manipulation

**Configuration (SW4):**
```
interface f0/2
 spanning-tree vlan 1 cost 100
```

**Result:** SW4 selected Fa0/1 as root port instead of Fa0/2

**Why:** STP selects root port based on lowest total path cost. Increasing Fa0/2 cost forced STP to recalculate and prefer Fa0/1.

---

### Step 4 – Port Priority Manipulation

**Configuration (SW1):**
```
interface f0/1
 spanning-tree vlan 1 port-priority 240
```

**Result:** SW3 did not change its root port

**Why:** Port priority is only a tie-breaker when path costs are equal. Since cost was unchanged, priority had no effect.

> STP decision order: Root Bridge ID → Path Cost → Sender Bridge ID → Port Priority → Port Number

---

### Step 5 – PortFast & BPDU Guard

**Configuration (SW3 & SW4 Fa0/3):**
```
interface f0/3
 spanning-tree portfast
 spanning-tree bpduguard enable
```

**Observed behavior:** When a BPDU was received, the port entered **err-disabled** state

**Recovery:**
```
interface f0/3
 shutdown
 no shutdown
```

---

### Verification Commands

| Command | Purpose |
|---------|---------|
| `show spanning-tree` | View port roles, states, and root bridge |
| `show spanning-tree vlan 1` | Per-VLAN STP detail |
| `show errdisable recovery` | Check err-disable status and timers |

---

### Key Observations
- `root primary` sets priority to 24576
- `root secondary` sets priority to 28672
- Path cost directly affects root port selection
- Port priority only matters when costs are tied
- BPDU Guard protects PortFast ports from rogue switch connections
- STP dynamically recalculates when metrics change

---

### Skills Demonstrated
- STP root bridge election and manual manipulation
- Per-VLAN STP load balancing
- Path cost and port priority influence
- PortFast and BPDU Guard configuration
- Err-disable observation and recovery

---

### CCNA Exam Alignment
- **2.5** – Describe the need for and basic operations of Rapid PVST+ spanning tree protocol

---

### Files
- `Configuring_Spanning_Tree.pkt`
