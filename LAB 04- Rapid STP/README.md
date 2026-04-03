# CCNA-LABS

## Lab 04 – Rapid Spanning Tree Protocol (RSTP) Analysis

### Objective
Analyze Rapid Spanning Tree Protocol behavior in a multi-switch topology. Identify the root bridge, determine port roles and states without the CLI first, then verify results and configure appropriate RSTP link types and edge ports.

---

### Topology
- 4x Cisco 2960 Switches
- 1x Hub (shared collision domain)
- Multiple PCs on access ports
- Single VLAN (VLAN 1)
- All switch priorities: 32769 (32768 + VLAN 1)
- Root election determined by lowest MAC address (all priorities equal)

---

### Step 1 – Root Bridge Identification

**Command used:** `show spanning-tree`

**Root Bridge: SW1**
```
Root ID  Priority  32769
         Address   0005.5E4E.714B
         This bridge is the root
```

**Important observation — Backup Port:**
SW1 had two interfaces connected to the same hub (same collision domain).
Only one can be designated — the other became a **Backup port**.

```
Fa0/3  Back  BLK  Shr
```

> Key refinement: The root bridge has one designated port per collision domain — not one per interface.

---

### Step 2 – Port Role and State Analysis

| Switch | Interface | Role | State | Type |
|--------|-----------|------|-------|------|
| SW2 | Fa0/1 | Root | Forwarding | — |
| SW2 | Fa0/2 | Designated | Forwarding | — |
| SW2 | Gi0/1 | Alternate | Blocking | P2p |
| SW3 | Fa0/2 | Root | Forwarding | Shr |
| SW3 | Fa0/1 | Designated | Forwarding | — |
| SW3 | Gi0/1 | Designated | Forwarding | — |
| SW4 | Fa0/1 | Root | Forwarding | — |
| SW4 | Fa0/2 | Alternate | Blocking | P2p |

---

### Step 3 – RSTP Link Type Configuration

**RSTP Link Types:**

| Type | When Used |
|------|-----------|
| Point-to-Point | Full-duplex switch-to-switch links |
| Shared | Hub connections (half-duplex) |
| Edge | PortFast-enabled access ports |

**SW4:**
```
interface range f0/1 - 2
 spanning-tree link-type point-to-point

interface f0/24
 spanning-tree portfast
```

**SW3:**
```
interface f0/24
 spanning-tree portfast
```
> Fa0/2 auto-detected as Shared due to hub connection

**SW2:**
```
interface range f0/23 - 24
 spanning-tree portfast
```

**SW1:**
```
interface f0/24
 spanning-tree portfast
```

> SW1 Fa0/24 is connected to a hub (Shared) but also has PortFast enabled (Edge).
> In Packet Tracer only `Shr` appears. On real hardware it would display `Edge Shr`.
> A port can be both Shared and Edge simultaneously.

---

### Verification Commands

| Command | Purpose |
|---------|---------|
| `show spanning-tree` | Verify root bridge, port roles, and link types |
| `spanning-tree mode rapid-pvst` | Confirm RSTP mode |
| `spanning-tree link-type point-to-point` | Set P2P link type |
| `spanning-tree portfast` | Configure edge port |

---

### Key Observations
- RSTP uses protocol version 2 — all switches originate their own BPDUs
- Non-designated role split into Alternate (backs up root port) and Backup (backs up designated port on same switch)
- Port states simplified to: Discarding, Learning, Forwarding
- PortFast, UplinkFast, and BackboneFast behavior built into RSTP by default

---

### Skills Demonstrated
- RSTP root bridge identification
- Alternate and Backup port role distinction
- RSTP link type configuration (P2P, Shared, Edge)
- PortFast configuration on access ports
- CLI verification of RSTP behavior

---

### CCNA Exam Alignment
- **2.5** – Describe the need for and basic operations of Rapid PVST+ spanning tree protocol

---

### Files
- `Rapid_STP_Lab.pkt`
