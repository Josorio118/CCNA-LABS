# CCNA-LABS

## Lab 35 – QoS (Quality of Service)

### Objective

Configure basic QoS on R1 using class maps, a policy map, and a service policy to classify and mark HTTPS, HTTP, and ICMP traffic with appropriate DSCP values. Apply the policy outbound on G0/0/0 and verify markings in simulation mode.

> **Note:** QoS configuration is not a CCNA exam topic. This lab provides hands-on exposure to how QoS works in practice — class maps identify traffic, policy maps define actions, service policies apply the policy to an interface.

---

### Topology

- 1x ISR Router (R1) — QoS configured outbound on G0/0/0
- 1x ISR4331 Router (R2)
- 2x Cisco 2960 Switches (SW1, SW2)
- 1x PC (PC1) — 192.168.0.10
- 1x Server-PT (SRV1) — 10.0.0.100 (jeremysitlab.com)

```
PC1 (192.168.0.10) - SW1 - R1 (G0/0/1: 192.168.0.1, G0/0/0: 172.16.0.1) - R2 - SW2 - SRV1 (10.0.0.100)
```

Default route on R1 pointing to 172.16.0.2 (R2) was pre-configured.

---

### Configuration Summary

#### Step 1: Create Class Maps (Identify Traffic)

```
R1(config)# class-map HTTPS_MAP
R1(config-cmap)# match protocol https

R1(config)# class-map HTTP_MAP
R1(config-cmap)# match protocol http

R1(config)# class-map ICMP_MAP
R1(config-cmap)# match protocol icmp
```

#### Step 2: Create Policy Map (Define Actions)

```
R1(config)# policy-map G0/0/0_OUT

R1(config-pmap)# class HTTPS_MAP
R1(config-pmap-c)# set ip dscp af31
R1(config-pmap-c)# priority percent 10

R1(config-pmap)# class HTTP_MAP
R1(config-pmap-c)# set ip dscp af32
R1(config-pmap-c)# bandwidth percent 10

R1(config-pmap)# class ICMP_MAP
R1(config-pmap-c)# set ip dscp cs2
R1(config-pmap-c)# bandwidth percent 5
```

#### Step 3: Apply Service Policy to Interface

```
R1(config)# interface g0/0/0
R1(config-if)# service-policy output G0/0/0_OUT
```

#### Step 4: Verify DSCP Markings in Simulation Mode

Sent ICMP, HTTP, and HTTPS traffic from PC1 to SRV1 and inspected outbound PDU details at R1.

---

### Verification

| Traffic Type | DSCP Marking | Hex Value | Decimal | Confirmed |
|-------------|-------------|-----------|---------|-----------|
| ICMP (ping) | CS2 | 0x10 | 16 | Yes |
| HTTPS | AF31 | 0x1a | 26 | Yes |
| HTTP | AF32 | 0x1c | 28 | (configured) |
| DNS query | DF (default) | 0x00 | 0 | Yes — unclassified traffic unmarked |

DSCP verification formula used: AF31 = 8(3) + 2(1) = 26 = 0x1a; CS2 = 2 × 8 = 16 = 0x10

---

### Troubleshooting Encountered

- `int g0/0/0class-map HTTPS_MAP` typed without a newline; caused invalid input error — commands must be entered separately
- Accidentally exited policy-map config before adding the ICMP class; re-entered the policy-map and added it successfully
- `show running-config` rejected from interface config mode without `do` prefix

---

### Key Observations

- Class maps use `match-all` by default — traffic must match ALL match statements in the class map (only one each here, so no difference)
- DNS traffic (UDP port 53) is not matched by any class map and exits with DSCP 0x00 (default forwarding); QoS only affects classified traffic
- `priority percent 10` creates a strict priority queue for HTTPS; `bandwidth percent 10` guarantees minimum bandwidth for HTTP but without strict priority
- DSCP markings are set in the IP header's ToS byte and survive across routers; PCP/CoS only works on tagged L2 links
- The service policy is applied `output` on G0/0/0 — R1 marks packets as they are forwarded out toward R2; R2 would need its own QoS config to continue honoring the markings (per-hop behavior)
- QoS configuration requires three steps in order: class-map (identify) → policy-map (define actions) → service-policy on interface (apply)

---

### Skills Demonstrated

- Class map configuration using `match protocol`
- Policy map configuration with DSCP marking and bandwidth allocation
- Strict priority queue vs guaranteed bandwidth queue distinction
- Service policy application on an interface
- DSCP value verification using simulation mode PDU inspection
- Binary-to-hex DSCP conversion (AF formula: 8x + 2y; CS formula: CS# × 8)

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 4.7 | Explain the forwarding per-hop behavior (PHB) for QoS such as classification, marking, queuing, congestion, policing, and shaping |

> QoS **configuration** is not required for the CCNA exam. This lab covers the conceptual operation only.

---

### Files

- `Lab35-QoS.pkt`
