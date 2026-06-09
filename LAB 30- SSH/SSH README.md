# CCNA-LABS

## Lab 30 – SSH (Secure Shell)

### Objective

Configure a newly added switch (SW2) from scratch via console connection, apply console line security, and enable remote access via SSH with access restricted to PC1 only. Verify SSH works from PC1 and is refused from R2.

---

### Topology

- 2x Cisco 2911 Routers (R1, R2)
- 2x Cisco 2960 Switches (SW1, SW2)
- 1x PC (PC1): 192.168.1.1
- 1x Laptop (Laptop1): console connected to SW2

```
PC1 (192.168.1.1) - SW1 - R1 (G0/1: .254) - 10.0.0.0/30 - R2 (G0/0: .2) - G0/1: .254 - SW2 (VLAN1: 192.168.2.253) - Laptop1
```

| Device | Interface/SVI | IP Address |
|--------|--------------|------------|
| R1 | G0/1 | 192.168.1.254 |
| R2 | G0/0 | 10.0.0.2 |
| R2 | G0/1 | 192.168.2.254 |
| SW2 | VLAN1 | 192.168.2.253/24 |
| PC1 | NIC | 192.168.1.1/24 |

---

### Configuration Summary

#### Step 1: Basic Configuration (via Laptop1 console connection)

```
SW2(config)# hostname SW2
SW2(config)# enable secret ccna
SW2(config)# username jeremy secret ccna
SW2(config)# interface vlan 1
SW2(config-if)# ip address 192.168.2.253 255.255.255.0
SW2(config-if)# no shutdown
SW2(config)# ip default-gateway 192.168.2.254
```

#### Step 2: Console Line Security

```
SW2(config)# line console 0
SW2(config-line)# login local
SW2(config-line)# exec-timeout 5 0
```

#### Step 3: SSH Configuration

```
! Generate RSA keys (requires hostname + domain name first)
SW2(config)# ip domain name jeremysitlab.com
SW2(config)# crypto key generate rsa
! Enter 2048 when prompted for modulus size

SW2(config)# ip ssh version 2

! ACL to restrict SSH access to PC1 only
SW2(config)# access-list 1 permit host 192.168.1.1

! Configure VTY lines
SW2(config)# line vty 0 15
SW2(config-line)# login local
SW2(config-line)# exec-timeout 5 0
SW2(config-line)# transport input ssh
SW2(config-line)# access-class 1 in
```

---

### Verification

| Test | Command | Expected Result |
|------|---------|-----------------|
| SW2 reachability from PC1 | `ping 192.168.2.253` | Success (first 3 may timeout due to ARP) |
| SSH from PC1 | `ssh -l jeremy 192.168.2.253` | Prompted for password, connects successfully |
| SSH from R2 | `ssh -l jeremy 192.168.2.253` | Connection refused by remote host |
| R2 ping SW2 | `ping 192.168.2.253` | Success; ACL only blocks SSH, not ICMP |
| Config check | `show run` | All VTY lines show login local, transport input ssh, access-class 1 in |

---

### Troubleshooting Encountered

- `line vty 015` (no space) only configured line 15; correct syntax is `line vty 0 15` with a space
- `crypto key generate rsa modulus 2048` is not supported in Packet Tracer; must enter `crypto key generate rsa` and specify modulus size at the prompt
- `no username julian` must be run from global config mode, not privileged EXEC
- Steps were completed out of order (SSH configured before basic config); verified final running-config matched all requirements before testing
- Packet Tracer output from PC1 was accidentally pasted into R2's CLI; R2 treated each line as a command and rejected them; corrected by running `ssh` directly from R2's prompt

---

### Key Observations

- `crypto key generate rsa` requires both a non-default hostname AND a domain name; missing either causes the command to be rejected
- After RSA keys are generated, a Syslog message confirms SSH 1.99 has been enabled; version 1.99 means the device supports both SSHv1 and SSHv2
- `ip ssh version 2` forces SSHv2 only (best practice)
- `access-class 1 in` on VTY lines restricts which source IPs can connect; R2 was able to ping SW2 but SSH was refused because R2's IP is not permitted by ACL 1
- `login local` is required for SSH; `login` alone (line password only) does not work with SSH
- `ip default-gateway` is required on Layer 2 switches for communication with devices outside the local subnet; without it SW2 could not be reached from PC1 across R1 and R2
- Packet Tracer splits VTY line config display into 0-4 and 5-15 sections even when configured as 0-15; both sections must show identical config

---

### Skills Demonstrated

- Console cable connection and terminal access to unconfigured switch
- Basic switch hardening (enable secret, local username, hostname)
- Layer 2 switch management IP configuration (SVI + default gateway)
- Console line security (login local, exec-timeout)
- SSH configuration from scratch (hostname, domain name, RSA keys, SSHv2, VTY lines)
- ACL-based VTY line access restriction using access-class
- SSH verification and negative testing (confirmed refused connection from unauthorized host)

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 4.8 | Configure network devices for remote access using SSH |

---

### Files

- `Lab30-SSH.pkt`
