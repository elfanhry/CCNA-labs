# CCNA-labs
This repo contains the labs that I have done during my prepration for the ccna exam ,along with notes about the configrations and the concepts I learned through hands-on labs .
# Enterprise Network Design — Multi-Area OSPF Lab

**Author:** Hasan Elfanhry  
**Tool:** Cisco Packet Tracer  
**Level:** CCNA

---

## Overview

A full enterprise network built in Cisco Packet Tracer featuring multi-area OSPF routing, VLANs, inter-VLAN routing, centralized network services, and internet connectivity.

---

## Network Topology

```
                        [ The Internet ]
                               |
                          [ Router0 ]  ← static default route 0.0.0.0/0
                               |
                        [ Area 0 — Backbone ]
                         (ASBR — default-information originate)
                        /       |        \
                    ABR1       ABR3       ABR2
                     |          |          |
                  Area 1    Area 3      Area 2
               (Building 1) (Data Center) (Building 2)
```

---

## Area Details

### Area 0 — Backbone
- Connects all areas via ABRs
- ASBR router has a static default route to the internet (`0.0.0.0/0`)
- Default route distributed to all areas using `default-information originate`

---

### Area 1 — Building 1

| VLAN | ID | Subnet | Department |
|------|----|--------|------------|
| HR | 10 | 192.168.10.0/24 | Human Resources |
| Engineering | 20 | 192.168.20.0/24 | Engineering |
| Sales | 30 | 192.168.30.0/24 | Sales |
| Printers | 100 | 192.168.100.0/24 | Network Printers |
| Native | 200 | — | Trunk Native VLAN |

**Key configurations:**
- Inter-VLAN routing via **multilayer switch** (SVIs)
- **VTP domain:** `area1` — multilayer switch = VTP Server, access switches = VTP Clients
- **Two floors**, each with its own access switch
- PCs get dynamic IPs via **DHCP** — `ip helper-address 10.10.20.10` configured on each SVI
- Printers have **static IPs** with **DNS A Records** for easy access by name

---

### Area 2 — Building 2

| VLAN | ID | Subnet | Department |
|------|----|--------|------------|
| IT | 40 | 172.31.40.0/24 | IT Department |
| Security | 50 | 172.31.50.0/24 | Security |
| Finance | 60 | 172.31.60.0/24 | Finance |
| Printers | 100 | 172.31.100.0/24 | Network Printers |
| Native | 200 | — | Trunk Native VLAN |

**Same architecture as Building 1** — multilayer switch with SVIs, VTP server/client, DHCP relay, static printer IPs with DNS entries.

---

### Area 3 — Data Center

| Service | Server | IP |
|---------|--------|----|
| DHCP Server | Server-PT | 10.10.20.10 |
| DNS Server | Server-PT | 10.10.20.20 |
| TFTP Server | Server-PT | 10.10.20.30 |

- VLAN 90 only
- SVI gateway: `10.10.20.1`
- Provides centralized DHCP, DNS, and TFTP for the entire network

---

## Technologies Used

- OSPF Multi-Area (Area 0, 1, 2, 3)
- VLANs + 802.1Q Trunking
- Inter-VLAN Routing (Layer 3 Switch SVIs)
- VTP (Server / Client)
- DHCP with `ip helper-address` relay
- DNS (A Records for printers)
- Static routing + `default-information originate` for internet access
- Native VLAN security (VLAN 200)

---

## Verification

| Test | Result |
|------|--------|
| `show ip route` on all routers | OSPF routes present ✅ |
| `ipconfig` on PCs | DHCP lease confirmed ✅ |
| Cross-area ping (Area 1 → Area 2) | Successful ✅ |
| DNS resolution for printers | Working ✅ |
| Ping to internet server | Successful ✅ |

---

## Files

| File | Description |
|------|-------------|
| `first project.pkt` | Cisco Packet Tracer lab file |
| `README.md` | This documentation |
