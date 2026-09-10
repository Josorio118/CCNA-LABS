# HSRP — First Hop Redundancy Protocol

### Objective
Build a self-designed two-router HSRP topology to reinforce first-hop redundancy concepts: virtual IP addressing, priority-based active/standby election, and preemption. Verify normal operation, simulate a router failure to confirm failover, then confirm the active router reclaims its role on recovery.

### Topology
```
                            R1 - Active
                           Gi0/0: .2, priority 150
                          /
PC1 ---- SW1 (VLAN 10) --
192.168.10.10             \
                            R2 - Standby
                           Gi0/0: .3, priority 100 (default)

                    Virtual IP: 192.168.10.1
```

### IP Addressing
| Device | Interface | IP Address | Role |
|---|---|---|---|
| R1 | Gi0/0 | 192.168.10.2/24 | HSRP Active (priority 150) |
| R2 | Gi0/0 | 192.168.10.3/24 | HSRP Standby (priority 100, default) |
| — | HSRP Group 1 | 192.168.10.1/24 | Virtual IP (shared gateway) |
| PC1 | — | 192.168.10.10/24 | Gateway set to virtual IP |

### Configuration Summary
```
! R1
interface GigabitEthernet0/0
 ip address 192.168.10.2 255.255.255.0
 no shutdown
 standby version 2
 standby 1 ip 192.168.10.1
 standby 1 priority 150
 standby 1 preempt
```

```
! R2
interface GigabitEthernet0/0
 ip address 192.168.10.3 255.255.255.0
 no shutdown
 standby version 2
 standby 1 ip 192.168.10.1
 standby 1 preempt
```

### Verification
```
R1#sh standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Gig0/0      1    150 P Active   local           192.168.10.3    192.168.10.1

R2#sh standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Gig0/0      1    100 P Standby  192.168.10.2    local           192.168.10.1
```

| Test | Result |
|---|---|
| PC1 → virtual IP (192.168.10.1), normal operation | Success, routed via R1 |
| R1 Gi0/0 shutdown (simulated failure) | R2 transitions Standby → Active (`%HSRP-6-STATECHANGE`) |
| PC1 → virtual IP during R1 outage | Success, routed via R2 |
| R1 Gi0/0 restored | R1 transitions Init → Listen → Speak → Standby → Active, reclaims role via preemption |

### Troubleshooting Encountered
While configuring HSRP, a duplicate-address conflict appeared on the virtual IP (`%IP-4-DUPADDR: Duplicate address 192.168.10.1`), with each router's log identifying a different source MAC — R1 showed `0000.0C07.AC01` and R2 showed `0000.0C9F.F001`. These are the well-known HSRP version 1 and version 2 virtual MAC address formats respectively, which indicated the two routers were running mismatched HSRP versions rather than pairing up as an active/standby group. Each router was independently claiming the virtual IP as its own since neither recognized the other as an HSRP peer.

The root cause traced back to the console: syslog messages were interrupting command entry before `logging synchronous` was applied to the console line, which had caused the `standby version 2` command on R2 to not fully register. Re-applying `logging synchronous` first, then re-issuing `standby version 2` on R2 to match R1, resolved the conflict immediately and allowed the group to form correctly.

### Key Observations
- The source MAC address reported in a duplicate-address log is diagnostic on its own: `0000.0C07.ACxx` identifies HSRPv1, `0000.0C9F.Fxxx` identifies HSRPv2 — a mismatch between the two on either side of a supposed HSRP pair means the routers were never actually negotiating together.
- Unsynchronized console logging isn't just a readability annoyance — it can cause a command to be dropped or only partially entered mid-type, leading to a config that looks intentional in the running-config but was never actually what was meant to be typed. `logging synchronous` should be configured early, before any extended troubleshooting session.
- HSRP's full state progression (Init → Listen → Speak → Standby → Active) is directly observable via `%HSRP-6-STATECHANGE` messages, which is useful for confirming preemption is actually working step by step rather than just checking the end state.

### Skills Demonstrated
- HSRP virtual IP, priority, and preemption configuration
- Diagnosing an HSRP version mismatch from duplicate-address log output and virtual MAC address patterns
- Correcting a console logging issue that caused a silently incomplete configuration change
- Verifying failover and preemption behavior through live interface shutdown/recovery rather than static config review

### CCNA Exam Alignment
Per the CCNA 200-301 v1.1 blueprint, IP Connectivity (25% weight) objective 3.5 covers the purpose, functions, and concepts of first-hop redundancy protocols. This lab reinforces that objective directly:
- HSRP virtual IP and active/standby election via priority
- Preemption behavior and full HSRP state transitions
- Distinguishing HSRP (Cisco-proprietary) from VRRP (IETF standard) via version-specific virtual MAC address behavior

### Files
- `HSRP_Lab.pkt`
