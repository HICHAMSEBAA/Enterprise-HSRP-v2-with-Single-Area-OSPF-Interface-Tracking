# Enterprise HSRP v2 with Single-Area OSPF & Interface Tracking

![Topology](https://img.shields.io/badge/Platform-EVE--NG-blue)
![Vendor](https://img.shields.io/badge/Vendor-Cisco%20IOS-orange)
![Protocol](https://img.shields.io/badge/Protocol-HSRP%20v2%20%7C%20OSPFv2-green)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

This repository contains the complete network architecture, Cisco IOS configurations, Linux host parameters, and failover verification logs for a High Availability First Hop Redundancy Protocol (**HSRP v2**) lab integrated with **OSPF Area 0** routing and **Uplink Interface Tracking**.

---

## 📋 Overview & Architecture

The objective of this design is to eliminate single points of failure at the default gateway layer while maintaining optimal dynamic routing propagation into the core network:

1. **First-Hop Redundancy**: **R1** and **R2** run HSRP v2 on Group 1, presenting a Virtual IP (`192.168.1.254`) to VLAN 10 access hosts (**PC1** and **PC2**).
2. **Deterministic Forwarding**: **R1** is configured with an elevated priority (`110`) and preemption to act as the primary Active gateway. **R2** operates in Standby (`100`).
3. **Uplink Tracking (Deterministic Failover)**: **R1** monitors its core interface (`GigabitEthernet0/1`). If this uplink fails, Object Tracking decrements R1's priority by 20 (`110` → `90`), allowing **R2** to preempt and assume Active gateway status.
4. **Core Dynamic Routing**: Single-area **OSPF (Area 0)** connects edge routers **R1** and **R2** to core router **R3**. Routing topology updates dynamically reflect interface state changes to preserve symmetric return paths from the application server (`172.16.1.100`).

---

## 📐 Network Topology

```text
                     +--------------------------+
                     |    Linux Ubuntu Server   |
                     |     172.16.1.100/24      |
                     +------------+-------------+
                                  |
                                  | (Gi0/2 - 172.16.1.1)
                            +-----+-----+
                            |    R3     | (RID: 3.3.3.3)
                            +--+-----+--+
                              /       \
              (10.1.3.0/24)  /         \  (10.2.3.0/24)
              Gi0/0 (.3)    /           \   Gi0/1 (.3)
                           /             \
             Gi0/1 (.1)   /               \  Gi0/1 (.2)
                   +-----+-----+     +-----+-----+
                   |    R1     |     |    R2     |
                   | (Active)  |     | (Standby) |
                   +-----+-----+     +-----+-----+
             Gi0/0 (.1)   \               /  Gi0/0 (.2)
                           \             /
                            \   VGW     /
                          +--+---------+--+
                          | 192.168.1.254 |  (HSRP Group 1)
                          +-------+-------+
                                  |
                           (192.168.1.0/24)
                                  |
                  +---------------+---------------+
                  |                               |
             +----+----+                     +----+----+
             |   SW1   |<===[Trunk 802.1q]==>|   SW2   |
             +----+----+                     +----+----+
                  |                               |
          (Eth0/0 - VLAN 10)              (Gi0/0 - VLAN 10)
                  |                               |
          +-------+-------+               +-------+-------+
          |  Debian PC1   |               |  Debian PC2   |
          | 192.168.1.10  |               | 192.168.1.20  |
          +---------------+               +---------------+