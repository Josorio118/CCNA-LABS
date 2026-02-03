# CCNA-LABS
📌 Project: Inter-VLAN Routing with Router-on-a-Stick
Objective
Design and validate a segmented Layer 2 network with multiple VLANs and enable Layer 3 communication using router-on-a-stick architecture.
Topology
Cisco 2960 Switch
Cisco 2911 Router
Two end hosts in separate VLANs
802.1Q trunk between switch and router
VLAN Design
VLAN	Name	Subnet
10	USER	192.168.10.0/24
20	TEST	192.168.20.0/24
99	MGMT	192.168.99.0/24
Key Configurations
Access ports assigned per VLAN
Trunk configured with allowed VLANs
Router subinterfaces with dot1Q encapsulation
Static IP addressing on hosts
Default gateways mapped to router subinterfaces
Validation
Successful intra-VLAN communication
Predictable inter-VLAN communication failure prior to policy changes
Verified routing table and trunk state
Used ICMP testing to confirm design correctness
Skills Demonstrated
VLAN segmentation
802.1Q trunking
Inter-VLAN routing
Network troubleshooting
CCNA-level network design principles
