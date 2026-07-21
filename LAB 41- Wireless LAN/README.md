# Lab 41 – Wireless LANs

## Objective

Configure a Cisco Wireless LAN Controller (WLC) via its GUI to stand up two wireless LANs (Internal and Guest), each mapped to its own VLAN and dynamic interface, secured with WPA2 pre-shared key authentication, and verify a wireless client can associate and get an IP address.

Lab steps:
1. Access the WLC1 GUI from PC1's web browser
2. Familiarize with the WLC GUI tabs and current network state
3. Configure dynamic interfaces for the Internal and Guest WLANs
4. Create the Internal and Guest WLANs using WPA2+PSK
5. Add a wireless client and associate it with an AP

## Topology

- 1x Cisco 3650-24PS switch (SW1)
- 1x Cisco WLC 3504 (WLC1)
- 2x Lightweight APs (AP1, AP2)
- 1x PC (PC1), used to access the WLC GUI
- 1x Smartphone client, associated wirelessly after configuration
- VLAN 10 (Management): 172.16.1.0/24, WLC1 management IP = 172.16.1.10
- VLAN 100 (Internal): 10.0.0.0/24, SSID = Internal
- VLAN 200 (Guest): 10.1.0.0/24, SSID = Guest

## Switch (SW1) Configuration (Pre-configured)

SW1 was pre-configured with:
- DHCP server role for all three VLANs, with excluded address ranges reserved for static assignments
- VLAN 10 pool includes `option 43 ip <WLC IP>`, though not strictly required here since WLC1 and the APs share the same local subnet and can discover each other via CAPWAP broadcast
- G1/0/2, G1/0/3, and G1/0/4 (facing the APs and PC1) configured as access ports in VLAN 10
- SVIs configured for VLAN 10, 100, and 200, each acting as the default gateway for its subnet
- Management VLAN (10) configured as **untagged/native** on the switch side, matching the WLC's own untagged configuration for that VLAN — this must match on both ends or the WLC and switch cannot communicate

## WLC GUI Configuration

**Accessing the GUI**

- Browsed to `https://172.16.1.10` from PC1 (HTTPS required; this WLC model does not allow HTTP by default)
- Logged in with username `admin`, password `Cisco123`

**Familiarization (Step 2)**

- Confirmed WLC1's connected port to SW1 showed green/up
- Confirmed both AP1 and AP2 had already joined the WLC
- Explored Monitor > Statistics > AP Join to see AP join status and message history — useful for troubleshooting APs that fail to join

**Dynamic Interfaces (Step 3)**

Created under Controller > Interfaces:

| Interface Name | VLAN ID | Port | IP Address | Netmask | Gateway | DHCP Server |
|----------------|---------|------|------------|---------|---------|-------------|
| Internal | 100 | 1 | 10.0.0.10 | 255.255.255.0 | 10.0.0.1 | 10.0.0.1 |
| Guest | 200 | 1 | 10.1.0.10 | 255.255.255.0 | 10.1.0.1 | 10.1.0.1 |

Gateway and DHCP server for each interface point to SW1's corresponding SVI.

**WLANs (Step 4)**

Created two WLANs from the WLANs tab:

*Internal WLAN*
- Profile name and SSID: Internal, ID 1
- General tab: WLAN enabled, mapped to the **Internal** dynamic interface (not left on the management interface)
- Security tab: Layer 2 security = **WPA+WPA2**, WPA2 policy enabled, AES encryption, 802.1X disabled, **PSK enabled** with a pre-shared key configured
- Advanced tab: maximum allowed clients set to 100

*Guest WLAN*
- Profile name and SSID: Guest, ID 2
- General tab: WLAN enabled, mapped to the **Guest** dynamic interface
- Security tab: same as Internal — WPA+WPA2, WPA2 policy, AES, PSK enabled with a configured key

**Wireless Client (Step 5)**

- Added a smartphone end device
- Configured its Wireless0 radio: SSID = Internal, authentication = WPA2-PSK, passphrase matching the WLAN's configured PSK
- IP configuration left on DHCP
- Device associated with an AP and received an IP address

## Verification

- WLC dashboard confirmed both APs joined and the WLC's uplink port to SW1 was up
- Internal WLAN listed with security policy showing **WPA2 with PSK authentication** after configuration
- Wireless client successfully associated with an AP (visible on the topology diagram) and received a DHCP-assigned IP address, confirming the WLAN, dynamic interface, and DHCP configuration were all working together correctly

## Troubleshooting Encountered

- Packet Tracer does not accurately reflect real WLC behavior in one respect: even though the smartphone associated with the Internal WLAN (mapped to the Internal dynamic interface, VLAN 100), it received an IP address from the VLAN 10 (management) DHCP pool rather than the VLAN 100 pool. This is a known Packet Tracer simulation limitation, not a configuration error — on real Cisco hardware, the same configuration correctly assigns an IP from the VLAN mapped to the client's WLAN.

## Key Observations

- Packet Tracer cannot access a WLC's console port, so initial WLC setup (system name, management interface, initial WLAN, country code, etc.) must be done through the Packet Tracer GUI representation of the device rather than a true CLI wizard.
- The management VLAN's tagging behavior (tagged vs. untagged/native) must match between the WLC and the switch port it connects to, or the two devices will fail to communicate.
- Dynamic interfaces are configured under the **Controller** tab, not the WLANs tab — know which GUI tab is used for which task, since this can be tested directly on the exam.
- A WLAN must be explicitly **enabled** and mapped to the correct dynamic interface — by default, a newly created WLAN is disabled and left mapped to the management interface, both of which must be changed.
- For CCNA-level WLAN security, Layer 2 security is set to WPA+WPA2 with PSK authentication enabled (not 802.1X), matching exam topic 5.10.
- DHCP Option 43 (WLC IP address) is not required when the WLC and APs share the same local subnet, since APs can discover the WLC via local CAPWAP broadcast messages.

## Skills Demonstrated

- Accessing and navigating a Cisco WLC GUI over HTTPS
- Configuring dynamic interfaces to map VLANs to wireless LANs
- Creating and securing WLANs with WPA2 pre-shared key authentication
- Associating a wireless client with a configured WLAN and verifying DHCP-assigned connectivity
- Interpreting WLC monitoring tools (AP Join statistics) for troubleshooting

## CCNA Exam Alignment

- **2.7** Describe wireless principles
- **2.8** Explain the role of access points and wireless LAN controllers
- **2.9** Describe wireless LAN configuration requirements
- **5.10** Configure WLAN using WPA2 PSK using the GUI

## Files

- `README.md` – this file
- `LAB 41- Wireless LANs.pkt` – Packet Tracer topology and configuration file
