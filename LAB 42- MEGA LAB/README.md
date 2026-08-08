# CCNA Mega Lab (Jeremy's IT Lab)

## Objective

Design, build, and fully configure a 12+ device enterprise network in Cisco Packet Tracer, covering the complete CCNA 200-301 syllabus in a single integrated topology. The build progresses through Layer 2 foundation, Layer 3 routing, network services, security hardening, NAT, IPv6, and wireless, with every stage independently verified against live `show` command output rather than assumed correct after configuration. This was completed across multiple sessions and represents the most comprehensive lab in this repository.

## Topology

12 core network devices plus wireless infrastructure and endpoints:

- **R1** - edge router, dual ISP uplinks (ISPA, ISPB), NAT/PAT boundary, DHCP server, ASBR for OSPF
- **CSW1, CSW2** - core switches, Layer 3 EtherChannel between them (PAgP)
- **DSW-A1, DSW-A2** - Office A distribution switches, HSRP active/standby split
- **DSW-B1, DSW-B2** - Office B distribution switches, HSRP active/standby split
- **ASW-A1, ASW-A2, ASW-A3** - Office A access switches
- **ASW-B1, ASW-B2, ASW-B3** - Office B access switches
- **WLC1, LWAP1, LWAP2** - wireless controller and lightweight access points
- **SRV1** - DNS, DHCP relay target, static NAT'd server (Office B)

**Office A VLANs:** 10 (PCs), 20 (Phones), 40 (Wi-Fi), 99 (Management)
**Office B VLANs:** 10 (PCs), 20 (Phones), 30 (Servers), 99 (Management)

## Configuration Summary

### 1. Initial Setup, Layer 2 EtherChannel, Trunking, VTP, Access Ports

- Hostnames, enable secret (type 9/5), local user accounts, console line configuration on all 12 devices
- L2 EtherChannel: DSW-A1<->A2 (PAgP desirable), DSW-B1<->B2 (LACP active), both as Port-channel 1
- Trunking on all interswitch links, native VLAN 1000, DTP disabled via `switchport nonegotiate`
- VTP v2, domain `JeremysITLab`: DSW-A1 and DSW-B1 as VTP servers, all other switches as clients
- Access ports: LWAPs to Vlan99, Phone+PC ports configured with both `switchport access vlan 10` and `switchport voice vlan 20`, SRV1 to Vlan30
- WLC1 trunk on ASW-A1 F0/2, native and allowed VLANs 40 and 99
- All unused switch ports administratively shut down

```
interface range g1/0/x
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 10,20,30,99
 switchport mode trunk
 switchport nonegotiate
```

### 2. IP Addressing, Layer 3 EtherChannel, HSRP

- R1 addressed on all five interfaces (two ISP-facing DHCP clients, two internal, one loopback)
- CSW1<->CSW2 Layer 3 EtherChannel, requiring `no switchport` before IP assignment on the member ports
- All DSW loopback and distribution uplink interfaces addressed
- SRV1 assigned a static IP via GUI
- Access switch management addressed via Vlan99, using the global `ip default-gateway` command appropriate for Layer 2-only devices
- HSRPv2 deployed in four groups per office, with priority 105 and preempt configured on the active router in each group

```
interface vlan10
 standby version 2
 standby 2 ip 10.1.0.1
 standby 2 priority 105
 standby 2 preempt
```

### 3. Spanning Tree: Rapid PVST+, Root Priority Alignment, PortFast/BPDU Guard

- `spanning-tree mode rapid-pvst` enabled on every switch
- Root bridge priority deliberately aligned with HSRP roles: priority 0 assigned to the distribution switch active for a given VLAN, priority 4096 on its standby counterpart, keeping the STP root and the FHRP gateway co-located
- PortFast and BPDU Guard enabled on all end-host access ports
- WLC1's trunk port configured with `spanning-tree portfast trunk`, the correct variant for a trunk-mode PortFast port

```
spanning-tree portfast
spanning-tree bpduguard enable
```

### 4. OSPF and Static Routing

- OSPF process 1, single area (0), router ID set explicitly to each device's Loopback0 address
- R1 configured with point-to-point network type on ISP-facing and internal links, Loopback0 advertised and passive
- Core and distribution switches use per-interface `network` statements scoped to their own IPs, all non-management SVIs passive, and point-to-point network type on physical interswitch links (the CSW1<->CSW2 Port-channel remains default broadcast/DR-BDR)
- Dual default static routes on R1: a primary via ISPA at administrative distance 1, and a floating backup via ISPB at administrative distance 2
- `default-information originate` configured on R1 as the network's ASBR, propagating the default route into OSPF

```
router ospf 1
 router-id 10.0.0.76
 passive-interface Loopback0
 default-information originate
!
ip route 0.0.0.0 0.0.0.0 203.0.113.1
ip route 0.0.0.0 0.0.0.0 203.0.113.5 2
```

### 5. Network Services

- Seven DHCP pools on R1 covering both offices' PC, phone, and management subnets, plus Wi-Fi
- DHCP relay (`ip helper-address 10.0.0.76`) configured on every active VLAN, wired and management alike, across all four distribution switches
- SRV1 configured as the network's DNS server with A and CNAME records
- Domain name and name server configured on every device
- NTP: R1 as authenticated master, all switches as authenticated clients (MD5, key 1)
- SNMP read-only community string on every router and switch
- Centralized syslog to SRV1 at all severity levels, with an 8192-byte local buffer
- IOS upgrade on R1 performed via FTP, including verification and cleanup of the prior image
- SSH hardened with 4096-bit RSA keys, SSHv2 only, VTY access restricted by ACL to Office A's PC subnet, local authentication, and synchronous logging
- Static NAT for SRV1 and pool-based dynamic PAT for internal client subnets, both verified with live translations
- CDP disabled and LLDP enabled globally, with LLDP transmission disabled on each access switch's end-host port

```
ip dhcp pool A-PC
 network 10.1.0.0 255.255.255.0
 default-router 10.1.0.1
 dns-server 10.5.0.4
 domain-name jeremysitlab.com
!
interface vlan10
 ip helper-address 10.0.0.76
!
crypto key generate rsa modulus 4096
ip ssh version 2
!
ip nat pool POOL1 203.0.113.200 203.0.113.207 netmask 255.255.255.248
ip nat inside source list 2 pool POOL1 overload
ip nat inside source static 10.5.0.4 203.0.113.113
```

### 6. Layer 2 Security: DHCP Snooping and Dynamic ARP Inspection

- DHCP snooping enabled on every active VLAN per office, including the management VLAN
- Distribution-facing uplinks trusted; untrusted end-host ports rate-limited to 15 pps
- WLC1's uplink rate-limited to 100 pps and trusted, reflecting its higher legitimate traffic volume
- DHCP Option 82 insertion disabled network-wide
- Dynamic ARP Inspection enabled on the same VLAN scope, with all three optional validation checks (source MAC, destination MAC, IP address) active

```
ip dhcp snooping vlan 10,20,40,99
no ip dhcp snooping information option
interface range g0/1-2
 ip dhcp snooping trust
interface f0/1
 ip dhcp snooping limit rate 15
!
ip arp inspection vlan 10,20,40,99
ip arp inspection validate src-mac dst-mac ip
```

### 7. Extended ACLs and Port Security

- Named extended ACL restricting Office A's PC subnet to ICMP-only access toward Office B's PC subnet, all other inter-subnet traffic denied, all remaining traffic explicitly permitted
- Applied inbound on DSW-A1's Vlan10 SVI, closest to the traffic source per extended ACL best practice
- Port security enabled on every access switch's end-host port: maximum of one MAC address on single-device ports (access points, SRV1), maximum of two on phone ports to account for a downstream PC
- Violation mode set to `restrict`, blocking invalid traffic while leaving legitimate traffic unaffected and generating a notification
- Sticky MAC learning enabled so dynamically learned addresses persist in the running configuration

```
ip access-list extended OfficeA_to_OfficeB
 permit icmp 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
 deny ip 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
 permit ip any any
!
interface vlan10
 ip access-group OfficeA_to_OfficeB in
!
interface f0/1
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
```

### 8. IPv6

- IPv6 unicast routing enabled on R1, CSW1, and CSW2
- R1's ISP-facing links statically addressed; R1<->CSW1 and R1<->CSW2 links use prefix-based addressing with EUI-64 interface ID generation
- CSW1<->CSW2 Port-channel enabled for IPv6 without an explicit address, using `ipv6 enable` for link-local-only operation
- Two default static routes configured on R1: a recursive route via ISPA's next-hop address, and a fully specified floating route (administrative distance 2) via ISPB's exit interface and next-hop address, providing IPv6 Internet failover

```
ipv6 unicast-routing
!
interface g0/0
 ipv6 address 2001:db8:a1::/64 eui-64
!
interface po1
 ipv6 enable
!
ipv6 route ::/0 2001:db8:a::1
ipv6 route ::/0 GigabitEthernet0/1/0 2001:db8:b::1 2
```

### 9. Wireless

- Dynamic interface created on WLC1 for the Wi-Fi VLAN (VLAN 40, 10.6.0.4/24, gateway 10.6.0.1, DHCP server 10.0.0.76)
- WLAN profile "Wi-Fi" created and bound to the dynamic interface, secured with WPA2-PSK and AES encryption
- WLAN enabled and both LWAPs confirmed associated with WLC1

## Verification

| Feature | Command | Result |
|---|---|---|
| EtherChannel | `show etherchannel summary` | Port-channel up, correct protocol per link |
| Trunking | `show interfaces trunk` | Native VLAN 1000, correct allowed VLAN list |
| HSRP | `show standby brief` | Correct active/standby split per office and group |
| OSPF adjacencies | `show ip ospf neighbor` | FULL state on all expected neighbors |
| Default route propagation | `show ip route` | `O*E2 0.0.0.0/0` present on all downstream devices |
| DHCP relay | `show run \| include helper-address` | Present on all active VLANs, all four DSWs |
| NAT / PAT | `show ip nat translations` | Static and dynamic entries populate on live traffic |
| SSH | `show ip ssh` | Version 2.0, 4096-bit keys generated |
| Port security | `show port-security interface` | Correct maximum and violation mode per port |
| DHCP snooping | `show ip dhcp snooping` | Correct VLAN scope, trust, and rate limits per switch |
| DAI | `show ip arp inspection` | All three validation checks enabled, correct VLANs active |
| IPv6 | `show ipv6 route`, `show run \| include ipv6 route` | Correct connected, local, and static entries |
| Wireless | WLC1 GUI, Wireless > Access Points | Both LWAPs associated |

Full topology completed and verified end to end, including live DHCP, DNS, NAT, and Internet failover reachability tests from client devices.

## Troubleshooting Encountered

- **HSRP version mismatch:** a missing `standby version 2` on one side of a group caused duplicate-address errors (`%IP-4-DUPADDR`), since both routers believed they were operating alone. Resolved by explicitly matching the HSRP version on both sides; the same class of error did not recur when the second office's HSRP groups were built afterward.
- **DHCP failure isolated to a single host:** systematically traced through the relay agent, the DHCP pool configuration, and R1's binding table before determining the actual cause was local to the client: a manually assigned static IP outside the correct subnet, meaning no DHCP request was ever sent. Corrected by switching the host back to DHCP.
- **OSPF default route not propagating despite correct configuration:** confirmed via `show ip ospf database external` that R1 was not generating the Type-5 external LSA at all, despite valid `default-information originate` and a valid candidate default route. Isolated to a stuck OSPF process state rather than a configuration error, and resolved with a full reload of R1.
- **Running-config VLAN list inconsistency:** after a configuration change, `show run` displayed a merged and duplicated internal VLAN entry alongside the intended VLAN list on one switch's DAI configuration, even though `show ip arp inspection` confirmed the feature was fully operational on the correct VLANs. Resolved by removing and re-adding the VLAN list to force a clean rewrite, and adopted feature-specific `show` commands as the more reliable verification method going forward.
- **VTP mode discrepancy from original design notes:** cross-checked against the source reference material and confirmed one distribution switch was required to operate in VTP server mode, correcting an assumption carried over from earlier planning.
- **Missing NTP authentication key on R1:** caught during a deliberate second review against the requirements. R1 had `ntp master` and `ntp server` configured but was missing the authentication key, `ntp authenticate`, and `ntp trusted-key` commands that client-side authentication depends on.
- **Management VLAN excluded from DHCP snooping and DAI:** initial configuration scoped both features to end-host VLANs only; re-reading the requirement confirmed the management VLAN needed to be included as well, and this was corrected across all six access switches.
- **WLAN bound to the wrong interface:** the WLAN profile defaulted to the built-in "management" interface rather than the newly created "Wi-Fi" dynamic interface. Caught during review of the WLAN's General configuration tab and corrected before wireless clients were tested.

## Key Observations

- Aligning STP root priority with HSRP active role keeps the spanning tree root and the first-hop gateway on the same physical device per VLAN, avoiding suboptimal forwarding paths between the two independent protocols.
- `default-information originate` without the `always` keyword only advertises a default route if the originating router's own routing table already contains a valid default at the moment the command is evaluated, and does not automatically re-check when the underlying route changes. This has direct implications for testing Internet failover scenarios.
- An IPv6 static default route must use `::/0` as its destination; a specific prefix, however broad, does not function as a default route regardless of intent.
- A "fully specified" IPv6 static route names both the exit interface and the next-hop link-local address, avoiding a recursive routing table lookup; a route with only a next-hop global address requires the router to resolve that address via a second lookup.
- Packet Tracer's running-config display can summarize or merge VLAN lists in ways that don't reflect what was actually typed, particularly around internally allocated system VLANs. Feature-specific `show` commands are more reliable for confirming true operational state than `show running-config` alone.

## Skills Demonstrated

- Enterprise Layer 2 design: EtherChannel, 802.1Q trunking, VTP, VLAN segmentation, Rapid PVST+ with deliberate root bridge placement
- First-hop redundancy: HSRPv2 with priority and preempt tuning aligned to spanning tree root placement
- OSPF routing with passive interfaces, point-to-point network types, and ASBR redistribution of a default route
- Network services administration: DHCP server and relay, DNS, authenticated NTP, SNMP, centralized syslog, FTP-based IOS lifecycle management
- Remote access hardening: SSH key size, protocol version, ACL-restricted access, local authentication
- NAT and PAT: static one-to-one translation, pool-based PAT with ACL-defined scope, and floating-route Internet failover
- Layer 2 attack mitigation: port security, DHCP snooping, Dynamic ARP Inspection
- IPv6 fundamentals: unicast routing, EUI-64 addressing, link-local-only interfaces, recursive versus fully specified static routing
- Wireless LAN controller administration: dynamic interfaces, WLAN profiles, WPA2-PSK security, access point association verification
- Independent verification discipline: every configuration stage confirmed against live device output before being considered complete, with multiple self-identified corrections made against original requirements and reference material

## CCNA Exam Alignment

This lab spans the full 200-301 blueprint: Network Fundamentals, Network Access (VLANs, trunking, EtherChannel, wireless), IP Connectivity (OSPF, static and default routing), IP Services (DHCP, NTP, SNMP, syslog, NAT), and Security Fundamentals (port security, DHCP snooping, DAI, ACLs, device hardening). Automation and programmability content is out of scope for this lab.

## Files

- `CCNA Mega Lab (Jeremy's IT Lab).pka` - final Packet Tracer topology file
