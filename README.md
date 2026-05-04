# 🏢 Enterprise Network Simulation (CCNA Level Project)

A full-scale enterprise network designed and implemented using Cisco Packet Tracer, built as a hands-on CCNA-level lab.

This project simulates a real corporate environment with multi-area OSPF, VLAN segmentation, redundancy, and centralized services.

---

## 📌 Project Summary

- 🔹 **4 OSPF Areas:** (Core + 3 branches)
- 🔹 **2 Enterprise Buildings:** Full hierarchical design.
- 🔹 **Server Farm:** Centralized DHCP, DNS, and TFTP.
- 🔹 **Layer 3 Switching:** Robust Inter-VLAN Routing.
- 🔹 **HSRP:** Gateway Redundancy for high availability.
- 🔹 **STP Design:** Rapid PVST+ with load balancing.
- 🔹 **EtherChannel:** Link aggregation for increased bandwidth and redundancy.
- 🔹 **DHCP Relay:** Centralized IP management using `ip helper-address`.

---

## 🗺️ Network Topology

<p align="center">
  <img src="1000086882.jpg" alt="Enterprise Network Topology" width="850">
</p>

*Full enterprise hierarchical design: Core → Distribution → Access → Servers.*

---

## 📄 Documentation & Lab Files

For a deep dive into the technical details, I have included comprehensive documentation:

* **[PDF] Configuration & Verification Guide:** This file contains all CLI commands, screenshots of the configuration, and verification tests (Ping, Traceroute, OSPF Neighbor Adjacency, etc.).
* **Enterprise-Network-Simulationpkt.pkt Lab File:** The original Cisco Packet Tracer file for testing and simulation.

---

## 🔷 Technical Highlights

### 🌐 OSPF Multi-Area Design
- **Area 0:** Backbone connecting the Core switches.
- **Area 1 & 2:** Building 1 and Building 2.
- **Area 3:** Dedicated area for the Server Farm.

### 🔁 Inter-VLAN Routing & HSRP
Implemented SVI-based routing with HSRP for gateway redundancy.
```markdown
interface Vlan10
 ip address 192.168.10.2 255.255.255.0
 ip helper-address 10.10.100.10
 standby 10 ip 192.168.10.1
 standby 10 priority 110
 standby 10 preempt
