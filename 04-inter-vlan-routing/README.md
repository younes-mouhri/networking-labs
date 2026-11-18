# Cisco Inter-VLAN Routing — Router-on-a-Stick

## Overview

This project documents Cisco Lab 4.5.2 — **Implement Inter-VLAN Routing**, completed as part of the CNET 217 Switching & Routing Essentials course at Centennial College.

The lab combines two Cisco switches, one router, and two workstations to route traffic between separate VLANs using **Router-on-a-Stick**. It builds on [Project 03 — VLANs and Trunking](../03-vlans-and-trunking) by adding Layer 3 connectivity.

**Lab status:** Completed according to my lab notes. Screenshots and recorded command outputs are being organized for the portfolio.

## Technologies

- Cisco IOS and Cisco Catalyst switches
- IPv4 addressing and default gateways
- VLANs, access ports, and switch virtual interfaces (SVIs)
- IEEE 802.1Q trunking and router subinterfaces
- Router-on-a-Stick inter-VLAN routing
- Cisco IOS CLI, `ping`, and Windows `tracert`

## Network Topology

The physical topology consists of **R1**, **S1**, **S2**, **PC-A**, and **PC-B**:

```text
               R1
         G0/0/1 (802.1Q)
                |
             trunk
                |
               S1 ---------------- S2
               |       trunk       |
              PC-A                PC-B
            VLAN 20             VLAN 30
```

The Cisco worksheet uses FastEthernet switch port labels; the physical Catalyst switches in the lab use GigabitEthernet naming. The interface examples below follow the prepared Catalyst port mapping and should be checked against actual cabling evidence.

| Connection | Worksheet port(s) | Prepared Catalyst mapping |
|---|---|---|
| R1 ↔ S1 | R1 G0/0/1 ↔ S1 F0/5 | R1 G0/0/1 ↔ S1 Gi1/0/5 |
| S1 ↔ S2 | F0/1 ↔ F0/1 | Gi1/0/1 ↔ Gi1/0/1 |
| PC-A ↔ S1 | S1 F0/6 | S1 Gi1/0/6 |
| PC-B ↔ S2 | S2 F0/18 | S2 Gi1/0/18 |

### IP Addressing

| Device | Interface / VLAN | IPv4 address | Default gateway |
|---|---|---|---|
| R1 | G0/0/1.10 (VLAN 10) | 192.168.10.1/24 | — |
| R1 | G0/0/1.20 (VLAN 20) | 192.168.20.1/24 | — |
| R1 | G0/0/1.30 (VLAN 30) | 192.168.30.1/24 | — |
| R1 | G0/0/1.1000 (native) | No IP address | — |
| S1 | SVI VLAN 10 | 192.168.10.11/24 | 192.168.10.1 |
| S2 | SVI VLAN 10 | 192.168.10.12/24 | 192.168.10.1 |
| PC-A | Ethernet (VLAN 20) | 192.168.20.3/24 | 192.168.20.1 |
| PC-B | Ethernet (VLAN 30) | 192.168.30.3/24 | 192.168.30.1 |

## VLAN Configuration

Five VLANs were configured on both switches.

| VLAN ID | Name | Purpose |
|---|---|---|
| 10 | Management | Switch management SVIs |
| 20 | Sales | PC-A |
| 30 | Operations | PC-B |
| 999 | ParkingLot | Unused switch ports |
| 1000 | Native | Native VLAN on trunk links |

### Creating VLANs

Example configuration from the Lab 4.5.2 preparation notes:

```cisco
configure terminal
vlan 10
 name Management
vlan 20
 name Sales
vlan 30
 name Operations
vlan 999
 name ParkingLot
vlan 1000
 name Native
```

Confirm VLAN membership and port assignments with:

```cisco
show vlan brief
```

## Access Port Configuration

The workstations belong to **different VLANs and subnets**, unlike the endpoints in the previous trunking lab.

**S1 — PC-A in VLAN 20**

```cisco
interface GigabitEthernet1/0/6
 switchport mode access
 switchport access vlan 20
 no shutdown
```

**S2 — PC-B in VLAN 30**

```cisco
interface GigabitEthernet1/0/18
 switchport mode access
 switchport access vlan 30
 no shutdown
```

Unused switch ports were assigned to the dedicated parking VLAN 999 and administratively disabled. Active PC and trunk interfaces must be excluded from shutdown ranges.

## Management VLAN Configuration

Both switches use a VLAN 10 SVI for management.

**S1**

```cisco
interface vlan 10
 ip address 192.168.10.11 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.10.1
```

**S2**

```cisco
interface vlan 10
 ip address 192.168.10.12 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.10.1
```

The gateway points to the router's VLAN 10 subinterface. It allows these Layer 2 switches to send management traffic to other subnets.

## 802.1Q Trunk Configuration

The topology requires **two kinds of trunk links**: S1 ↔ S2 and S1 ↔ R1.

The prepared configurations use static trunking, native VLAN **1000**, and an explicit allowed-VLAN list of **10,20,30,1000**. The parking VLAN 999 is deliberately excluded.

### S1 — Inter-Switch and Router-Facing Trunks

```cisco
interface GigabitEthernet1/0/1
 switchport mode trunk
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 10,20,30,1000
 no shutdown

interface GigabitEthernet1/0/5
 switchport mode trunk
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 10,20,30,1000
 no shutdown
```

### S2 — Inter-Switch Trunk

```cisco
interface GigabitEthernet1/0/1
 switchport mode trunk
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 10,20,30,1000
 no shutdown
```

Verify operational trunks and permitted VLANs with:

```cisco
show interfaces trunk
```

### Why the Trunk and Native VLAN Matter

The trunks transport VLAN-tagged frames between switches and from S1 to the router. Setting a consistent native VLAN prevents a native-VLAN mismatch; restricting allowed VLANs avoids carrying unnecessary VLANs over these links.

A configured trunk may not appear as operational if the link is down. For example, shutting down **R1 G0/0/1** would bring down its subinterfaces and interrupt inter-VLAN routing, even if the switch-side trunk is configured.

## Router-on-a-Stick Configuration

R1 uses one physical interface, **G0/0/1**, with logical subinterfaces for VLANs 10, 20 and 30. Each routed subinterface provides the default gateway for its subnet. A fourth subinterface identifies native VLAN 1000 without assigning an IP address.

Example router configuration from the lab notes:

```cisco
interface GigabitEthernet0/0/1
 no shutdown

interface GigabitEthernet0/0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 description VLAN 10 Management

interface GigabitEthernet0/0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 description VLAN 20 Sales

interface GigabitEthernet0/0/1.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 description VLAN 30 Operations

interface GigabitEthernet0/0/1.1000
 encapsulation dot1Q 1000 native
 description Native VLAN
```

Verify interface states on R1:

```cisco
show ip interface brief
```

### How Inter-VLAN Routing Works

When PC-A in VLAN 20 contacts PC-B in VLAN 30, it sends traffic to its default gateway, **192.168.20.1**. R1 receives the tagged VLAN 20 traffic, routes it toward the connected VLAN 30 network, and forwards it back through the trunk for delivery to PC-B.

Without a routing device or Layer 3 switch, the two VLANs remain separate Layer 2 broadcast domains.

## Connectivity Verification

The lab worksheet and completed checkboxes identify the following tests. **Exact success rates, round-trip times, and traceroute output have not yet been transcribed into the notes** and will be added from screenshots.

| Source | Test | Purpose |
|---|---|---|
| PC-A | `ping 192.168.20.1` | Reach its VLAN 20 default gateway |
| PC-A | `ping 192.168.30.3` | Test inter-VLAN routing to PC-B |
| PC-A | `ping 192.168.10.12` | Test routed access to S2 management |
| PC-B | `tracert 192.168.20.3` | Identify the routed path toward PC-A |
| S1 / S2 | `show vlan brief` | Check VLAN creation and access-port membership |
| S1 / S2 | `show interfaces trunk` | Check native and allowed VLANs |
| R1 | `show ip interface brief` | Check the physical interface and subinterfaces |

For the PC-B traceroute, the expected first routed hop is **192.168.30.1** (R1's VLAN 30 gateway). Actual output should be confirmed from the saved capture rather than assumed.

## Troubleshooting and Verification Notes

The key operational dependencies in this lab were:

1. **Physical link state:** Router subinterfaces depend on the parent physical interface being enabled.
2. **VLAN membership:** Each PC-facing access port must belong to the correct VLAN.
3. **Trunk consistency:** Both switches and the router-facing link must transport the required VLANs, with a matching native VLAN.
4. **Default gateways:** PC-A, PC-B, S1 and S2 need the correct gateway to communicate beyond their own subnet.

The Notion worksheet does not record a specific fault → diagnosis → fix sequence for Lab 4.5.2, so no troubleshooting incident is claimed here.

## Verification Summary

| Lab task | Status in notes |
|---|---|
| Physical topology and basic settings | Completed |
| VLAN creation and access-port assignment | Completed |
| Management SVIs and gateways | Completed |
| Unused-port parking and shutdown | Completed |
| Static 802.1Q trunks | Completed |
| Router-on-a-Stick subinterfaces | Completed |
| Connectivity checks and traceroute procedure | Marked completed; output screenshots pending |
| Publicly verified screenshot evidence | Pending |

## Key Takeaways

This lab extended my previous switching work by implementing **Layer 3 communication between separate VLANs**. I practiced configuring a router with multiple 802.1Q subinterfaces, managing VLAN-specific default gateways, and restricting which VLANs are carried on trunks.

The main concept was understanding how one router interface can support multiple networks while the switches maintain separate Layer 2 broadcast domains.

---

**Environment:** Centennial College Cisco networking laboratory  
**Course:** CNET 217 — Switching & Routing Essentials  
**Source lab:** Cisco 4.5.2 — Implement Inter-VLAN Routing  
**Notes:** [Lab 4.5.2 in Notion](https://app.notion.com/p/414c74aed02c81ea89c9d522809f69d3)
