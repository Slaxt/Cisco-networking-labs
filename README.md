# Cisco Networking Labs – Alexander Duong

> Hands-on network engineering labs completed using **Cisco Packet Tracer** as part of my AAS in Cybersecurity at Collin College. Labs cover enterprise networking concepts from VLAN segmentation to redundancy protocols and IPv6 — skills directly applicable to network security and infrastructure roles.

---

## 📋 Table of Contents
- [Lab Overview](#lab-overview)
- [Lab 1 – VLANs & Trunking](#lab-1--vlans--trunking)
- [Lab 2 – Router-on-a-Stick / Inter-VLAN Routing](#lab-2--router-on-a-stick--inter-vlan-routing)
- [Lab 3 – EtherChannel (LACP)](#lab-3--etherchannel-lacp)
- [Lab 4 – Spanning Tree Protocol & PortFast](#lab-4--spanning-tree-protocol--portfast)
- [Lab 5 – OSPF Routing](#lab-5--ospf-routing)
- [Lab 6 – HSRP / VRRP / GLBP (FHRP)](#lab-6--hsrp--vrrp--glbp-fhrp)
- [Lab 7 – DHCP Relay](#lab-7--dhcp-relay)
- [Lab 8 – SSH & Port Security](#lab-8--ssh--port-security)
- [Lab 9 – VLSM Subnetting](#lab-9--vlsm-subnetting)
- [Lab 10 – IPv6 Addressing](#lab-10--ipv6-addressing)
- [Capstone – Enterprise Network Topology](#capstone--enterprise-network-topology)
- [Tools Used](#tools-used)

---

## Lab Overview

| Lab | Topic | Key Concepts |
|-----|-------|-------------|
| 1 | VLANs & Trunking | VLAN creation, 802.1Q, DTP, trunk links |
| 2 | Router-on-a-Stick | Subinterfaces, inter-VLAN routing, encapsulation |
| 3 | EtherChannel | LACP, port-channel bundling, load balancing |
| 4 | STP & PortFast | Root bridge election, port states, PortFast/BPDU Guard |
| 5 | OSPF | Single-area OSPF, DR/BDR election, route advertisement |
| 6 | FHRP | HSRP, VRRP, GLBP, active/standby failover |
| 7 | DHCP Relay | ip helper-address, relay across VLANs |
| 8 | SSH & Port Security | Secure remote access, MAC address limiting, violation modes |
| 9 | VLSM Subnetting | Variable-length subnet masking, IP address planning |
| 10 | IPv6 | Static IPv6, EUI-64, link-local addresses |
| Capstone | Full Enterprise Topology | Combined all concepts into a multi-site network |

---

## Lab 1 – VLANs & Trunking

**Objective:** Segment a network into multiple VLANs and configure trunk links between switches.

**What I did:**
- Created VLANs (e.g., VLAN 10 for Sales, VLAN 20 for IT, VLAN 30 for Management) on Cisco IOS switches
- Assigned access ports to VLANs and configured trunk ports using 802.1Q encapsulation
- Verified trunk links and VLAN propagation using `show vlan brief` and `show interfaces trunk`

**Key commands:**
```
vlan 10
 name SALES
interface fa0/1
 switchport mode access
 switchport access vlan 10
interface fa0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

**Why it matters for security:** VLAN segmentation is a foundational network security control — isolating traffic between departments limits the blast radius of a breach and enforces least-privilege network access.

---

## Lab 2 – Router-on-a-Stick / Inter-VLAN Routing

**Objective:** Enable communication between VLANs using a single router interface with subinterfaces.

**What I did:**
- Configured subinterfaces on a router with 802.1Q encapsulation for each VLAN
- Set up a trunk link between the switch and router
- Verified inter-VLAN routing with ping tests across VLAN boundaries

**Key commands:**
```
interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

**Why it matters for security:** Understanding how VLANs route traffic is essential for designing secure network zones (e.g., separating guest Wi-Fi from internal systems).

---

## Lab 3 – EtherChannel (LACP)

**Objective:** Bundle multiple physical links into one logical channel for redundancy and increased bandwidth.

**What I did:**
- Configured EtherChannel using LACP (802.3ad) between two switches
- Verified port-channel formation and load balancing
- Tested failover behavior when one link went down

**Key commands:**
```
interface range fa0/1-2
 channel-group 1 mode active
interface port-channel 1
 switchport mode trunk
```

**Why it matters for security:** Link redundancy ensures availability — a core pillar of the CIA triad — preventing network outages from becoming security incidents.

---

## Lab 4 – Spanning Tree Protocol & PortFast

**Objective:** Prevent Layer 2 loops and optimize end-device port convergence.

**What I did:**
- Observed STP root bridge election based on bridge priority and MAC address
- Manipulated root bridge selection by adjusting priority values
- Enabled PortFast and BPDU Guard on access ports to speed up host connectivity and prevent rogue switch attacks

**Key commands:**
```
spanning-tree vlan 10 priority 4096
interface fa0/1
 spanning-tree portfast
 spanning-tree bpduguard enable
```

**Why it matters for security:** BPDU Guard prevents unauthorized switches from being plugged into the network and taking over as root bridge — a real attack vector.

---

## Lab 5 – OSPF Routing

**Objective:** Configure dynamic routing between routers using OSPF in a single area.

**What I did:**
- Enabled OSPF on multiple routers and advertised networks into Area 0
- Observed DR/BDR election on multi-access networks
- Verified routing tables with `show ip route ospf`

**Key commands:**
```
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
```

**Why it matters for security:** Understanding routing protocols is essential for identifying misconfigurations that could expose internal networks or allow traffic to leak between security zones.

---

## Lab 6 – HSRP / VRRP / GLBP (FHRP)

**Objective:** Provide gateway redundancy so end devices maintain connectivity if a router fails.

**What I did:**
- Configured HSRP with active/standby router roles and a virtual IP gateway
- Compared VRRP (open standard) and GLBP (load balancing) behavior
- Simulated failover by shutting down the active router and verifying the standby took over

**Key commands:**
```
interface g0/0
 standby 1 ip 192.168.10.254
 standby 1 priority 110
 standby 1 preempt
```

**Why it matters for security:** High availability configurations are critical in security infrastructure — firewalls, IDS appliances, and gateways all need redundancy to avoid single points of failure.

---

## Lab 7 – DHCP Relay

**Objective:** Allow hosts on remote VLANs to receive IP addresses from a centralized DHCP server.

**What I did:**
- Configured a DHCP server on one subnet
- Used `ip helper-address` on router interfaces to forward DHCP requests across VLAN boundaries
- Verified hosts on remote VLANs received correct IP addressing

**Key commands:**
```
interface g0/0.20
 ip helper-address 192.168.1.10
```

**Why it matters for security:** Rogue DHCP servers are a common attack vector. Understanding how DHCP relay works is the foundation for implementing DHCP snooping as a security control.

---

## Lab 8 – SSH & Port Security

**Objective:** Secure remote device management and restrict unauthorized devices from connecting to switch ports.

**What I did:**
- Replaced Telnet with SSH for encrypted remote access to Cisco devices
- Generated RSA keys, configured domain name, and created local user accounts
- Configured port security to limit the number of MAC addresses per port and set violation modes (shutdown, restrict, protect)

**Key commands:**
```
ip domain-name lab.local
crypto key generate rsa modulus 2048
ip ssh version 2
line vty 0 4
 transport input ssh
 login local

interface fa0/1
 switchport port-security maximum 2
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
```

**Why it matters for security:** This lab is directly security-focused — SSH hardening and port security are standard controls in any network security policy and compliance framework (NIST, CIS Controls).

---

## Lab 9 – VLSM Subnetting

**Objective:** Efficiently allocate IP address space using Variable Length Subnet Masking.

**What I did:**
- Received a base network block and divided it into subnets of varying sizes based on host requirements
- Calculated network address, broadcast address, usable host range, and subnet mask for each subnet
- Applied subnets to a routed topology and verified end-to-end connectivity

**Example:**
```
Base network: 192.168.100.0/24

Subnet A (60 hosts) → 192.168.100.0/26   (62 usable)
Subnet B (30 hosts) → 192.168.100.64/27  (30 usable)
Subnet C (10 hosts) → 192.168.100.96/28  (14 usable)
Subnet D (2 hosts)  → 192.168.100.112/30 (2 usable)
```

**Why it matters for security:** Proper subnetting enforces network segmentation and minimizes broadcast domains — reducing attack surface and containing lateral movement.

---

## Lab 10 – IPv6 Addressing

**Objective:** Configure and verify IPv6 addressing on routers and end devices.

**What I did:**
- Assigned static IPv6 addresses and configured EUI-64 to auto-generate interface IDs from MAC addresses
- Identified link-local addresses and verified neighbor discovery
- Tested IPv6 connectivity with `ping ipv6`

**Key commands:**
```
interface g0/0
 ipv6 address 2001:db8:acad:1::1/64
 ipv6 address fe80::1 link-local
 ipv6 enable
```

**Why it matters for security:** IPv6 adoption is growing and many organizations run dual-stack. Security professionals need to understand IPv6 to audit and secure modern networks.

---

## Capstone – Enterprise Network Topology

**Objective:** Combine all lab concepts into a single cohesive multi-site enterprise network.

**What I built:**
- Multi-switch campus topology with full VLAN segmentation (Sales, IT, HR, Management VLANs)
- Inter-VLAN routing via router-on-a-stick and Layer 3 SVIs
- EtherChannel uplinks between access and distribution switches
- STP with tuned root bridge priority and PortFast/BPDU Guard on all access ports
- OSPF routing between campus and remote site routers
- HSRP for gateway redundancy on both sites
- DHCP relay forwarding requests to a centralized server
- SSH-only management access with port security on all access ports
- VLSM-based IP addressing scheme across all VLANs and WAN links
- IPv6 addressing configured alongside IPv4 (dual-stack)

**Why it matters:** This topology mirrors what you'd find in a real small-to-medium enterprise environment and demonstrates the ability to plan, build, and troubleshoot a complete network infrastructure.

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Cisco Packet Tracer | Network simulation and lab environment |
| Cisco IOS | Router and switch configuration |
| Windows / Linux hosts | End-device testing and connectivity verification |

---

## About Me

AAS in Cybersecurity — Collin College (2025)
CompTIA Security+ (SY0-701) — In Progress

Certificates: Information Systems Cybersecurity Certificate · Cybersecurity Infrastructure Technician Certificate · Entry-Level Network Support Occupational Skill Award · Information Systems Cybersecurity

📧 alexander.duong4@gmail.com
🔗 linkedin.com/in/alexanderduong
