# CCNA-LABS

## Lab 31 – FTP & TFTP (File Transfer Protocols)

### Objective

Use TFTP to copy a new IOS image to R1 from SRV1, and FTP to copy the same image to R2. Configure both routers to boot from the new IOS, reload, verify the upgrade, and delete the old IOS files from flash.

---

### Topology

- 2x Cisco 2911 Routers (R1, R2)
- 1x Cisco 2960 Switch (SW1)
- 1x Server-PT (SRV1): 10.0.0.1 (TFTP and FTP server)

```
SRV1 (10.0.0.1) - SW1 - R1 (G0/1: 10.0.0.254, G0/0: 192.168.12.1) - R2 (G0/0: 192.168.12.2)
```

| Device | Interface | IP Address |
|--------|-----------|------------|
| SRV1 | Fa0 | 10.0.0.1/24 |
| R1 | G0/1 | 10.0.0.254/24 |
| R1 | G0/0 | 192.168.12.1/30 |
| R2 | G0/0 | 192.168.12.2/30 |

---

### Configuration Summary

#### Step 1: IP Addressing and Routing

```
! R1
R1(config)# interface g0/1
R1(config-if)# ip address 10.0.0.254 255.255.255.0
R1(config-if)# no shutdown
R1(config)# interface g0/0
R1(config-if)# ip address 192.168.12.1 255.255.255.252
R1(config-if)# no shutdown

! R2
R2(config)# interface g0/0
R2(config-if)# ip address 192.168.12.2 255.255.255.252
R2(config-if)# no shutdown
R2(config)# ip route 10.0.0.0 255.255.255.0 192.168.12.1

! SRV1: IP 10.0.0.1/24, default gateway 10.0.0.254 (configured via GUI)
```

#### Step 2: TFTP IOS Upgrade on R1

```
! Verify current version and flash contents
R1# show version
R1# show flash

! Copy new IOS from SRV1 via TFTP
R1# copy tftp: flash:
  Address: 10.0.0.1
  Source filename: c2900-universalk9-mz.SPA.155-3.M4a.bin
  Destination: [Enter to keep same name]

! Configure boot from new IOS
R1(config)# boot system flash:c2900-universalk9-mz.SPA.155-3.M4a.bin

! Save and reload
R1# write memory
R1# reload

! Verify new version
R1# show version

! Delete old IOS
R1# delete flash:c2900-universalk9-mz.SPA.151-4.M4.bin
```

#### Step 3: FTP IOS Upgrade on R2

```
! Configure FTP credentials
R2(config)# ip ftp username jeremy
R2(config)# ip ftp password ccna

! Copy new IOS from SRV1 via FTP
R2# copy ftp: flash:
  Address: 10.0.0.1
  Source filename: c2900-universalk9-mz.SPA.155-3.M4a.bin
  Destination: [Enter to keep same name]

! Configure boot from new IOS
R2(config)# boot system flash:c2900-universalk9-mz.SPA.155-3.M4a.bin

! Save and reload
R2# write memory
R2# reload

! Verify new version
R2# show version

! Delete old IOS
R2# delete flash:c2900-universalk9-mz.SPA.151-4.M4.bin
```

---

### Verification

| Device | Command | Expected Result |
|--------|---------|-----------------|
| R1 | `show version` | Version 15.5(3)M4a |
| R1 | `show flash` | Only new IOS file present |
| R2 | `show version` | Version 15.5(3)M4a |
| R2 | `show flash` | Only new IOS file present |
| R2 | `ping 10.0.0.1` | 5/5 success before beginning transfers |

---

### Troubleshooting Encountered

- SRV1 had no default gateway configured; R2 could not reach it because replies had no path back; fixed by setting SRV1's default gateway to 10.0.0.254 via the Config tab GUI
- `boot system flash:` entered from privileged EXEC mode fails; must be in global config mode
- `write` entered from inside config mode fails; must exit to privileged EXEC first then run `write memory`
- TFTP entered wrong source filename on first attempt (old file instead of new); cancelled and re-entered correctly
- FTP transfer is significantly slower than TFTP in Packet Tracer; used the fast-forward button to speed up the simulation

---

### Key Observations

- TFTP requires no authentication; just server IP and filename
- FTP requires credentials configured with `ip ftp username` and `ip ftp password` before the copy command
- `boot system flash:` must be configured AND config must be saved with `write memory` before reload; without saving the command is ignored on reboot
- Both flash files coexist until the old one is deleted; `show flash` confirms what is present before and after deletion
- After reload, `show version` confirms the running IOS image matches the new file in flash
- R1 did not need a static route to reach SRV1 since the 10.0.0.0/24 network was directly connected via G0/1; R2 needed a static route pointing to R1

---

### Skills Demonstrated

- IP addressing and static routing for multi-router connectivity
- TFTP file transfer from server to router flash
- FTP credential configuration and file transfer
- IOS upgrade process: copy, boot system, write, reload, verify, delete
- Flash memory management using `show flash` and `delete flash:`
- Troubleshooting connectivity issues by verifying default gateway on end devices

---

### CCNA Exam Alignment

| Exam Topic | Description |
|------------|-------------|
| 4.9 | Describe the capabilities and function of TFTP and FTP in the network |

---

### Files

- `Lab31-FTP-TFTP.pkt`
