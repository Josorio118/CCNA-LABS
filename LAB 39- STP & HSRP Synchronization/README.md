# Lab 39 – STP and HSRP Synchronization

## Objective

Configure HSRP on two distribution layer switches and ensure the HSRP active router for each VLAN is also the STP root bridge for that VLAN. This synchronization guarantees that traffic from end hosts always takes the most direct path to their active default gateway rather than crossing an extra hop through the redundant link.

Requirements:

- VLAN 10: DSW1 is HSRP active and STP root, DSW2 is HSRP standby and STP secondary root
- VLAN 20: DSW2 is HSRP active and STP root, DSW1 is HSRP standby and STP secondary root

## Topology

- 2x Cisco 3650-24PS multilayer switches (DSW1, DSW2) acting as the distribution layer
- 2x Cisco 2960-24TT switches (ASW1, ASW2) acting as the access layer
- DSW1 and DSW2 are cross-connected to both access switches (DSW1 to ASW1 via Gi0/1, DSW1 to ASW2 via Gi0/2, DSW2 to ASW2 via Gi0/1, DSW2 to ASW1 via Gi0/2), providing redundant uplinks
- DSW1 and DSW2 are also directly connected to each other via Gi1/0/2 and Gi1/0/3
- PC1 in VLAN 10 connects to ASW1, PC2 in VLAN 20 connects to ASW2

## VLAN Design

| VLAN | Subnet | DSW1 SVI | DSW2 SVI | HSRP Virtual IP |
|------|--------|----------|----------|------------------|
| 10 | 10.0.10.0/24 | 10.0.10.1 | 10.0.10.2 | 10.0.10.254 |
| 20 | 10.0.20.0/24 | 10.0.20.1 | 10.0.20.2 | 10.0.20.254 |

## Configuration Summary

**DSW1**

```
interface vlan 10
 standby version 2
 standby 10 ip 10.0.10.254
 standby 10 priority 200
 standby 10 preempt

interface vlan 20
 standby version 2
 standby 20 ip 10.0.20.254
 standby 20 priority 95
 standby 20 preempt

spanning-tree vlan 10 root primary
spanning-tree vlan 20 root secondary
```

**DSW2**

```
interface vlan 20
 standby version 2
 standby 20 ip 10.0.20.254
 standby 20 priority 200
 standby 20 preempt

interface vlan 10
 standby version 2
 standby 10 ip 10.0.10.254
 standby 10 priority 95
 standby 10 preempt

spanning-tree vlan 20 root primary
spanning-tree vlan 10 root secondary
```

## Verification

**HSRP status (`show standby brief` / `show standby`)**

| Switch | VLAN | Group | State | Priority | Virtual IP |
|--------|------|-------|-------|----------|------------|
| DSW1 | 10 | 10 | Active | 200 | 10.0.10.254 |
| DSW1 | 20 | 20 | Standby | 95 | 10.0.20.254 |
| DSW2 | 10 | 10 | Standby | 95 | 10.0.10.254 |
| DSW2 | 20 | 20 | Active | 200 | 10.0.20.254 |

**STP status (`show spanning-tree vlan 10` / `show spanning-tree vlan 20`)**

| Switch | VLAN | Role | Root Priority | Root Port |
|--------|------|------|----------------|-----------|
| DSW1 | 10 | Root | 24586 | N/A (this bridge is root) |
| DSW1 | 20 | Secondary | 28692 | Gi1/0/3 |
| DSW2 | 10 | Secondary | 28682 | Gi1/0/3 |
| DSW2 | 20 | Root | 24596 | N/A (this bridge is root) |

Both switches confirm HSRP active/standby roles align with STP root/secondary root roles per VLAN. Each switch's root port for the VLAN it does not own correctly points to the other distribution switch via Gi1/0/3, not toward the access layer.

## Troubleshooting Encountered

- Attempted `standby version 2` under a physical interface (Gi1/0/1) instead of the VLAN SVI. Rejected with `% Invalid input detected`. Corrected by exiting to global config and entering `interface vlan 10` before configuring HSRP.
- While configuring the second VLAN's `preempt` command, remained in the first VLAN's interface context by mistake (e.g., typed `standby 20 preempt` while still inside `interface vlan 10`). This created a phantom, unconfigured HSRP group on the wrong interface and triggered unexpected `%HSRP-6-STATECHANGE` syslog messages. Resolved by resetting the lab and reconfiguring each VLAN interface one at a time, fully exiting before moving to the next.

## Key Observations

- HSRP configuration commands only apply under a Layer 3 VLAN interface (SVI), never under a physical switchport.
- HSRP group numbers are scoped per-interface, not global to the switch. The same group number can be reused across different VLAN interfaces without conflict, but the group number must match between the two switches participating in that same VLAN's HSRP pairing.
- The HSRP virtual IP must be a third, unused address in the subnet, distinct from either switch's own SVI address.
- `preempt` must be configured on both switches for both VLANs, not just on the switch that is active. Without it, a switch that comes back online after a failure will not reclaim the active role even if its priority is higher.
- STP root primary sets bridge priority to 24576 (plus the VLAN sys-id-ext), and root secondary sets it to 28672 (plus sys-id-ext), which is what aligns the STP root election with the HSRP active router per VLAN.

## Skills Demonstrated

- Configuring HSRP version 2 with custom priorities and preemption
- Synchronizing HSRP active/standby roles with STP root/secondary root roles per VLAN
- Diagnosing and correcting interface-mode errors between physical switchports and VLAN SVIs
- Verifying FHRP and STP state using `show standby brief`, `show standby`, and `show spanning-tree vlan X`

## CCNA Exam Alignment

- **2.5** Interpret basic operations of Rapid PVST+ Spanning Tree Protocol
- **3.5** Explain the purpose of first hop redundancy protocols (HSRP)

## Files

- `README.md` – this file
- `LAB 39- STP & HSRP Synchronization.pkt` – Packet Tracer topology and configuration file
