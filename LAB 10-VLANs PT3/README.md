# CCNA-LABS

## Lab 10 – VLANs Part 3 (Multilayer Switch & SVIs)

### Objective
Replace a Router-on-a-Stick configuration with Layer 3 inter-VLAN routing using Switch Virtual Interfaces (SVIs) on a multilayer switch. Configure a point-to-point Layer 3 link between the multilayer switch and an upstream router, and verify both inter-VLAN and internet connectivity.

---

### Topology
- 1x Cisco 2911 Router (R1) — connected to internet
- 1x Cisco 2960-24TT Switch (SW1) — Layer 2 access switch
- 1x Cisco 3650-24PS Multilayer Switch (SW2) — Layer 3 inter-VLAN routing via SVIs
- 7x End Hosts (PC1–PC7)
- Trunk link between SW1 and SW2
- Point-to-point Layer 3 link between SW2 and R1

---

### VLAN Design

| VLAN | Subnet | SVI IP (Gateway) |
|-----:|--------|-----------------|
| 10 | 10.0.0.0/26 | 10.0.0.62 |
| 20 | 10.0.0.64/26 | 10.0.0.126 |
| 30 | 10.0.0.128/26 | 10.0.0.190 |

### Point-to-Point Link

| Device | Interface | IP Address |
|--------|-----------|-----------|
| SW2 | Gig1/0/2 | 10.0.0.193/30 |
| R1 | Gig0/0 | 10.0.0.194/30 |

---

### Configuration Summary

**R1 — Remove ROAS, configure point-to-point IP:**
```
no interface g0/0.10
no interface g0/0.20
no interface g0/0.30

interface g0/0
 ip address 10.0.0.194 255.255.255.252
```

**SW2 — Convert uplink to routed port:**
```
default interface g1/0/2

interface g1/0/2
 no switchport
 ip address 10.0.0.193 255.255.255.252
```

**SW2 — Enable routing and add default route:**
```
ip routing
ip route 0.0.0.0 0.0.0.0 10.0.0.194
```

**SW2 — SVI configuration:**
```
interface vlan 10
 ip address 10.0.0.62 255.255.255.192
 no shutdown

interface vlan 20
 ip address 10.0.0.126 255.255.255.192
 no shutdown

interface vlan 30
 ip address 10.0.0.190 255.255.255.192
 no shutdown
```

**SW2 — Fix trunk to include VLAN 20:**
```
interface g1/0/1
 switchport trunk allowed vlan add 20
```

---

### Verification

| Command | Purpose |
|---------|---------|
| `show ip interface brief` | Confirm SVI and routed port IPs are up/up |
| `show interfaces trunk` | Verify allowed VLANs on trunk |
| `show ip route` | Confirm connected routes and default route |
| `ping` | Validate inter-VLAN and internet connectivity |

---

### Troubleshooting Encountered

**Issue 1 — VLAN 20 missing from trunk**

After configuring SVIs, VLAN 20 was not in the allowed VLAN list on Gig1/0/1 (trunk to SW1). VLAN 20 hosts on SW1 could not reach SW2's SVI.

**Fix:** `switchport trunk allowed vlan add 20` on Gig1/0/1.

**Key Lesson:** Use `add` when adding VLANs to an existing trunk allowed list. Using `switchport trunk allowed vlan 20` without `add` replaces the entire list with only VLAN 20.

**Issue 2 — Missing `no shutdown` on VLAN 20 SVI**

VLAN 20 SVI was configured but left in shutdown state. SVIs are shutdown by default — `no shutdown` is required on every SVI.

**Issue 3 — `no switchport` must be run in interface config mode**

Attempting `no switchport` from global config mode failed. Must enter the interface first before issuing the command.

**Key Lesson:** `no switchport` converts a Layer 2 switchport to a Layer 3 routed port. It must be issued in interface config mode, not global config mode.

---

### Key Observations
- `ip routing` must be enabled on the multilayer switch before SVIs can route traffic — without it no routing table is built
- SVIs are shutdown by default — always add `no shutdown` after configuring each one
- After `no switchport`, the interface shows as 'routed' in `show interfaces status`
- Inter-VLAN routing now happens inside SW2 — traffic between VLANs no longer travels to R1
- TTL=253 on ping to 1.1.1.1 confirms 3 hops: SW2 → R1 → Internet router
- A local (/32) route may not appear in Packet Tracer routing table — known simulator limitation

---

### Skills Demonstrated
- Removing ROAS configuration and replacing with Layer 3 point-to-point link
- Converting a Layer 2 switchport to a Layer 3 routed port with `no switchport`
- SVI configuration on a multilayer switch
- Enabling IP routing on a multilayer switch
- Default route configuration on a multilayer switch
- Verifying inter-VLAN and internet connectivity

---

### CCNA Exam Alignment
- **2.1c** – InterVLAN connectivity
- **3.1** – Interpret the components of a routing table
- **3.3** – Configure and verify static routing (default route)

---

### Files
- `Lab10- VLANs Pt3.pkt`
