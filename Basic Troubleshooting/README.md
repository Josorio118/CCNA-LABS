# Troubleshooting Fundamentals

## Series Objective

A nine-part progressive troubleshooting series covering Layer 1 through Layer 3 fault isolation on Cisco IOS switches and routers. Each phase presents a "Host X cannot reach Host Y" scenario in Cisco Packet Tracer with no prior indication of the root cause. The series builds from single-switch physical and VLAN faults through multi-switch trunking issues to multi-router Layer 3 routing faults.

## Methodology

Every phase follows the same structured approach:

1. **Verify the problem** with an initial ping to confirm the reported symptom.
2. **Decide where to start** by asking what must be true for communication to work, and by checking whether other hosts on the network have the same problem. This isolates whether the fault is localized or widespread before touching any configuration.
3. **Investigate bottom-up** through the OSI layers: physical connectivity and port status (Layer 1), VLAN and trunk configuration (Layer 2), then IP addressing and routing (Layer 3).
4. **Correct and validate** the fix with a re-test ping.

This bottom-up, evidence-first approach, rather than guessing at a fix, is the core skill the series is meant to build.

---

## Basic_Troubleshooting_1 — Disabled Switchport

## Objective

Host A is unable to reach Host B. Investigate and resolve the issue.

## Topology

Host A (192.168.1.2) and Host B (192.168.1.6) connected through a single switch, SW1.

## Configuration Summary

```
interface fa0/2
 no shutdown
```

## Verification

| Check | Command | Result |
|---|---|---|
| Initial connectivity | `ping 192.168.1.6` from Host A | Fails, 100% loss |
| Switchport status | `show interface status` on SW1 | Fa0/2 shown as disabled |
| Post-fix connectivity | `ping 192.168.1.6` from Host A | Succeeds |

## Troubleshooting Encountered

- Initial ping from Host A to Host B failed with 100% loss.
- Both hosts were confirmed powered on with the correct straight-through cabling to SW1, ruling out a cabling issue.
- `show interface status` on SW1 revealed Fa0/2 (Host B's port) in a disabled state, meaning the port had been administratively shut down.
- Issued `no shutdown` on Fa0/2 to bring the port back up.
- Re-tested with a ping from Host A to Host B; connectivity was restored.

## Key Observations

- A disabled or err-disabled switchport is a Layer 1 fault and should be one of the first things checked when a host cannot reach anything.
- `notconnect` and `disabled` are different statuses. `notconnect` can indicate a Layer 1 cabling or powered-off device issue, while `disabled` specifically means the port has been shut down.

---

## Basic_Troubleshooting_2 — VLAN Mismatch (Single Switch)

## Objective

Host C is unable to communicate with Host A. Investigate and resolve the issue.

## Topology

Host A (192.168.1.2), Host B (192.168.1.3), and Host C (192.168.1.4), all connected to a single switch, SW1.

## VLAN Design

All hosts are intended to share the default VLAN 1. Host A's port was found misassigned to VLAN 2.

## Configuration Summary

```
interface fa0/1
 no switchport access vlan 2
```

## Verification

| Check | Command | Result |
|---|---|---|
| Initial connectivity | `ping 192.168.1.2` from Host C | Fails, 100% loss |
| Control test | `ping 192.168.1.3` from Host C | Succeeds |
| VLAN assignment | `show interface status` on SW1 | Fa0/1 (Host A) shown in VLAN 2 |
| Post-fix connectivity | `ping 192.168.1.2` from Host C | Succeeds |

## Troubleshooting Encountered

- Initial ping from Host C to Host A failed.
- Before checking hardware, tested whether Host B could reach either Host A or Host C to determine whether the fault was widespread or isolated. Host B successfully reached Host C, which narrowed the problem down to Host A or its switchport.
- Verified Host A's physical connectivity and switchport status (up, connected) at Layer 1, ruling out a cabling or shutdown issue.
- `show interface status` on SW1 showed Fa0/1 (Host A) assigned to VLAN 2, while Hosts B and C remained on the default VLAN 1. This placed Host A in a separate broadcast domain.
- Removed the VLAN 2 assignment from Fa0/1 to return it to the default VLAN.
- Re-tested; ping succeeded.

## Key Observations

- Testing an unaffected host's connectivity to both sides first is a useful diagnostic step. It confirms whether the switch as a whole is functioning and narrows the fault to a specific host or port before further investigation.
- A confirmed "connected" switchport status does not rule out a Layer 2 issue. VLAN assignment must be checked separately from physical link status.
- Assigning the ports connected to Hosts B and C into VLAN 2 instead would also have resolved connectivity; removing Host A's VLAN assignment was chosen since it returns the topology to its intended default-VLAN design.

---

## Basic_Troubleshooting_3 — Trunk/Access Mode Mismatch (Inter-Switch Link)

## Objective

Host C is unable to communicate with Hosts A and B. Investigate and resolve the issue.

## Topology

Host A and Host B connected to SW1. Host C connected to SW2. SW1 and SW2 joined by an inter-switch link.

## VLAN Design

All host-facing ports assigned to VLAN 10. The inter-switch link is intended to operate as a trunk carrying VLAN 10 between SW1 and SW2.

## Configuration Summary

```
interface fa0/1
 switchport mode trunk
```
(applied on SW2's port facing SW1)

## Verification

| Check | Command | Result |
|---|---|---|
| Initial connectivity | `ping` from Host C to Hosts A and B | Fails, 100% loss |
| Control test | `ping` between Host A and Host B | Succeeds |
| Trunk/access status | `show interface status` on SW1 and SW2 | SW1's link to SW2 shown as trunk; SW2's link to SW1 shown as access, VLAN 1 |
| Post-fix status | `show interface trunk` on SW2 | Fa0/1 confirmed trunking |
| Post-fix connectivity | `ping` from Host C to Hosts A and B | Succeeds |

## Troubleshooting Encountered

- Initial ping from Host C to Hosts A and B failed.
- Confirmed Hosts A and B could reach each other, which isolated the fault to Host C's switch or the inter-switch link rather than a widespread issue.
- Verified Host C's own port status and VLAN assignment were correct, ruling out a Layer 1/2 issue local to Host C.
- Checked switchport status on both SW1 and SW2. SW1's port toward SW2 was configured as a trunk, but SW2's corresponding port toward SW1 was left as an access port in the default VLAN 1. This port mode mismatch prevented VLAN 10 traffic from crossing the link correctly.
- Reconfigured SW2's port as a trunk to match SW1's side.
- Re-tested connectivity immediately after the change; the ping still failed on the first attempt because Spanning Tree Protocol had not yet transitioned the newly formed trunk into the forwarding state.
- Waited for STP convergence (roughly 50 seconds) and confirmed with `show interface trunk` that VLAN 10 had reached the forwarding state before re-testing.
- Re-tested connectivity; ping succeeded.

## Key Observations

- Both sides of an inter-switch link must agree on trunk versus access mode. A one-sided trunk configuration will not forward tagged VLAN traffic correctly.
- A newly formed trunk port does not forward traffic immediately. STP must transition the port through its states first, which can cause a fix to appear unsuccessful if tested too soon.
- `show interface trunk` reporting a VLAN as allowed does not guarantee it is actively forwarding. The VLAN must also appear in the spanning tree forwarding state line of the same output.

---

## Basic_Troubleshooting_4 — Trunk Allowed-VLAN List

## Objective

Host C is unable to communicate with Host B. Investigate and resolve the issue.

## Topology

Host A and Host B connected to SW1. Host C connected to SW2. SW1 and SW2 joined by an inter-switch trunk link.

## VLAN Design

All host-facing ports assigned to VLAN 10. The inter-switch trunk is configured on both sides but its allowed-VLAN list must include VLAN 10 for host traffic to cross.

## Configuration Summary

```
interface fa0/4
 switchport trunk allowed vlan add 10
```

## Verification

| Check | Command | Result |
|---|---|---|
| Initial connectivity | `ping 192.168.1.3` from Host C | Fails, 100% loss |
| Control test | `ping` between Host A and Host B | Succeeds |
| Trunk mode | `show interface status` on SW1 and SW2 | Both sides confirmed trunking |
| Allowed VLAN list | `show interface trunk` on SW1 | VLAN 10 absent from allowed list |
| Post-fix connectivity | `ping 192.168.1.3` from Host C | Succeeds |

## Troubleshooting Encountered

- Initial ping from Host C to Host B failed.
- Confirmed Hosts A and B could reach each other, isolating the fault to Host C's switch or the inter-switch link.
- Confirmed both inter-switch ports were correctly configured as trunks, ruling out the mode mismatch seen in Basic_Troubleshooting_3.
- `show interface trunk` on SW1 showed the trunk's allowed-VLAN list excluded VLAN 10, meaning tagged frames for that VLAN were filtered at the trunk even though the link itself was healthy.
- Added VLAN 10 to SW1's allowed-VLAN list on the trunk interface.
- Re-tested connectivity; ping succeeded.

## Key Observations

- A trunk being up and correctly negotiated does not guarantee all required VLANs are permitted across it. The allowed-VLAN list must be checked independently of trunk status.
- This fault produces the same symptom as a trunk mode mismatch (Basic_Troubleshooting_3), reinforcing the need to check both trunk negotiation and the allowed-VLAN list rather than stopping at the first plausible cause.

---

## Basic_Troubleshooting_5 — Missing Host IP Configuration (Multi-Router)

## Objective

Host D is unable to communicate with Host B. Investigate and resolve the issue.

## Topology

Host A and Host B connected to SW1. Host C and Host D connected to SW2. Both switches connect to RTR1, which routes between the two subnets (192.168.1.0/29 and 192.168.2.0/29).

## Configuration Summary

Host B's Desktop IP configuration was set as follows:

```
IP Address:      192.168.1.3
Subnet Mask:      255.255.255.248
Default Gateway:  192.168.1.1
```

## Verification

| Check | Command | Result |
|---|---|---|
| Initial connectivity | `ping` from Host D to Host B | Fails, 100% loss |
| Host D IP config | `ipconfig` on Host D | Valid IP, mask, and gateway; gateway reachable |
| Host B IP config | `ipconfig` on Host B | 0.0.0.0 across IP, mask, and gateway |
| Post-fix connectivity | `ping` from Host D to Host B | Succeeds |

## Troubleshooting Encountered

- Initial ping from Host D to Host B failed.
- Since the two hosts sit on different subnets connected through a router, Layer 3 reachability was tested directly rather than starting at Layer 1 or 2, since successful Layer 3 reachability also confirms the lower layers are functioning correctly along the path.
- Confirmed Host D had a valid IP address and could successfully reach its own default gateway.
- Checked Host B's IP configuration and found it had no IP address, subnet mask, or default gateway assigned at all.
- Statically assigned Host B a valid IP address, subnet mask, and default gateway matching its subnet.
- Re-tested connectivity from Host B to Host D; ping succeeded.

## Key Observations

- When two hosts are on different subnets separated by a router, testing Layer 3 reachability first is more efficient than starting at Layer 1, since a successful result at Layer 3 confirms the lower layers are already functioning.
- A completely unconfigured host (no IP, mask, or gateway) is a common and easily overlooked root cause, particularly when only one side of a conversation is affected.

---

## Basic_Troubleshooting_6 — Router Interface Mask Mismatch

## Objective

Host E is unable to communicate with Hosts A and B. Investigate and resolve the issue.

## Topology

Host A and Host B connected to SW1. Hosts C, D, and E connected to SW2. Both switches connect to RTR1, which routes between the two subnets.

## Configuration Summary

```
interface GigabitEthernet0/0/1
 ip address 192.168.2.1 255.255.255.248
```

## Verification

| Check | Command | Result |
|---|---|---|
| Initial connectivity | `ping` from Host E to Hosts A and B | Fails, 100% loss |
| Host E IP config | `ipconfig` on Host E | Valid IP and default gateway configured |
| Gateway reachability | `ping` from Host E to its gateway | Fails |
| RTR1 interface status | `show ip interface brief` on RTR1 | Interface up/up, IP appeared to match Host E's gateway |
| RTR1 interface config | `show run` for the interface facing SW2 | Mask configured as /30 (255.255.255.252) |
| Post-fix connectivity | `ping` from Host E to Hosts A and B | Succeeds |

## Troubleshooting Encountered

- Initial ping from Host E to Hosts A and B failed.
- Confirmed Host E had a valid IP address and default gateway configured, but could not reach that gateway.
- An initial `show ip interface brief` on RTR1 showed the relevant interface as up/up with an IP address that appeared to match Host E's configured gateway, creating a false impression that the router side was correctly configured.
- Checking `show run` for that same interface revealed the actual configured mask was a /30 (usable range of only two addresses), while Host E's own IP configuration placed it on a /29 subnet. Host E's address fell entirely outside the router interface's configured range, meaning Host E could not reach its own gateway despite both appearing correctly addressed at a glance.
- Corrected RTR1's interface mask to 255.255.255.248 to match the actual /29 subnet in use.
- Re-tested connectivity; ping succeeded.

## Key Observations

- A quick check like `show ip interface brief` can create a false sense that a device is correctly configured. The subnet mask is not shown in that output and must be verified separately with `show run` or `show ip route`.
- The order in which reachability is tested can shape the direction of an investigation. Testing gateway reachability from Hosts C and D first (rather than Host E) would have shown success, since their addressing did match the router's configured range, and likely would have misdirected the investigation toward Host E's own Layer 1/2 configuration instead of the router.

---

## Basic_Troubleshooting_7 — Missing Static Routes (Both Directions)

## Objective

Host B is unable to communicate with Host D. Investigate and resolve the issue.

## Topology

SW1 and RTR1 (Host A and Host B, 192.168.1.0/24) connected via a WAN link (10.1.1.0/30) to RTR2 and SW2 (Host C and Host D, 192.168.2.0/24).

## Configuration Summary

```
! On RTR1
ip route 192.168.2.0 255.255.255.0 10.1.1.2
```
The mirrored route on RTR2 (pointing to RTR1's WAN interface, 10.1.1.1, for the 192.168.1.0/24 network) was configured following the same logic to restore connectivity in both directions.

## Verification

| Check | Command | Result |
|---|---|---|
| Initial connectivity | `ping` from Host B to Host D | Fails, 100% loss |
| Host gateway reachability | `ping` from each host to its own gateway | Succeeds |
| RTR1 routing table | `show ip route` on RTR1 | No route to 192.168.2.0/24, only local and connected entries |
| RTR2 routing table | `show ip route` on RTR2 | No route to 192.168.1.0/24, only local and connected entries |
| Post-fix connectivity | `ping` from Host B to Host D | Succeeds |

## Troubleshooting Encountered

- Initial ping from Host B to Host D failed.
- Confirmed both hosts could reach their own default gateways, ruling out a local Layer 1/2 or host addressing issue.
- Checked `show ip route` on both RTR1 and RTR2 and found neither router had any route to the other side's LAN. Only connected and local entries were present, and no dynamic routing protocol was running.
- Concluded static routes needed to be configured on both routers.
- Writing the actual commands required a couple of corrections along the way: an initial attempt used `ip add` instead of the correct `ip route` command, and the next-hop was first given as the network address itself (10.1.1.0) rather than a specific usable host address on the shared link.
- Identified the correct next-hop addresses as each router's own WAN interface IP as seen from the other router's perspective (RTR2's 10.1.1.2 from RTR1, and RTR1's 10.1.1.1 from RTR2).
- Configured the static routes on both routers.
- Re-tested connectivity in both directions; pings succeeded.

## Key Observations

- With no dynamic routing protocol running, an empty routing table entry for a known remote network is diagnostic on its own. It confirms the fault is a missing route rather than a reachability or Layer 2 problem.
- A next-hop must be a specific, directly reachable router interface address. Supplying a network address instead of a host address for the next-hop is an invalid route and will not work as expected.

---

## Basic_Troubleshooting_8 — Incorrect Static Route Next-Hop

## Objective

Host A is unable to communicate with Host D. Investigate and resolve the issue.

## Topology

SW1 and RTR1 (Host A and Host B, 192.168.1.0/24) connected via a WAN link (10.1.1.0/30) to RTR2 and SW2 (Host C and Host D, 192.168.2.0/24).

## Configuration Summary

```
no ip route 192.168.2.0 255.255.255.0 10.1.1.4
ip route 192.168.2.0 255.255.255.0 10.1.1.2
```

## Verification

| Check | Command | Result |
|---|---|---|
| Initial connectivity | `ping` from Host A to Host D | Fails, 100% loss |
| Host A gateway reachability | `ping` from Host A to RTR1 | Succeeds |
| RTR1 to Host D's gateway | `ping` from RTR1 to RTR2 | Fails |
| Path trace | `traceroute` from RTR1 to RTR2 | Fails at the first hop |
| RTR1 routing table | `show ip route` on RTR1 | Route to 192.168.2.0/24 present |
| Next-hop reachability | `ping` from RTR1 to configured next-hop 10.1.1.4 | Fails |
| RTR1 route configuration | `show run \| sec ip route` | Static route next-hop set to 10.1.1.4 |
| Post-fix connectivity | `ping` from Host A to Host D | Succeeds |

## Troubleshooting Encountered

- Initial ping from Host A to Host D failed.
- Confirmed Host A could reach its own gateway (RTR1), ruling out a local Layer 1/2 issue.
- Logged into RTR1 and tested reachability to RTR2's LAN gateway directly, which also failed, suggesting the issue affected the entire destination network rather than just Host D.
- Ran a traceroute to the destination from RTR1. The traceroute failed at the very first hop, which was a meaningful clue: if RTR1 had a working route toward RTR2, RTR2 (a directly connected neighbor) should have responded with a TTL-expired message at hop 1.
- Checked `show ip route` on RTR1 and found a route to 192.168.2.0/24 already present in the table, which initially seemed to rule out a routing table issue.
- Pinged the configured next-hop directly and it failed, revealing that the route's next-hop address (10.1.1.4) did not correspond to any real device on the shared WAN link.
- Checked the running configuration and confirmed the static route pointed to 10.1.1.4, while RTR2's actual WAN interface address was 10.1.1.2.
- Removed the incorrect static route and added a corrected one pointing to the real next-hop.
- Re-tested with a traceroute, which now completed in a single hop directly to RTR2, and confirmed with a successful ping from Host A to Host D.

## Key Observations

- A route being present in the routing table does not guarantee it is correct. The next-hop address itself must also be validated as a real, reachable device.
- Traceroute failing at the very first hop, when the next hop should be a directly connected neighbor, points specifically to either a missing route or an incorrect next-hop rather than a problem further down the path.
- This type of misconfiguration (a route that exists but points to the wrong next-hop) is not expected to occur often on a stable network with few changes, but is a useful pattern to recognize for troubleshooting and exam scenarios.

---

## Basic_Troubleshooting_9 — Static Route Mask Too Narrow

## Objective

Host G is unable to communicate with Host D. Investigate and resolve the issue.

## Topology

SW1 and RTR1 (Hosts A through D, 192.168.1.0/24) connected via a WAN link (10.1.1.0/30) to RTR2 and SW2 (Hosts E through H, 192.168.2.0/24).

## Configuration Summary

```
! Corrected static route on RTR1
ip route 192.168.2.0 255.255.255.0 10.1.1.2
```

## Verification

| Check | Command | Result |
|---|---|---|
| Initial connectivity | `ping` from Host G to Host D | Fails, 100% loss |
| Host G to Host D's subnet gateway (RTR1) | `ping` from Host G to 192.168.1.1 | Fails |
| Host G to its own gateway (RTR2) | `ping` from Host G to 192.168.2.1 | Succeeds, 0% loss |
| RTR1 running config, WAN interface | `show run` for RTR1's WAN interface | Actual mask was /29, not /30 as diagrammed |
| RTR1 static route | `show run` on RTR1 | Route to 192.168.2.0 configured with a /30 mask (255.255.255.252) |
| Post-fix connectivity | `ping` from Host G to Host D | Succeeds |

## Troubleshooting Encountered

- Initial ping from Host G to Host D failed.
- Confirmed Host G could reach its own gateway (RTR2) with 0% loss, but could not reach either Host D or RTR1 directly, isolating the fault to RTR1's route back to Host G's subnet rather than a problem on Host G's own segment.
- An early read of RTR1's WAN interface configuration noted the interface mask appeared to be a /30 rather than the /30 shown in the topology diagram; on closer inspection the actual configured mask was a /29, and the diagram label was not authoritative. Working through the valid host range for that mask confirmed it did not actually explain Host G's inability to be reached, since the true fault was elsewhere.
- The actual root cause was found in RTR1's static route to the 192.168.2.0 network, which was configured with a /30 mask (255.255.255.252) instead of the correct /24. A /30 mask only covers four addresses (192.168.2.0 through 192.168.2.3), so the route did not match Host G's actual address (192.168.2.4) or most of the real subnet.
- Corrected the mask on RTR1's static route from 255.255.255.252 to 255.255.255.0.
- Re-tested connectivity; ping succeeded.

## Key Observations

- A diagram's labeled subnet mask is not a substitute for checking the actual running configuration. In this lab the interface mask shown in the diagram did not match what was actually configured on the device.
- A static route's mask defines exactly which addresses that route matches. A mask that is too narrow (a /30 applied to what is actually a /24 network) will silently fail to match most of the real hosts on that subnet, even though the route appears present and pointed at the correct network address.
- Testing a host's reachability to its own gateway first, before testing reachability to the remote gateway or remote host, is an efficient way to immediately rule out a local Layer 1/2 fault and focus the investigation on the routing path.

---

## Skills Demonstrated Across the Series

- Bottom-up OSI-layer troubleshooting methodology (Layer 1 through Layer 3)
- Isolating fault scope by testing control paths against unaffected hosts before touching device configuration
- Interface and switchport status verification (`show interface status`, `show ip interface brief`)
- VLAN and trunk troubleshooting (`show interface trunk`, allowed-VLAN lists, trunk/access mode mismatches)
- Recognizing Spanning Tree Protocol convergence delay as distinct from a persistent misconfiguration
- Static route troubleshooting, including next-hop validation and subnet mask scope errors
- Reading `show run` output critically rather than trusting a surface-level `show ip interface brief` check
- Correct `ip route` syntax, including selecting a valid next-hop host address rather than a network address

## CCNA Exam Alignment

| Exam Topic | Description |
|---|---|
| 1.0 | Network Fundamentals — cabling, interface status, IP addressing and subnetting |
| 2.0 | Network Access — VLANs, trunking, Spanning Tree Protocol |
| 3.0 | IP Connectivity — static routing configuration and troubleshooting |

## Files

- Individual Packet Tracer lab files (`Basic_Troubleshooting_1.pka` through `Basic_Troubleshooting_9.pka`)
