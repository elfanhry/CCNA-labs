# Enterprise Network Simulation — Cisco Packet Tracer

> A full-scale enterprise network simulation designed and configured from scratch over 3 days,
> covering multi-area OSPF, Layer 2 redundancy, HSRP gateway failover, centralized services,
> and hierarchical network design.

**Built with:** Cisco Packet Tracer | **Level:** CCNA Preparation | **Time:** ~3 days

---

## Repository Files

| File | Description |
|---|---|
| `Enterprise-Network-Simulationpkt.pkt` | Cisco Packet Tracer project file — open to explore the full topology |
| `Enterprise-Network-Topology.jpg` | Full network topology diagram |
| `Hasan_Elfanhry_Enterprise_Network_Project.pdf` | Full project documentation with screenshots and verification |

---

## Network Topology

![Full Topology](Enterprise-Network-Topology.jpg)

---

## Project Overview

This project simulates a real enterprise network built entirely from scratch in Cisco Packet Tracer.
The design covers a multi-building company with centralized services, full redundancy, load balancing,
and fault tolerance at every layer of the network stack.

The network is divided into 4 OSPF areas: a backbone core (Area 0), two company buildings
(Areas 1 and 2), and a dedicated server farm (Area 3). Every design decision was made to
reflect real enterprise practices — not just to make the lab work.

---

## Area 0 — Backbone (Core)

| Feature | Detail |
|---|---|
| Devices | 3x Layer 3 Switches |
| Routing Protocol | OSPF Area 0 |
| Uplinks | Dual interfaces per area switch |
| Redundancy | EtherChannel (LACP) on all uplinks |

Each area connects to the backbone via 2 physical interfaces bundled into an EtherChannel.
If one interface fails, traffic continues on the remaining member without interruption.
The /30 point-to-point links are organized by area (10.0.1.x, 10.0.2.x, 10.0.3.x).

---

## Area 1 — Building 1 | `192.168.0.0/16`
## Area 2 — Building 2 | `172.31.0.0/16`

Areas 1 and 2 represent two physical buildings of the same company. Both share identical
VLAN structure and design, differing only in IP address ranges.

### VLAN Design

| VLAN ID | Name | Area 1 Subnet | Area 2 Subnet |
|---|---|---|---|
| 10 | HR | 192.168.10.0/24 | 172.31.10.0/24 |
| 20 | IT | 192.168.20.0/24 | 172.31.20.0/24 |
| 30 | Management | 192.168.30.0/24 | 172.31.30.0/24 |
| 40 | Finance | 192.168.40.0/24 | 172.31.40.0/24 |
| 50 | Employees | 192.168.50.0/24 | 172.31.50.0/24 |
| 60 | Printers | 192.168.60.0/24 | 172.31.60.0/24 |
| 200 | Native VLAN | — | — |

### Switch Architecture

Each area has:
- 2x Layer 3 Core Switches (C1 & C2) — inter-VLAN routing via SVIs, OSPF ABR, redundancy
- 2x Access Layer Switches — end device connectivity

### Inter-VLAN Routing

Used Layer 3 Switch SVIs — the modern enterprise approach, not router-on-a-stick.

```
interface Vlan10
 ip address 192.168.10.2 255.255.255.0
 ip helper-address 10.10.100.10
 standby 1 ip 192.168.10.1
 standby 1 priority 110
 standby 1 preempt
```

### STP Load Balancing — Rapid PVST+

Both core switches actively forward traffic at all times — no idle standby links.

| Switch | Root Bridge For | Secondary For |
|---|---|---|
| C1 | VLANs 10, 20, 30, 200 | VLANs 40, 50, 60, 1 |
| C2 | VLANs 40, 50, 60, 1 | VLANs 10, 20, 30, 200 |

```
! On C1
spanning-tree vlan 10,20,30,200 root primary
spanning-tree vlan 40,50,60,1 root secondary

! On C2
spanning-tree vlan 40,50,60,1 root primary
spanning-tree vlan 10,20,30,200 root secondary
```

### STP Security

| Feature | Applied On | Purpose |
|---|---|---|
| Rapid PVST+ | All switches | Convergence in ~6 seconds |
| PortFast | End-device ports | Skip STP states — instant connectivity |
| BPDUGuard | End-device ports | Shutdown port if BPDU received |
| RootGuard | Core to Access uplinks | Prevent unauthorized root election |

### HSRP — Gateway Redundancy & Load Balancing

Each VLAN has a Virtual IP shared between C1 and C2. The split mirrors the STP design
so both switches carry real traffic simultaneously.

| VLAN | Active (Priority 110) | Standby (Priority 90) | Virtual IP |
|---|---|---|---|
| 10 — HR | C1 | C2 | 192.168.10.1 |
| 20 — IT | C1 | C2 | 192.168.20.1 |
| 30 — Management | C1 | C2 | 192.168.30.1 |
| 40 — Finance | C2 | C1 | 192.168.40.1 |
| 50 — Employees | C2 | C1 | 192.168.50.1 |
| 60 — Printers | C2 | C1 | 192.168.60.1 |

### OSPF Route Summarization

Each ABR advertises a single summary route to Area 0, keeping the backbone routing table clean.

```
! Area 1 ABR
area 1 range 192.168.0.0 255.255.0.0

! Area 2 ABR
area 2 range 172.31.0.0 255.255.0.0
```

---

## Area 3 — Server Farm | `10.10.100.0/24`

| Device | IP | Role |
|---|---|---|
| DHCP Server | 10.10.100.10 | Assigns IPs to all VLANs in Areas 1 & 2 |
| DNS Server | 10.10.100.20 | Resolves hostnames for all network devices |
| TFTP Server | 10.10.100.30 | Config backup and restore |

- HSRP VIP: `10.10.100.1`
- C1 = Root Bridge, C2 = Secondary
- PortFast + BPDUGuard + RootGuard enabled
- All server IPs are static and registered in DNS

---

## DHCP — Centralized Configuration

All pools are hosted on the central DHCP server in Area 3.
The `ip helper-address` on each SVI relays DHCP requests across areas.
The default gateway in every pool is the HSRP VIP — not a physical interface IP.

| Pool | Gateway (VIP) | DNS | Start IP | Max Hosts |
|---|---|---|---|---|
| VLAN 10 Area 1 | 192.168.10.1 | 10.10.100.20 | 192.168.10.9 | 100 |
| VLAN 20 Area 1 | 192.168.20.1 | 10.10.100.20 | 192.168.20.9 | 100 |
| VLAN 30 Area 1 | 192.168.30.1 | 10.10.100.20 | 192.168.30.9 | 100 |
| VLAN 40 Area 1 | 192.168.40.1 | 10.10.100.20 | 192.168.40.9 | 100 |
| VLAN 50 Area 1 | 192.168.50.1 | 10.10.100.20 | 192.168.50.9 | 100 |
| VLAN 10 Area 2 | 172.31.10.1 | 10.10.100.20 | 172.31.10.9 | 100 |
| VLAN 20 Area 2 | 172.31.20.1 | 10.10.100.20 | 172.31.20.9 | 100 |
| VLAN 30 Area 2 | 172.31.30.1 | 10.10.100.20 | 172.31.30.9 | 100 |
| VLAN 40 Area 2 | 172.31.40.1 | 10.10.100.20 | 172.31.40.9 | 100 |
| VLAN 50 Area 2 | 172.31.50.1 | 10.10.100.20 | 172.31.50.9 | 100 |

Printers (VLAN 60) have static IPs registered in DNS by hostname.

---

## DNS Records

| Hostname | IP |
|---|---|
| dhcp-server | 10.10.100.10 |
| dns-server | 10.10.100.20 |
| tftp-server | 10.10.100.30 |
| printer1-f1-b1 | 192.168.60.30 |
| printer1-f1-b2 | 172.31.60.10 |
| printer1-f2-b1 | 192.168.60.10 |
| printer2-f1-b1 | 192.168.60.40 |
| printer2-f2-b2 | 172.31.60.30 |

---

## Verification Summary

- PC in VLAN 10 Area 1 receives `192.168.10.11`, gateway `192.168.10.1` (HSRP VIP) via DHCP
- OSPF routing table on Area 0 shows `O IA 192.168.0.0/16`, `O IA 172.31.0.0/16`, `O IA 10.10.100.0/24`
- Cross-area ping by hostname: `man 1 b1` pings `printer1-f2-b2` (172.31.60.30) successfully
- STP: C1 confirmed Root for VLANs 10/20/30, C2 confirmed Root for VLANs 40/50/60
- HSRP: C1 Active on VLANs 10/20/30, C2 Active on VLANs 40/50/60

Full verification screenshots are available in the project PDF.

---

## Planned Improvements (v2)

- [ ] NAT — Internet access simulation
- [ ] ACLs — Inter-VLAN and inter-area traffic filtering
- [ ] Port Security — Restrict MAC addresses per port
- [ ] DHCP Snooping — Prevent rogue DHCP servers
- [ ] Dynamic ARP Inspection — Protect against ARP poisoning
- [ ] NTP — Centralized time synchronization
- [ ] Syslog — Centralized logging

---

## Technologies Used

`Cisco Packet Tracer` `OSPF Multi-Area` `HSRP` `Rapid PVST+` `EtherChannel`
`STP Load Balancing` `Inter-VLAN Routing` `DHCP Relay` `DNS` `TFTP`
`Layer 3 Switching` `Route Summarization`

---

## Author

**Hasan Elfanhry**
Networking Student | Aspiring Network Engineer
Building toward CCNA



