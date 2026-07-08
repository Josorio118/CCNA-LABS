# Lab 40 – GRE Tunnel with OSPF

## Objective

Configure a GRE tunnel between two routers connected over a simulated WAN/ISP underlay, then run OSPF over the tunnel to demonstrate why GRE is used instead of a plain IPsec VPN when a routing protocol needs to run between two sites (IPsec alone does not support the multicast traffic OSPF relies on for neighbor discovery).

## Topology

- 2x Cisco 2911 routers (R1, R2)
- R1 Gi0/0: 10.0.1.1/24 (LAN side)
- R2 Gi0/0: 10.0.2.1/24 (LAN side)
- R1 Gi0/0/0: 100.0.0.2/30 (WAN/underlay side, facing simulated ISP)
- R2 Gi0/0/0: 200.0.0.2/30 (WAN/underlay side, facing simulated ISP)
- GRE Tunnel0 between R1 and R2: 192.168.1.0/30 (R1 = .1, R2 = .2)
- Static default routes on each router pointing to their respective ISP-facing next hop

## Configuration Summary

**R1**

```
interface tunnel 0
 tunnel source GigabitEthernet0/0/0
 tunnel destination 200.0.0.2
 ip address 192.168.1.1 255.255.255.252

ip route 0.0.0.0 0.0.0.0 100.0.0.1

router ospf 1
 network 192.168.1.1 0.0.0.0 area 0
 network 10.0.1.1 0.0.0.0 area 0
 passive-interface GigabitEthernet0/0
```

**R2**

```
interface tunnel 0
 tunnel source GigabitEthernet0/0/0
 tunnel destination 100.0.0.2
 ip address 192.168.1.2 255.255.255.252

ip route 0.0.0.0 0.0.0.0 200.0.0.1

router ospf 1
 network 192.168.1.2 0.0.0.0 area 0
 network 10.0.2.1 0.0.0.0 area 0
 passive-interface GigabitEthernet0/0
```

Only the tunnel interface and LAN-side interface were brought into OSPF on each router. The WAN-facing underlay interfaces (Gi0/0/0) were intentionally left out of OSPF entirely, since the GRE tunnel itself is meant to carry the routing protocol traffic, not the raw underlay link.

## Verification

**Tunnel status**

```
show ip interface brief
```
Confirmed Tunnel0 up/up on both routers once correctly addressed, with the correct 192.168.1.1 and 192.168.1.2 assignments.

**Reachability**

```
ping 192.168.1.2   (from R1)
ping 192.168.1.1   (from R2)
```
Both directions reached 100 percent success once the tunnel was fully addressed on both ends.

**OSPF adjacency**

Syslog on both routers confirmed:
```
%OSPF-5-ADJCHG: Process 1, Nbr <neighbor-ip> on Tunnel0 from LOADING to FULL, Loading Done
```

**Routing table**

```
show ip route
```
- R1 learned 10.0.2.0/24 via OSPF, next hop 192.168.1.2, through Tunnel0
- R2 learned 10.0.1.0/24 via OSPF, next hop 192.168.1.1, through Tunnel0

Both LAN subnets became reachable through the GRE tunnel via OSPF, confirming the tunnel is correctly carrying routing protocol traffic end to end.

## Troubleshooting Encountered

- On R1, attempted to assign the tunnel interface IP with a /32 mask: `ip address 192.168.1.1 255.255.255.255`, which was rejected with `Bad mask /32 for address 192.168.1.1`. This command silently failed rather than applying partially, which left Tunnel0 unassigned and caused repeated ping failures (`Success rate is 0 percent`) until the mistake was identified. Corrected by re-entering the command with the proper /30 mask: `ip address 192.168.1.1 255.255.255.252`.
- Various command abbreviation typos on both routers (`int int tunnel 0`, `tunn dest 200/0/0/2`, `tun src g0/0/0`, `tunn src g0/0/0`, `pass-int g0/0`, `network 192.168.1.2 0.0.0.0 area0`) were rejected with `% Invalid input detected` and corrected to the proper syntax (`interface tunnel 0`, `tunnel destination 100.0.0.2`, `tunnel source GigabitEthernet0/0/0`, `passive-interface g0/0`, and adding the missing space before `area 0`).

## Key Observations

- A rejected configuration command (such as an invalid subnet mask) does not partially apply. It leaves the interface completely unconfigured until the corrected command is entered, which can cause confusing downstream symptoms (like 0 percent ping success) that look unrelated to the original typo.
- GRE tunnels require `tunnel source` and `tunnel destination` to reference the underlying physical interfaces or their IPs, while the tunnel's own IP address lives in a separate, unrelated subnet used only for the tunnel endpoints.
- IPsec alone does not support multicast, which is why OSPF (which relies on multicast hellos) cannot run over a plain IPsec VPN. GRE supports multicast and broadcast, which is why OSPF can run over a GRE tunnel.
- The underlay/WAN-facing interfaces should generally be excluded from the routing protocol entirely; only the tunnel interface and LAN-facing interfaces need to participate in OSPF.
- `passive-interface` is applied to LAN-facing interfaces where no OSPF neighbor is expected, preventing unnecessary hello traffic from being sent to end hosts.

## Skills Demonstrated

- Configuring a GRE tunnel between two routers over a simulated WAN underlay
- Correctly addressing tunnel interfaces in a separate subnet from the underlay
- Configuring OSPF to run over a GRE tunnel rather than the physical WAN interface
- Diagnosing a silently-failed interface configuration caused by an invalid subnet mask
- Verifying tunnel status, end-to-end reachability, OSPF adjacency, and routing table convergence

## CCNA Exam Alignment

- **1.2.d** Describe WAN topology options
- **3.5** (related) First hop and tunneling concepts as part of WAN architecture
- **5.5** Explain the security concepts relevant to site-to-site connectivity (GRE and IPsec context)

## Files

- `README.md` – this file
- GRE tunnel and OSPF lab Packet Tracer file
