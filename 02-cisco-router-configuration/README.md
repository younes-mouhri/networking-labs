
# Cisco Router Configuration — IPv4, IPv6 & SSH

> 🚧 **Work in Progress** — Physical Cisco Lab

## Overview

This project documents a hands-on Cisco router configuration lab completed at Centennial College as part of my CCNA studies.

The objective is to configure and secure a Cisco router, establish IPv4 and IPv6 connectivity between two networks, enable SSH remote administration, and examine routing and interface information.

The lab is being performed on physical networking equipment.

## Technologies

- Cisco IOS
- Cisco Routers and Switches
- IPv4 and IPv6
- Static IP Addressing
- Switch Virtual Interfaces
- SSHv2
- Windows PowerShell
- Tera Term

## Network Topology

The topology consists of:

- One Cisco router (R1)
- One Cisco switch (S1)
- Two Windows workstations (PC-A and PC-B)
- Ethernet and console connections

PC-A connects to S1, which connects to R1. PC-B connects directly to R1.

### IP Addressing

| Device | Interface | IPv4 Address | IPv6 Address |
|---|---|---|---|
| R1 | G0/0/0 | 192.168.0.1/24 | 2001:db8:acad::1/64 |
| R1 | G0/0/1 | 192.168.1.1/24 | 2001:db8:acad:1::1/64 |
| R1 | Loopback0 | 10.0.0.1/24 | 2001:db8:acad:2::1/64 |
| PC-A | Ethernet | 192.168.1.10/24 | 2001:db8:acad:1::10/64 |
| PC-B | Ethernet | 192.168.0.10/24 | 2001:db8:acad::10/64 |

*The table represents the addressing plan used during the lab.*

## Configuration Progress

### 1. Physical Setup and Initialization

- [x] Connected PC-A to switch S1
- [x] Connected S1 to router R1
- [x] Connected PC-B directly to R1
- [x] Established console connections
- [x] Initialized and reloaded the network devices
- [x] Verified the initial configuration state

### 2. Workstation Configuration

- [x] Configured PC-A with static IPv4 and IPv6 addresses
- [x] Configured PC-B with static IPv4 and IPv6 addresses
- [x] Configured the appropriate default gateways
- [x] Verified the network settings using `ipconfig /all`

### 3. Basic Router Configuration

The following tasks were completed on R1:

- [x] Configured the router hostname
- [x] Disabled DNS lookup
- [x] Configured the IP domain name
- [x] Configured local user authentication
- [x] Configured privileged EXEC authentication
- [x] Generated RSA keys for SSH
- [x] Enabled SSH version 2
- [x] Configured console and VTY access
- [x] Configured session inactivity timeouts
- [x] Configured login blocking after repeated failed attempts
- [x] Configured an unauthorized-access warning banner
- [x] Enabled IPv6 unicast routing
- [x] Saved the running configuration to startup configuration

Passwords and authentication secrets are intentionally excluded from this public repository.

### 4. IPv4 and IPv6 Interface Configuration

Configured the router's two physical interfaces and Loopback0 with IPv4 and IPv6 addresses.

Example configuration:

```cisco
interface GigabitEthernet0/0/0
 description Connection to PC-B
 ip address 192.168.0.1 255.255.255.0
 ipv6 address 2001:db8:acad::1/64
 ipv6 address fe80::1 link-local
 no shutdown

interface GigabitEthernet0/0/1
 description Connection to S1
 ip address 192.168.1.1 255.255.255.0
 ipv6 address 2001:db8:acad:1::1/64
 ipv6 address fe80::1 link-local
 no shutdown

interface Loopback0
 ip address 10.0.0.1 255.255.255.0
 ipv6 address 2001:db8:acad:2::1/64
```

The router interfaces were enabled and the configuration was saved.

## Connectivity Verification

### IPv4 Connectivity

From PC-A, I tested connectivity to PC-B using:

```powershell
ping 192.168.0.10
```

**Result:** Successful.

### IPv6 Connectivity

I also verified IPv6 connectivity between the workstations:

```powershell
ping 2001:db8:acad::10
```

**Result:** Successful.

These tests confirmed connectivity between the two networks through R1.

## SSH Remote Administration

### IPv4 SSH

Configured SSHv2 and local authentication to allow secure remote router administration.

From PC-A, I successfully connected to R1 using its Loopback0 IPv4 address:

```powershell
ssh SSHadmin@10.0.0.1
```

**Result:** Successful.

### IPv6 SSH

- [ ] Establish SSH connectivity using the Loopback0 IPv6 address
- [ ] Verify remote access from PC-B
- [ ] Document the results and troubleshooting observations

**Status:** Pending the next physical lab session.

## Verification Summary

| Verification | Result | Evidence |
|---|---|---|
| Physical topology setup | Completed | Lab notes / photo |
| Device initialization | Completed | Lab notes |
| PC IPv4/IPv6 configuration | Completed | Screenshots |
| Router basic configuration | Completed | Lab notes |
| Router security configuration | Completed | Lab notes |
| IPv4/IPv6 interface configuration | Completed | Lab notes |
| IPv4 connectivity | Successful | Screenshot |
| IPv6 connectivity | Successful | Screenshot |
| IPv4 SSH remote access | Successful | Screenshot |
| IPv6 SSH remote access | Pending | — |
| Router information and routing-table analysis | Pending | — |
| IPv6 autoconfiguration | Pending | — |

## Remaining Objectives

- [ ] Verify SSH access over IPv6
- [ ] Retrieve router hardware and IOS information
- [ ] Examine the routing table
- [ ] Inspect startup configuration and interface status
- [ ] Investigate IPv6 autoconfiguration on PC-B
- [ ] Complete the lab reflection questions
- [ ] Document final observations and lessons learned

## Key Takeaways

So far, this lab has helped me gain hands-on experience configuring a Cisco router with both IPv4 and IPv6.

I practiced configuring network interfaces, securing administrative access through SSH, and verifying connectivity between different networks.

It also reinforced the importance of saving the running configuration to startup configuration to preserve changes after a device reload.

Additional observations will be added after completing the remaining lab exercises.

---

**Status:** In Progress

**Environment:** Physical networking laboratory — Centennial College

**Curriculum:** Cisco CCNA — Switching, Routing, and Wireless Essentials
