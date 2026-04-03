# CCNA-LABS

## Lab 01 – Inter-VLAN Routing (Router-on-a-Stick)

### Objective
Design and validate a segmented Layer 2 network using multiple VLANs and enable Layer 3 communication using a router-on-a-stick architecture.

---

### Topology
- 1x Cisco 2960 Switch
- 1x Cisco 2911 Router
- 2x End Hosts in separate VLANs
- 802.1Q trunk between switch and router

---

### VLAN Design

| VLAN | Name | Subnet |
|-----:|------|--------|
| 10 | USER | 192.168.10.0/24 |
| 20 | TEST | 192.168.20.0/24 |
| 99 | MGMT | 192.168.99.0/24 |

---

### Configuration Summary

**Switch:**
- Created VLANs 10, 20, and 99
- Assigned access ports per VLAN
- Configured trunk port toward router with 802.1Q encapsulation

**Router:**
- Created subinterfaces for each VLAN
- Applied `encapsulation dot1q` per subinterface
- Assigned gateway IPs on each subinterface

**Hosts:**
- Statically assigned IPs within their respective subnets
- Default gateways mapped to router subinterfaces

---

### Verification

| Command | Purpose |
|---------|---------|
| `show vlan brief` | Confirm VLAN creation and port assignments |
| `show interfaces trunk` | Verify trunk status and allowed VLANs |
| `show ip interface brief` | Confirm subinterface IPs are up |
| `ping` | Validate inter-VLAN connectivity end-to-end |

---

### Key Observations
- Intra-VLAN traffic forwarded at Layer 2 without routing
- Inter-VLAN traffic required the router subinterface as the default gateway
- ARP had to resolve the router MAC before first ping completed
- Trunk link carried tagged frames for all VLANs over a single physical cable

---

### Skills Demonstrated
- VLAN segmentation and access port assignment
- 802.1Q trunking configuration
- Router-on-a-stick subinterface design
- Static IP addressing and default gateway configuration
- End-to-end connectivity verification via ICMP

---

### CCNA Exam Alignment
- **2.1** – Configure and verify VLANs
- **2.3** – Configure and verify inter-switch connectivity (trunking)
- **3.1** – Interpret the components of a routing table

---

### Files
- `Inter-VLAN-Routing-Router-on-a-Stick.pkt`k.pkt`
