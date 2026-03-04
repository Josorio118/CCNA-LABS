# CCNA-LABS

## Lab 02 – Spanning Tree Protocol (STP) Analysis

### Objective
Analyze and verify Spanning Tree Protocol behavior in a multi-switch topology using Cisco Packet Tracer. Focus on root bridge election, port role determination, and CLI-based verification — without relying on visual indicators.

---

### Topology
- 4x Cisco Layer 2 Switches (interconnected in a mesh)
- Single VLAN (VLAN 1)
- STP Mode: IEEE 802.1D / PVST (classic spanning tree)
- Link lights disabled to force logical analysis over visual bias

---

### STP Analysis Process

#### Step 1 – Root Bridge Identification
Root bridge elected by comparing Bridge IDs (priority + MAC address).

| Switch | Priority | Result |
|--------|----------|--------|
| SW3 | 24577 | ✅ Root Bridge (lowest Bridge ID) |

- SW3 had the lowest bridge priority and was elected root
- All SW3 interfaces are **Designated Forwarding**

#### Step 2 – Root Port Selection
Each non-root switch selects one root port — the interface with the lowest total path cost to root.

Tie-breaker order applied:
1. Lowest root path cost
2. Lowest neighbor bridge ID
3. Lowest neighbor port ID

| Switch | Root Port |
|--------|-----------|
| SW1 | F0/4 |
| SW2 | G0/1 |
| SW4 | G0/2 |

#### Step 3 – Designated vs. Non-Designated Ports
For each shared segment, STP elects one designated port (forwarding) and one non-designated port (blocking).

| Interface | Role | State |
|-----------|------|-------|
| SW2 F0/1 | Designated | Forwarding |
| SW2 F0/2 | Designated | Forwarding |
| SW1 F0/1 | Non-Designated | Blocking |
| SW1 F0/2 | Non-Designated | Blocking |

Blocking ports were intentional loop-prevention — not errors.

---

### Verification Commands

| Command | Purpose |
|---------|---------|
| `show spanning-tree` | View root bridge, port roles, and states |
| `show spanning-tree vlan 1` | Filter output to VLAN 1 |
| `show spanning-tree detail` | Confirm total root path cost |
| `show spanning-tree summary` | High-level STP status across all VLANs |

---

### Key Observations
- SW3 correctly identified itself as root in the Root ID section
- Port roles matched all calculated results (root, designated, blocking)
- Default STP timers confirmed: Hello 2s, Max Age 20s, Forward Delay 15s
- CLI verification was essential — link lights alone were not reliable

---

### Skills Demonstrated
- Root bridge election using bridge ID logic
- Root, designated, and non-designated port identification
- STP tie-breaker rule application (cost → bridge ID → port ID)
- CLI-based STP verification and output interpretation

---

### CCNA Exam Alignment
- **2.5** – Describe the need for and basic operations of Rapid PVST+ spanning tree protocol

---

### Files
- `Analyzing_STP_Lab.pkt`
