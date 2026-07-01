# 🌐 VLAN & Inter-VLAN Routing — EEE Mini Project

> **Cisco Packet Tracer** simulation implementing VLAN segmentation and Inter-VLAN routing using a **Cisco ISR4331 Router** (ROAS) and **Cisco 3560-24PS Multilayer Switch** (L3 Switch SVI method).

---

## 📋 Table of Contents

- [Abstract](#abstract)
- [Introduction](#introduction)
- [Network Topology](#network-topology)
- [Device Summary](#device-summary)
- [IP Addressing Table](#ip-addressing-table)
- [VLAN Design](#vlan-design)
- [Configuration — Step by Step](#configuration--step-by-step)
  - [Access Switch VLAN Setup](#1-access-switches--vlan-assignment)
  - [Trunk Port Configuration](#2-trunk-port-configuration)
  - [Router-on-a-Stick ROAS](#3-router-on-a-stick-roas--isr4331-router0)
  - [Layer 3 Switch SVI Method](#4-layer-3-switch-svi-method--multilayer-switch0)
- [Verification Commands](#verification-commands)
- [Results — Ping Test](#results--ping-test)
- [ROAS vs Layer 3 Switch](#roas-vs-layer-3-switch-comparison)
- [Applications](#real-world-applications)
- [Conclusion](#conclusion)
- [Tools Used](#tools-used)
- [References](#references)

---

## Abstract

This project demonstrates **network segmentation using VLANs** (Virtual Local Area Networks) and **Inter-VLAN routing** in a simulated enterprise environment using Cisco Packet Tracer.

Three VLANs — Sales (VLAN 10), Management (VLAN 20), and Admin (VLAN 30) — are created across three Cisco 2960-24TT access switches, each serving two PCs. A **Cisco 3560-24PS Multilayer Switch** acts as the core/distribution layer, connected to a **Cisco ISR4331 Router** at the top.

Inter-VLAN communication is demonstrated using two methods:
1. **Router-on-a-Stick (ROAS)** — ISR4331 Router0 with dot1Q sub-interfaces
2. **Layer 3 Switch SVI** — 3560-24PS Multilayer Switch0 with `ip routing` and SVIs

The simulation verifies successful intra-VLAN and inter-VLAN communication using ping tests, validating correct 802.1Q trunking and IP routing behavior.

---

## Introduction

### What is a VLAN?

A **VLAN (Virtual Local Area Network)** is a logical grouping of network devices, regardless of their physical location, within a single broadcast domain. VLANs are configured on Layer 2 switches using the IEEE **802.1Q** standard.

### Why VLANs?

| Problem (Without VLAN) | Solution (With VLAN) |
|------------------------|----------------------|
| All devices share one broadcast domain | Separate broadcast domains per department |
| Security risk — all traffic visible to all | Traffic isolated between departments |
| High broadcast traffic → congestion | Reduced broadcasts, better performance |
| No logical separation of departments | Logical grouping regardless of physical location |

### Why Inter-VLAN Routing?

VLANs by default **cannot communicate** with each other. In real enterprise networks, departments need controlled access to shared resources. **Inter-VLAN Routing** (a Layer 3 function) allows this controlled communication while maintaining security boundaries.

---

## Network Topology

```
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
```

---

## Device Summary

| Device | Model | Role |
|--------|-------|------|
| Router0 | Cisco ISR4331 | Inter-VLAN routing (ROAS) |
| Multilayer Switch0 | Cisco 3560-24PS | Core switch + L3 SVI routing |
| Switch0 | Cisco 2960-24TT | Access switch — Sales (VLAN 10) |
| Switch1 | Cisco 2960-24TT | Access switch — Mgmt (VLAN 20) |
| Switch2 | Cisco 2960-24TT | Access switch — Admin (VLAN 30) |
| PC0, PC1 | PC-PT | Sales department end devices |
| PC2, PC3 | PC-PT | Management department end devices |
| PC4, PC5 | PC-PT | Admin department end devices |

---

## IP Addressing Table

| Device | Interface | VLAN | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|------|------------|-------------|-----------------|
| Router0 (ISR4331) | Gig0/0/0.10 | 10 | 192.168.10.1 | 255.255.255.0 | — |
| Router0 (ISR4331) | Gig0/0/0.20 | 20 | 192.168.20.1 | 255.255.255.0 | — |
| Router0 (ISR4331) | Gig0/0/0.30 | 30 | 192.168.30.1 | 255.255.255.0 | — |
| Multilayer Switch0 | VLAN 10 SVI | 10 | 192.168.10.1 | 255.255.255.0 | — |
| Multilayer Switch0 | VLAN 20 SVI | 20 | 192.168.20.1 | 255.255.255.0 | — |
| Multilayer Switch0 | VLAN 30 SVI | 30 | 192.168.30.1 | 255.255.255.0 | — |
| PC0 (Sales) | NIC | 10 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC1 (Sales) | NIC | 10 | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 |
| PC2 (Mgmt) | NIC | 20 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |
| PC3 (Mgmt) | NIC | 20 | 192.168.20.11 | 255.255.255.0 | 192.168.20.1 |
| PC4 (Admin) | NIC | 30 | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 |
| PC5 (Admin) | NIC | 30 | 192.168.30.11 | 255.255.255.0 | 192.168.30.1 |

> **Note:** All IP addresses are statically configured on each PC. No DHCP used in this project.

---

## VLAN Design

| VLAN ID | Name | Department | Network | Switch | PCs |
|---------|------|------------|---------|--------|-----|
| VLAN 10 | SALES | Sales Department | 192.168.10.0/24 | Switch0 | PC0, PC1 |
| VLAN 20 | MGMT | Management Department | 192.168.20.0/24 | Switch1 | PC2, PC3 |
| VLAN 30 | ADMIN | Admin Department | 192.168.30.0/24 | Switch2 | PC4, PC5 |
| VLAN 99 | NATIVE | Native Trunk VLAN | — | All trunk links | — |

---

## Configuration — Step by Step

### 1. Access Switches — VLAN Assignment

> Run on **Switch0, Switch1, Switch2** (all three 2960s):

```bash
Switch> enable
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
```

**Switch0 — PC0 and PC1 → VLAN 10 (Sales):**
```bash
Switch0(config)# interface fastEthernet 0/1
Switch0(config-if)# switchport mode access
Switch0(config-if)# switchport access vlan 10
Switch0(config-if)# exit

Switch0(config)# interface fastEthernet 0/2
Switch0(config-if)# switchport mode access
Switch0(config-if)# switchport access vlan 10
Switch0(config-if)# exit
```

**Switch1 — PC2 and PC3 → VLAN 20 (Management):**
```bash
Switch1(config)# interface fastEthernet 0/1
Switch1(config-if)# switchport mode access
Switch1(config-if)# switchport access vlan 20
Switch1(config-if)# exit

Switch1(config)# interface fastEthernet 0/2
Switch1(config-if)# switchport mode access
Switch1(config-if)# switchport access vlan 20
Switch1(config-if)# exit
```

**Switch2 — PC4 and PC5 → VLAN 30 (Admin):**
```bash
Switch2(config)# interface fastEthernet 0/1
Switch2(config-if)# switchport mode access
Switch2(config-if)# switchport access vlan 30
Switch2(config-if)# exit

Switch2(config)# interface fastEthernet 0/2
Switch2(config-if)# switchport mode access
Switch2(config-if)# switchport access vlan 30
Switch2(config-if)# exit
```

**Verify on each switch:**
```bash
Switch# show vlan brief
```

---

### 2. Trunk Port Configuration

**Multilayer Switch0 — Trunk toward Router0:**
```bash
MLS0(config)# interface gigabitEthernet 0/1
MLS0(config-if)# switchport trunk encapsulation dot1q
MLS0(config-if)# switchport mode trunk
MLS0(config-if)# switchport trunk native vlan 99
MLS0(config-if)# switchport trunk allowed vlan 10,20,30,99
MLS0(config-if)# exit
```

**Multilayer Switch0 — Trunk toward each 2960 (Switch0, Switch1, Switch2):**
```bash
MLS0(config)# interface gigabitEthernet 0/2
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
```

**Switch0 / Switch1 / Switch2 — Uplink trunk toward Multilayer Switch0:**
```bash
Switch(config)# interface gigabitEthernet 0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30,99
Switch(config-if)# exit
```

**Verify:**
```bash
MLS0# show interfaces trunk
```

---

### 3. Router-on-a-Stick (ROAS) — ISR4331 Router0

> The ISR4331 uses **GigabitEthernet0/0/0** (not Fa0/0):

```bash
Router0> enable
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
```

**Verify:**
```bash
Router0# show ip route
Router0# show ip interface brief
Router0# show interfaces gigabitEthernet 0/0/0.10
```

---

### 4. Layer 3 Switch SVI Method — Multilayer Switch0

> The **3560-24PS** supports Layer 3 routing using SVIs:

```bash
MLS0> enable
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
```

**Verify:**
```bash
MLS0# show ip route
MLS0# show interfaces vlan 10
MLS0# show interfaces vlan 20
MLS0# show interfaces vlan 30
```

---

## Verification Commands

```bash
! ── ACCESS SWITCH ────────────────────────────────────────────
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
```

---

## Results — Ping Test

| Source | Source IP | Destination | Dest IP | Path | Result |
|--------|-----------|-------------|---------|------|--------|
| PC0 (Sales) | 192.168.10.10 | PC1 (Sales) | 192.168.10.11 | VLAN 10 → VLAN 10 | ✅ Pass |
| PC0 (Sales) | 192.168.10.10 | PC2 (Mgmt) | 192.168.20.10 | VLAN 10 → VLAN 20 | ✅ Pass |
| PC0 (Sales) | 192.168.10.10 | PC4 (Admin) | 192.168.30.10 | VLAN 10 → VLAN 30 | ✅ Pass |
| PC2 (Mgmt) | 192.168.20.10 | PC4 (Admin) | 192.168.30.10 | VLAN 20 → VLAN 30 | ✅ Pass |
| PC0 (Sales) | 192.168.10.10 | PC2 (Mgmt) | 192.168.20.10 | No routing | ❌ Fail (expected) |

> Cross-VLAN ping **fails without routing configured** — confirming VLAN isolation works correctly before routing is enabled.

---

## ROAS vs Layer 3 Switch Comparison

| Feature | Router-on-a-Stick (ISR4331) | Layer 3 Switch (3560-24PS) |
|---------|----------------------------|---------------------------|
| **Hardware** | Separate router + switch | Single multilayer switch |
| **Cost** | Higher (two devices) | Lower (one device) |
| **Performance** | Router is single bottleneck | Wire-speed hardware routing |
| **Scalability** | Limited sub-interfaces | Easily scalable SVIs |
| **Key command** | `encapsulation dot1Q` | `ip routing` + SVI |
| **Latency** | Slightly higher | Lower |
| **Best for** | Small networks / learning labs | Enterprise networks |
| **Used in this project** | ✅ Yes — ISR4331 Router0 | ✅ Yes — 3560-24PS MLS0 |

---

## Real-World Applications

1. **Enterprise offices** — Sales, Management, Admin VLANs on shared physical infrastructure with controlled inter-department access
2. **University campuses** — Student, Faculty, Lab, and Admin VLANs logically separated across shared switches
3. **Data centres** — Application, Storage, and Management traffic on isolated VLANs
4. **Healthcare** — Medical device VLANs strictly separated from general staff VLANs for compliance
5. **Hotels / Airports** — Guest Wi-Fi VLAN isolated from backend operations network

---

## Conclusion

This project successfully demonstrates:

- ✅ VLAN creation and port assignment — VLAN 10 (SALES), VLAN 20 (MGMT), VLAN 30 (ADMIN)
- ✅ Three 2960-24TT access switches each serving two PCs in dedicated VLANs
- ✅ 802.1Q trunk links between all switches and the Multilayer Switch0 core
- ✅ Inter-VLAN routing using **Router-on-a-Stick** on the Cisco ISR4331 Router0
- ✅ Inter-VLAN routing using **Layer 3 SVI** on the Cisco 3560-24PS Multilayer Switch0
- ✅ Static IP addressing for all 6 PCs (PC0–PC5)
- ✅ Successful cross-VLAN ping verified between all three departments

VLANs enable security, broadcast control, and logical segmentation without extra physical hardware. Combined with Inter-VLAN routing, they form the backbone of modern enterprise network design.

---

## Project Structure

```
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
```

---

## Tools Used

| Tool | Model / Version | Purpose |
|------|----------------|---------|
| Cisco Packet Tracer | 8.2+ | Network simulation |
| Cisco ISR4331 | IOS 16.x | Router — ROAS Inter-VLAN routing |
| Cisco 3560-24PS | IOS 15.x | Multilayer Switch — SVI routing + core |
| Cisco 2960-24TT | IOS 15.x | Access switches — VLAN port assignment |

---

## References

1. Cisco Systems. *VLAN Configuration Guide — Cisco IOS.* https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960/software/release/15_0_2_se/configuration/guide/scg2960/swvlan.html
2. Odom, W. (2020). *CCNA 200-301 Official Cert Guide, Volume 1.* Cisco Press.
3. IEEE Standard 802.1Q — *Bridges and Bridged Networks.* IEEE Standards Association.
4. Lammle, T. (2020). *CCNA Cisco Certified Network Associate Study Guide.* Sybex.
5. Cisco Networking Academy. *CCNA: Switching, Routing, and Wireless Essentials.* https://www.netacad.com

