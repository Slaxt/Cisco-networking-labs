# CCNA Switching, Routing & Wireless Essentials — Lab Portfolio
### Alexander Duong | Collin College | ITCC 1344

> Completed lab series from the Cisco CCNA Switching, Routing and Wireless Essentials (SRWE) course. All labs were built and configured in **Cisco Packet Tracer** with 100% completion scores. Labs cover enterprise switching, inter-VLAN routing, Layer 3 switching, EtherChannel, and network troubleshooting.

---

## Lab Index

| # | Lab | Key Topics | Score |
|---|-----|-----------|-------|
| 1 | [Configure SSH](#lab-1--configure-ssh) | SSH hardening, password encryption, VTY security | 100% |
| 2 | [Configure Router Interfaces](#lab-2--configure-router-interfaces) | IPv4/IPv6 dual-stack, serial & gigabit interfaces | 100% |
| 3 | [Verify Directly Connected Networks](#lab-3--verify-directly-connected-networks) | show commands, IPv4/IPv6 verification, troubleshooting | 100% |
| 4 | [Implement a Small Network](#lab-4--implement-a-small-network) | Full build from scratch, device naming, cabling, basic config | 100% |
| 5 | [VLAN Configuration](#lab-5--vlan-configuration) | VLAN creation, port assignment, voice VLAN, QoS | 100% |
| 6 | [Configure Trunks](#lab-6--configure-trunks) | 802.1Q trunking, native VLAN, trunk verification | 100% |
| 7 | [Configure DTP](#lab-7--configure-dtp) | Static vs dynamic trunking, DTP modes, VLAN 999 native | 100% |
| 8 | [Router-on-a-Stick Inter-VLAN Routing](#lab-8--router-on-a-stick-inter-vlan-routing) | Subinterfaces, 802.1Q encapsulation, inter-VLAN routing | 100% |
| 9 | [Layer 3 Switching & Inter-VLAN Routing](#lab-9--layer-3-switching--inter-vlan-routing) | MLS, SVIs, ip routing, IPv6 inter-VLAN, dual-stack | 100% |
| 10 | [Troubleshoot Inter-VLAN Routing](#lab-10--troubleshoot-inter-vlan-routing) | Fault isolation, show commands, fix & verify | 100% |
| 11 | [Configure EtherChannel](#lab-11--configure-etherchannel) | PAgP, LACP (802.3ad), port-channel, STP interaction | 100% |

---

## Lab 1 – Configure SSH

**Topology:** PC1 → S1 (single switch, management access lab)

**Addressing:**
| Device | Interface | IP Address | Subnet Mask |
|--------|-----------|-----------|-------------|
| S1 | VLAN 1 | 10.10.10.2 | 255.255.255.0 |
| PC1 | NIC | 10.10.10.10 | 255.255.255.0 |

**What I did:**
- Replaced insecure Telnet with SSH for encrypted remote management of switch S1
- Enabled `service password-encryption` to protect plain-text passwords in the running config
- Set the IP domain name to `netacad.pka` and generated 1024-bit RSA keys
- Created a local administrator account and reconfigured VTY lines to accept SSH only, removing Telnet access
- Verified that Telnet connections were refused and SSH login succeeded from PC1

**Key commands:**
```
S1(config)# service password-encryption
S1(config)# ip domain-name netacad.pka
S1(config)# crypto key generate rsa modulus 1024
S1(config)# username administrator secret cisco
S1(config)# line vty 0 15
S1(config-line)# transport input ssh
S1(config-line)# login local
```

**Security relevance:** Disabling Telnet and enforcing SSH is a baseline hardening control required by CIS Controls, NIST 800-53 (IA-2), and most compliance frameworks. This lab directly mirrors real-world network device hardening procedures.

---

## Lab 2 – Configure Router Interfaces

**Topology:** Dual-router, dual-switch topology with a Dual Stack Server — 4 PCs, 4 switches (SW1–SW4), 2 routers (R1, R2), Internet cloud

**Addressing (selected):**
| Device | Interface | IP Address |
|--------|-----------|-----------|
| R1 | G0/0 | 172.16.20.1/25 |
| R1 | G0/1 | 172.16.20.129/25 |
| R1 | S0/0/0 | 209.165.200.225/30 |
| R2 | G0/0 | 2001:db8:c0de:12::1/64 |
| R2 | G0/1 | 2001:db8:c0de:13::1/64 |
| R2 | S0/0/1 | 2001:db8:c0de:11::2/64 |
| Dual Stack Server | — | 64.100.1.10 / 2001:db8:100:1::a |

**What I did:**
- Configured IPv4 addressing on R1 and all LAN devices (PC1, PC2) across two subnets using /25 VLSM
- Configured IPv6 addressing on R2 and LAN devices (PC3, PC4) using /64 prefixes with EUI-64
- Verified end-to-end IPv4 and IPv6 connectivity to the Dual Stack Server using ping

**Security relevance:** Understanding dual-stack environments is essential for modern network security — many organizations run both IPv4 and IPv6, and misconfigurations in either stack create attack surfaces.

---

## Lab 3 – Verify Directly Connected Networks

**Topology:** Same dual-router, dual-switch topology as Lab 2

**What I did:**
- Used `show ip interface brief | exclude unassigned` to audit interface status and IP assignments on R1
- Filtered routing tables with `show ip route | begin Gate` to identify the gateway of last resort
- Used `show interfaces | include Description` and `show interfaces g0/0 | include duplex` to inspect interface details
- Corrected IPv6 address misconfigurations on R2 using `no ipv6 address` and re-assigning correct /64 prefixes
- Verified full IPv4 and IPv6 connectivity from all PCs to the Dual Stack Server

**Key verification commands:**
```
R1# show ip interface brief | exclude unassigned
R1# show ip route | begin Gate
R2# show ipv6 interface brief
R2(config-if)# no ipv6 address 2001:db8:c0de:14::1/64
R2(config-if)# ipv6 address 2001:db8:c0de:13::1/64
```

**Security relevance:** Network verification and troubleshooting skills are foundational to both incident response and compliance audits — correctly reading interface and routing output is used daily by SOC and network security teams.

---

## Lab 4 – Implement a Small Network

**Topology built from scratch:** RTA → SW1 + SW2 → PC-1 + PC-2

**Addressing:**
| Device | Interface | Address |
|--------|-----------|---------|
| RTA | G0/0 | 10.10.10.1 / 255.255.255.0 |
| RTA | G0/1 | 10.10.20.1 / 255.255.255.0 |
| SW1 | VLAN 1 | 10.10.10.2 |
| SW2 | VLAN 1 | 10.10.20.2 |

**Cable connections:**
| From | Port | To | Port |
|------|------|----|------|
| RTA | G0/0 | SW1 | G0/1 |
| RTA | G0/1 | SW2 | G0/1 |
| SW1 | F0/1 | PC-1 | FastEthernet0 |
| SW2 | F0/1 | PC-2 | FastEthernet0 |

**What I did:**
- Built the entire topology from scratch by dragging and connecting devices in Packet Tracer
- Configured RTA with hostname, encrypted enable password (`Ciscoenpa55`), console password (`Ciscolinepa55`), MOTD banner, and interface descriptions
- Configured management IPs and default gateways on SW1 and SW2
- Assigned IP addresses to PC-1 and PC-2 and verified end-to-end connectivity with ping

**Security relevance:** This lab simulates standing up a new network from scratch — the same process used when deploying a new office segment or isolated security zone. Proper initial configuration (hostnames, passwords, banners) is required by compliance baselines.

---

## Lab 5 – VLAN Configuration

**Topology:** 3 switches (S1, S2, S3), 6 PCs, 1 IP Phone

**Addressing:**
| Device | IP Address | VLAN |
|--------|-----------|------|
| PC1 | 172.17.10.21 | VLAN 10 – Faculty/Staff |
| PC2 | 172.17.20.22 | VLAN 20 – Students |
| PC3 | 172.17.30.23 | VLAN 30 – Guest/Default |
| PC4 | 172.17.10.24 | VLAN 10 – Faculty/Staff |
| PC5 | 172.17.20.25 | VLAN 20 – Students |
| PC6 | 172.17.30.26 | VLAN 30 – Guest/Default |

**VLANs configured:**
| VLAN | Name |
|------|------|
| 10 | Faculty/Staff |
| 20 | Students |
| 30 | Guest/Default |
| 99 | Management/Native |
| 150 | VOICE |

**What I did:**
- Created and named VLANs 10, 20, 30, 99 (Management), and 150 (VOICE) on switches S1, S2, and S3
- Assigned access ports on S2 and S3 to the correct VLANs (Fa0/11 → VLAN 10, Fa0/18 → VLAN 20, Fa0/6 → VLAN 30)
- Configured the S3 Fa0/11 port with voice VLAN 150 and `mls qos trust cos` for the IP Phone
- Verified VLAN assignments using `show vlan brief` and confirmed intra-VLAN connectivity

**Key commands:**
```
S1(config)# vlan 10
S1(config-vlan)# name Faculty/Staff
S2(config)# interface fa0/11
S2(config-if)# switchport mode access
S2(config-if)# switchport access vlan 10
S3(config-if)# switchport voice vlan 150
S3(config-if)# mls qos trust cos
```

**Security relevance:** VLAN segmentation is one of the most fundamental network security controls — it enforces logical separation between user groups, limits broadcast domains, and contains lateral movement in the event of a breach.

---

## Lab 6 – Configure Trunks

**Topology:** 3 switches (S1, S2, S3), 6 PCs

**Addressing:** Same as Lab 5 (172.17.x.x /24 per VLAN)

**What I did:**
- Configured G0/1 and G0/2 on S1 as 802.1Q trunk ports
- Set native VLAN to 99 on S1 trunk interfaces: `switchport trunk native vlan 99`
- Observed and resolved native VLAN mismatch syslog messages between S1 (VLAN 99) and S2/S3 (VLAN 1)
- Corrected native VLAN on S2 and S3 to match VLAN 99 using `show interface trunk` for verification
- Verified which VLANs were allowed across trunks and confirmed end-to-end connectivity

**Key commands:**
```
S1(config)# interface range g0/1 - 2
S1(config-if-range)# switchport mode trunk
S1(config-if-range)# switchport trunk native vlan 99
```

**Security relevance:** Native VLAN mismatches are a known attack vector (VLAN hopping). Correctly configuring and auditing native VLANs on trunk ports is a CIS benchmark requirement and a common finding in network security assessments.

---

## Lab 7 – Configure DTP

**Topology:** 3 switches (S1, S2, S3), 6 PCs

**Addressing:**
| Device | Interface | IP Address | VLAN |
|--------|-----------|-----------|------|
| PC1 | NIC | 192.168.10.1 | — |
| S1 | VLAN 99 | 192.168.99.1 | Management |
| S2 | VLAN 99 | 192.168.99.2 | Management |
| S3 | VLAN 99 | 192.168.99.3 | Management |

**VLANs on S2 and S3:**
| VLAN | Name | Ports |
|------|------|-------|
| 10 | Red | F0/1–8 |
| 20 | Blue | F0/9–16 |
| 30 | Yellow | F0/17–24 |

**What I did:**
- Verified existing VLAN configuration across all three switches using `show vlan brief`
- Created VLANs 10 (Red), 20 (Blue), 30 (Yellow) on S2 and S3 and assigned port ranges to each VLAN
- Configured S1 G0/1 as `switchport mode dynamic desirable` (DTP active negotiation toward S2)
- Configured S1 G0/2 as a static trunk with `switchport nonegotiate` to disable DTP toward S3
- Set native VLAN 999 on all trunk links across S1, S2, and S3
- Verified DTP status with `show dtp` and trunk status with `show interfaces trunk`
- Resolved S3 trunk mismatch by reconfiguring G0/2 to match S1

**Security relevance:** DTP is a security risk — leaving ports in dynamic mode allows an attacker to negotiate a trunk and access all VLANs (VLAN hopping attack). Best practice is `switchport nonegotiate` on all access and trunk ports, which this lab demonstrates.

---

## Lab 8 – Router-on-a-Stick Inter-VLAN Routing

**Topology:** R1 → S1 → PC1 (VLAN 10) + PC3 (VLAN 30)

**Addressing:**
| Device | Interface | IP Address | Subnet |
|--------|-----------|-----------|--------|
| R1 | G0/0.10 | 172.17.10.1 | 255.255.255.0 |
| R1 | G0/0.30 | 172.17.30.1 | 255.255.255.0 |
| PC1 | NIC | 172.17.10.10 | VLAN 10 |
| PC3 | NIC | 172.17.30.10 | VLAN 30 |

**What I did:**
- Created VLANs 10 and 30 on S1 and assigned access ports (Fa0/6 → VLAN 30, Fa0/11 → VLAN 10)
- Verified that PC1 could not ping PC3 before inter-VLAN routing was configured (expected failure)
- Created subinterfaces G0/0.10 and G0/0.30 on R1 with 802.1Q encapsulation and correct IP gateways
- Enabled trunking on S1 G0/1 (the port connected to R1) to allow tagged VLAN traffic
- Verified inter-VLAN routing with successful pings between PC1 (VLAN 10) and PC3 (VLAN 30)

**Key commands:**
```
R1(config)# interface g0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 172.17.10.1 255.255.255.0
R1(config)# interface g0/0.30
R1(config-subif)# encapsulation dot1Q 30
R1(config-subif)# ip address 172.17.30.1 255.255.255.0
```

**Security relevance:** Router-on-a-stick is widely deployed in small-to-medium enterprise environments. Understanding subinterface routing is essential for auditing and securing inter-VLAN traffic flows — a key task in network security assessments.

---

## Lab 9 – Layer 3 Switching & Inter-VLAN Routing

**Topology:** MLS (Cisco Catalyst 3650 multilayer switch) → S1, S2, S3 → 6 PCs + Cloud

**Addressing:**
| Device | Interface | IPv4 | IPv6 |
|--------|-----------|------|------|
| MLS | VLAN 10 (Staff) | 192.168.10.254/24 | 2001:db8:acad:10::1/64 |
| MLS | VLAN 20 (Student) | 192.168.20.254/24 | 2001:db8:acad:20::1/64 |
| MLS | VLAN 30 (Faculty) | 192.168.30.254/24 | 2001:db8:acad:30::1/64 |
| MLS | VLAN 99 (Mgmt) | 192.168.99.254/24 | — |
| MLS | G0/2 (routed) | 209.165.200.225/30 | 2001:db8:acad:a::1/64 |

**VLANs:**
| VLAN | Name |
|------|------|
| 10 | Staff |
| 20 | Student |
| 30 | Faculty |
| 99 | Management/Native |

**What I did:**
- Configured MLS G0/2 as a Layer 3 routed port (`no switchport`) with uplink to Cloud
- Created VLANs 10, 20, 30, and 99 on MLS and configured SVIs with IPv4 and IPv6 addresses
- Enabled `ip routing` on MLS to activate Layer 3 inter-VLAN routing via SVIs
- Configured trunking on MLS G0/1 toward S1 with native VLAN 99 and dot1q encapsulation
- Enabled `ipv6 unicast-routing` for IPv6 inter-VLAN routing and verified with `show ipv6 route`
- Verified full dual-stack end-to-end connectivity from all PCs across VLANs to Cloud

**Security relevance:** Layer 3 switches are the backbone of modern enterprise networks. SVIs combined with ACLs allow security teams to enforce traffic policies between departments at line rate — a key tool in compliance-driven network segmentation.

---

## Lab 10 – Troubleshoot Inter-VLAN Routing

**Topology:** R1 → S1 → PC1 (VLAN 10) + PC3 (VLAN 30)

**Addressing:**
| Device | Interface | IP Address | VLAN |
|--------|-----------|-----------|------|
| R1 | G0/1.10 | 172.17.10.1 | VLAN 10 |
| R1 | G0/1.30 | 172.17.30.1 | VLAN 30 |
| PC1 | NIC | 172.17.10.10 | VLAN 10 |
| PC3 | NIC | 172.17.30.10 | VLAN 30 |

**What I did:**
- Received a pre-configured broken topology with multiple inter-VLAN routing faults
- Used `show ip interface brief`, `show interface g0/1.10`, `show interface g0/1.30`, and `show interface trunk` to systematically locate all misconfigurations
- Documented each problem and its solution in the lab's Documentation Table
- Implemented fixes and verified full connectivity between PC1 and PC3 and back to R1

**Security relevance:** Troubleshooting is a core SOC skill. This lab demonstrates the ability to methodically isolate faults using CLI verification commands — the same approach used when investigating network-based alerts or misconfigured security zones.

---

## Lab 11 – Configure EtherChannel

**Topology:** 3 switches (S1, S2, S3) with redundant uplinks forming a triangle

**Port Channel Configuration:**
| Channel Group | Switches | Ports | Protocol |
|--------------|----------|-------|----------|
| Port Channel 1 | S1 ↔ S3 | F0/21, F0/22 | PAgP (desirable) |
| Port Channel 2 | S1 ↔ S2 | G0/1, G0/2 | LACP (active) |
| Port Channel 3 | S2 ↔ S3 | F0/23, F0/24 | LACP (active/passive) |

**What I did:**
- Shut down interfaces before bundling to avoid EtherChannel misconfiguration errors
- Configured Port Channel 1 using Cisco PAgP with `channel-group 1 mode desirable` on S1 and S3
- Configured Port Channel 2 using IEEE 802.3ad LACP with `channel-group 2 mode active` on S1 and S2
- Configured Port Channel 3 as negotiated LACP with active on S2 and passive on S3
- Set all port-channels as trunk interfaces with `interface port-channel X / switchport mode trunk`
- Verified EtherChannel status using `show etherchannel summary` — confirmed all channels in P (in port-channel) state
- Observed STP interaction: Port Channel 2 Gigabit ports were placed in blocking state by STP; resolved by setting S1 as root primary with `spanning-tree vlan 1 root primary`

**Key commands:**
```
S1(config)# interface range f0/21 - 22
S1(config-if-range)# shutdown
S1(config-if-range)# channel-group 1 mode desirable
S1(config-if-range)# no shutdown
S1(config)# interface range g0/1 - 2
S1(config-if-range)# channel-group 2 mode active
S1(config)# interface port-channel 1
S1(config-if)# switchport mode trunk
S1# show etherchannel summary
S1# show spanning-tree active
```

**Security relevance:** EtherChannel provides link redundancy and increased bandwidth for critical infrastructure — essential for ensuring availability (CIA triad) of security appliances, SIEM platforms, and core network devices.

---

## Skills Demonstrated

| Skill | Labs |
|-------|------|
| VLAN creation, naming, and port assignment | 5, 6, 7, 8, 9 |
| 802.1Q trunking and native VLAN hardening | 6, 7, 8, 9 |
| DTP configuration and disabling | 7 |
| Router-on-a-stick subinterface routing | 8 |
| Layer 3 switching with SVIs | 9 |
| Dual-stack IPv4/IPv6 configuration | 2, 3, 9 |
| EtherChannel (PAgP and LACP) | 11 |
| Spanning Tree Protocol and root bridge manipulation | 11 |
| SSH hardening and Telnet removal | 1 |
| Network troubleshooting with show commands | 3, 10 |
| Full network build from scratch | 4 |

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Cisco Packet Tracer | Network simulation environment |
| Cisco IOS (2960/3650/1941) | Switch and router configuration |
| Windows end hosts | Connectivity testing (ping, ipconfig) |

---

## About

**Alexander Duong**
AAS Cybersecurity — Collin College (2025)
CompTIA Security+ (SY0-701) — In Progress
Certificates: Information Systems Cybersecurity · Cybersecurity Infrastructure Technician · Entry-Level Network Support · Information Systems Cybersecurity (10-req track)

📧 alexander.duong4@gmail.com | 🔗 linkedin.com/in/alexanderduong
