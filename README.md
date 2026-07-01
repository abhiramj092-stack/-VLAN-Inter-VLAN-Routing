# -VLAN-Inter-VLAN-Routing
📋 Table of Contents


Abstract
Introduction
Network Topology
IP Addressing Table
VLAN Design
Configuration — Step by Step

Switch — Basic VLAN Setup
Trunk Port Configuration
Router-on-a-Stick (ROAS)
Layer 3 Switch Method
DHCP Configuration
Port Security



Verification Commands
Results — Ping Test
ROAS vs Layer 3 Switch
Applications
Conclusion
Tools Used
References



Abstract

This project demonstrates network segmentation using VLANs (Virtual Local Area Networks) and Inter-VLAN routing in a simulated enterprise environment using Cisco Packet Tracer.

Three VLANs — HR (VLAN 10), IT (VLAN 20), and Admin (VLAN 30) — are created to isolate broadcast domains. Inter-VLAN communication is achieved using two methods:


Router-on-a-Stick (ROAS) — using sub-interfaces on a single router interface
Layer 3 Switch — using Switched Virtual Interfaces (SVI)


The simulation verifies successful intra-VLAN and inter-VLAN communication using ping tests, validating the correct operation of 802.1Q trunking and IP routing.


Introduction

What is a VLAN?

A VLAN (Virtual Local Area Network) is a logical grouping of network devices, regardless of their physical location, within a single broadcast domain. VLANs are configured on Layer 2 switches using IEEE 802.1Q standard tagging.

Why VLANs?

Problem (Without VLAN)Solution (With VLAN)All devices in one broadcast domainSeparate broadcast domains per departmentSecurity risk — all traffic visible to allTraffic isolated between departmentsNetwork congestion due to broadcastsReduced broadcast trafficNo logical separation of departmentsLogical grouping regardless of location

Why Inter-VLAN Routing?

VLANs by default cannot communicate with each other (that's the point — isolation). But in real networks, HR needs to access a shared file server in IT. This requires Inter-VLAN Routing — a Layer 3 (routing) function that allows controlled communication between VLANs.

Network Topology

                         ┌──────────────────┐
                         │  ISR4331 Router0  │
                         │  (ROAS Method)    │
                         │  Fa0/0.10         │
                         │  Fa0/0.20         │
                         │  Fa0/0.30         │
                         └────────┬──────────┘
                                  │ Trunk (802.1Q)
                         ┌────────▼──────────┐
                         │ 3560-24PS         │
                         │ Multilayer Switch0 │
                         │ (L3 Switch / Core) │
                         └───┬───────┬────┬──┘
               Trunk         │       │    │         Trunk
        ┌────────────────────┘       │    └─────────────────┐
   ┌────▼──────────┐       ┌─────────▼──────┐      ┌────────▼──────┐
   │ 2960-24TT     │       │ 2960-24TT      │      │ 2960-24TT     │
   │   Switch0     │       │   Switch1      │      │   Switch2     │
   └──┬────────┬───┘       └──┬─────────┬──┘      └──┬────────┬───┘
      │        │              │         │             │        │
    PC-PT    PC-PT          PC-PT     PC-PT         PC-PT    PC-PT
     PC0      PC1            PC2       PC3           PC4      PC5
  [VLAN 10] [VLAN 10]    [VLAN 20] [VLAN 20]    [VLAN 30] [VLAN 30]
        Sales Dept             Mgmt Dept               Admin Dept


Device Summary

DeviceModelRoleRouter0Cisco ISR4331Inter-VLAN routing (ROAS)Multilayer Switch0Cisco 3560-24PSCore switch + L3 SVI routingSwitch0Cisco 2960-24TTAccess switch — Sales (VLAN 10)Switch1Cisco 2960-24TTAccess switch — Mgmt (VLAN 20)Switch2Cisco 2960-24TTAccess switch — Admin (VLAN 30)PC0, PC1PC-PTSales department end devicesPC2, PC3PC-PTManagement department end devicesPC4, PC5PC-PTAdmin department end devices


IP Addressing Table

DeviceInterfaceVLANIP AddressSubnet MaskDefault GatewayRouter0 (ISR4331)Gig0/0/0.1010192.168.10.1255.255.255.0—Router0 (ISR4331)Gig0/0/0.2020192.168.20.1255.255.255.0—Router0 (ISR4331)Gig0/0/0.3030192.168.30.1255.255.255.0—Multilayer Switch0VLAN 10 SVI10192.168.10.1255.255.255.0—Multilayer Switch0VLAN 20 SVI20192.168.20.1255.255.255.0—Multilayer Switch0VLAN 30 SVI30192.168.30.1255.255.255.0—PC0 (Sales)NIC10192.168.10.10255.255.255.0192.168.10.1PC1 (Sales)NIC10192.168.10.11255.255.255.0192.168.10.1PC2 (Mgmt)NIC20192.168.20.10255.255.255.0192.168.20.1PC3 (Mgmt)NIC20192.168.20.11255.255.255.0192.168.20.1PC4 (Admin)NIC30192.168.30.10255.255.255.0192.168.30.1PC5 (Admin)NIC30192.168.30.11255.255.255.0192.168.30.1


Note: All IP addresses are statically configured on each PC. No DHCP used in this project.




VLAN Design

VLAN IDNameDepartmentNetworkSwitchPCsVLAN 10SALESSales Department192.168.10.0/24Switch0PC0, PC1VLAN 20MGMTManagement Department192.168.20.0/24Switch1PC2, PC3VLAN 30ADMINAdmin Department192.168.30.0/24Switch2PC4, PC5VLAN 99NATIVENative Trunk VLAN—All trunk links—


Configuration — Step by Step

1. Access Switches — VLAN Assignment


Run on Switch0, Switch1, Switch2 (all three 2960s):



bashSwitch> enable
Switch# configure terminal

Switch(config)# vlan 10
Switch(config-vlan)# name SALES
Switch(config-vlan)# exit

Switch(config)# vlan 20
Switch(config-vlan)# name MGMT
Switch(config-vlan)# exit

Switch(config)# vlan 30
Switch(config-vlan)# name ADMIN
Switch(config-vlan)# exit

Switch0 — PC0 and PC1 → VLAN 10 (Sales):

bashSwitch0(config)# interface fastEthernet 0/1
Switch0(config-if)# switchport mode access
Switch0(config-if)# switchport access vlan 10
Switch0(config-if)# exit

Switch0(config)# interface fastEthernet 0/2
Switch0(config-if)# switchport mode access
Switch0(config-if)# switchport access vlan 10
Switch0(config-if)# exit

Switch1 — PC2 and PC3 → VLAN 20 (Management):

bashSwitch1(config)# interface fastEthernet 0/1
Switch1(config-if)# switchport mode access
Switch1(config-if)# switchport access vlan 20
Switch1(config-if)# exit

Switch1(config)# interface fastEthernet 0/2
Switch1(config-if)# switchport mode access
Switch1(config-if)# switchport access vlan 20
Switch1(config-if)# exit

Switch2 — PC4 and PC5 → VLAN 30 (Admin):

bashSwitch2(config)# interface fastEthernet 0/1
Switch2(config-if)# switchport mode access
Switch2(config-if)# switchport access vlan 30
Switch2(config-if)# exit

Switch2(config)# interface fastEthernet 0/2
Switch2(config-if)# switchport mode access
Switch2(config-if)# switchport access vlan 30
Switch2(config-if)# exit

Verify on each switch:

bashSwitch# show vlan brief


2. Trunk Port Configuration

Multilayer Switch0 — Trunk toward Router0:

bashMLS0(config)# interface gigabitEthernet 0/1
MLS0(config-if)# switchport trunk encapsulation dot1q
MLS0(config-if)# switchport mode trunk
MLS0(config-if)# switchport trunk native vlan 99
MLS0(config-if)# switchport trunk allowed vlan 10,20,30,99
MLS0(config-if)# exit

Multilayer Switch0 — Trunk toward each 2960 (Switch0, Switch1, Switch2):

bashMLS0(config)# interface gigabitEthernet 0/2
MLS0(config-if)# switchport trunk encapsulation dot1q
MLS0(config-if)# switchport mode trunk
MLS0(config-if)# switchport trunk allowed vlan 10,20,30,99
MLS0(config-if)# exit

MLS0(config)# interface gigabitEthernet 0/3
MLS0(config-if)# switchport trunk encapsulation dot1q
MLS0(config-if)# switchport mode trunk
MLS0(config-if)# switchport trunk allowed vlan 10,20,30,99
MLS0(config-if)# exit

MLS0(config)# interface gigabitEthernet 0/4
MLS0(config-if)# switchport trunk encapsulation dot1q
MLS0(config-if)# switchport mode trunk
MLS0(config-if)# switchport trunk allowed vlan 10,20,30,99
MLS0(config-if)# exit

Switch0 / Switch1 / Switch2 — Uplink trunk toward Multilayer Switch0:

bashSwitch(config)# interface gigabitEthernet 0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30,99
Switch(config-if)# exit

Verify:

bashMLS0# show interfaces trunk


3. Router-on-a-Stick (ROAS) — ISR4331 Router0


The ISR4331 uses GigabitEthernet0/0/0 (not Fa0/0):



bashRouter0> enable
Router0# configure terminal

! Enable the physical interface
Router0(config)# interface gigabitEthernet 0/0/0
Router0(config-if)# no shutdown
Router0(config-if)# exit

! Sub-interface — VLAN 10 Sales
Router0(config)# interface gigabitEthernet 0/0/0.10
Router0(config-subif)# encapsulation dot1Q 10
Router0(config-subif)# ip address 192.168.10.1 255.255.255.0
Router0(config-subif)# exit

! Sub-interface — VLAN 20 Management
Router0(config)# interface gigabitEthernet 0/0/0.20
Router0(config-subif)# encapsulation dot1Q 20
Router0(config-subif)# ip address 192.168.20.1 255.255.255.0
Router0(config-subif)# exit

! Sub-interface — VLAN 30 Admin
Router0(config)# interface gigabitEthernet 0/0/0.30
Router0(config-subif)# encapsulation dot1Q 30
Router0(config-subif)# ip address 192.168.30.1 255.255.255.0
Router0(config-subif)# exit

! Native VLAN sub-interface
Router0(config)# interface gigabitEthernet 0/0/0.99
Router0(config-subif)# encapsulation dot1Q 99 native
Router0(config-subif)# ip address 192.168.99.1 255.255.255.0
Router0(config-subif)# exit

Verify:

bashRouter0# show ip route
Router0# show ip interface brief
Router0# show interfaces gigabitEthernet 0/0/0.10


4. Layer 3 Switch SVI Method — Multilayer Switch0


The 3560-24PS supports Layer 3 routing using SVIs:



bashMLS0> enable
MLS0# configure terminal

! Enable IP routing
MLS0(config)# ip routing

! SVI for VLAN 10 — Sales
MLS0(config)# interface vlan 10
MLS0(config-if)# ip address 192.168.10.1 255.255.255.0
MLS0(config-if)# no shutdown
MLS0(config-if)# exit

! SVI for VLAN 20 — Management
MLS0(config)# interface vlan 20
MLS0(config-if)# ip address 192.168.20.1 255.255.255.0
MLS0(config-if)# no shutdown
MLS0(config-if)# exit

! SVI for VLAN 30 — Admin
MLS0(config)# interface vlan 30
MLS0(config-if)# ip address 192.168.30.1 255.255.255.0
MLS0(config-if)# no shutdown
MLS0(config-if)# exit

Verify:

bashMLS0# show ip route
MLS0# show interfaces vlan 10
MLS0# show interfaces vlan 20
MLS0# show interfaces vlan 30


Verification Commands

bash! ── ACCESS SWITCH ────────────────────────────────────────────
Switch# show vlan brief               ! VLAN list + port assignments
Switch# show interfaces trunk         ! Trunk link status
Switch# show mac address-table        ! MAC addresses per VLAN

! ── MULTILAYER SWITCH ────────────────────────────────────────
MLS0# show interfaces trunk           ! All trunk ports
MLS0# show ip route                   ! Routing table (L3 method)
MLS0# show interfaces vlan 10         ! SVI status

! ── ROUTER ───────────────────────────────────────────────────
Router0# show ip interface brief      ! Sub-interface status
Router0# show ip route                ! ROAS routing table

! ── PC PING TESTS ────────────────────────────────────────────
! From PC0 (192.168.10.10 — Sales, Switch0):
ping 192.168.10.11    ! Intra-VLAN  → PASS ✅ (PC0 to PC1, same switch)
ping 192.168.20.10    ! Inter-VLAN  → PASS ✅ (PC0 to PC2, Switch1)
ping 192.168.30.10    ! Inter-VLAN  → PASS ✅ (PC0 to PC4, Switch2)
ping 192.168.10.1     ! Gateway     → PASS ✅

! From PC2 (192.168.20.10 — Mgmt, Switch1):
ping 192.168.30.10    ! Inter-VLAN  → PASS ✅ (PC2 to PC4)
ping 192.168.10.10    ! Inter-VLAN  → PASS ✅ (PC2 to PC0)

! ── TRACEROUTE ───────────────────────────────────────────────
tracert 192.168.20.10     ! From PC0 → shows path via router/MLS


Results — Ping Test

SourceSource IPDestinationDest IPPathResultPC0 (Sales)192.168.10.10PC1 (Sales)192.168.10.11VLAN 10 → VLAN 10✅ PassPC0 (Sales)192.168.10.10PC2 (Mgmt)192.168.20.10VLAN 10 → VLAN 20✅ PassPC0 (Sales)192.168.10.10PC4 (Admin)192.168.30.10VLAN 10 → VLAN 30✅ PassPC2 (Mgmt)192.168.20.10PC4 (Admin)192.168.30.10VLAN 20 → VLAN 30✅ PassPC0 (Sales)192.168.10.10PC2 (Mgmt)192.168.20.10No routing❌ Fail (expected)


Cross-VLAN ping fails without routing configured — confirming VLAN isolation works correctly before routing is enabled.




ROAS vs Layer 3 Switch Comparison

FeatureRouter-on-a-Stick (ISR4331)Layer 3 Switch (3560-24PS)HardwareSeparate router + switchSingle multilayer switchCostHigher (two devices)Lower (one device)PerformanceRouter is single bottleneckWire-speed hardware routingScalabilityLimited sub-interfacesEasily scalable SVIsKey commandencapsulation dot1Qip routing + SVILatencySlightly higherLowerBest forSmall networks / learning labsEnterprise networksUsed in this project✅ Yes — ISR4331 Router0✅ Yes — 3560-24PS MLS0


Real-World Applications


Enterprise offices — Sales, Management, Admin VLANs on shared physical infrastructure with controlled inter-department access
University campuses — Student, Faculty, Lab, and Admin VLANs logically separated across shared switches
Data centres — Application, Storage, and Management traffic on isolated VLANs
Healthcare — Medical device VLANs strictly separated from general staff VLANs for compliance
Hotels / Airports — Guest Wi-Fi VLAN isolated from backend operations network



Conclusion

This project successfully demonstrates:


✅ VLAN creation and port assignment — VLAN 10 (SALES), VLAN 20 (MGMT), VLAN 30 (ADMIN)
✅ Three 2960-24TT access switches each serving two PCs in dedicated VLANs
✅ 802.1Q trunk links between all switches and the Multilayer Switch0 core
✅ Inter-VLAN routing using Router-on-a-Stick on the Cisco ISR4331 Router0
✅ Inter-VLAN routing using Layer 3 SVI on the Cisco 3560-24PS Multilayer Switch0
✅ Static IP addressing for all 6 PCs (PC0–PC5)
✅ Successful cross-VLAN ping verified between all three departments


VLANs enable security, broadcast control, and logical segmentation without extra physical hardware. Combined with Inter-VLAN routing, they form the backbone of modern enterprise network design.


Project Structure

vlan-inter-vlan-routing/
│
├── README.md                          ← This file
├── inter_vlan_routing.pkt             ← Cisco Packet Tracer topology file
├── configs/
│   ├── Router0_ISR4331_config.txt     ← ROAS sub-interface configuration
│   ├── MLS0_3560_config.txt           ← Multilayer Switch SVI config
│   ├── Switch0_2960_config.txt        ← VLAN 10 Sales access switch
│   ├── Switch1_2960_config.txt        ← VLAN 20 Mgmt access switch
│   └── Switch2_2960_config.txt        ← VLAN 30 Admin access switch
├── topology/
│   └── network_diagram.png            ← Packet Tracer topology screenshot
└── results/
    ├── ping_intra_vlan.png            ← Same-VLAN ping success
    └── ping_inter_vlan.png           ← Cross-VLAN ping success


Tools Used

ToolModel / VersionPurposeCisco Packet Tracer8.2+Network simulationCisco ISR4331IOS 16.xRouter — ROAS Inter-VLAN routingCisco 3560-24PSIOS 15.xMultilayer Switch — SVI routing + coreCisco 2960-24TTIOS 15.xAccess switches — VLAN port assignment


References


Cisco Systems. VLAN Configuration Guide — Cisco IOS. https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960/software/release/15_0_2_se/configuration/guide/scg2960/swvlan.html
Odom, W. (2020). CCNA 200-301 Official Cert Guide, Volume 1. Cisco Press.
IEEE Standard 802.1Q — Bridges and Bridged Networks. IEEE Standards Association.
Lammle, T. (2020). CCNA Cisco Certified Network Associate Study Guide. Sybex.
Cisco Networking Academy. CCNA: Switching, Routing, and Wireless Essentials. https://www.netacad.com
