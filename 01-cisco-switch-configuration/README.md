# Cisco Switch Configuration & SSH Troubleshooting

## Overview

This project documents a hands-on networking lab completed
at Centennial College using a physical Cisco Catalyst 1000
switch running Cisco IOS 15.2(7)E6.

The objective was to configure a managed switch, establish
IPv4/IPv6 connectivity, enable remote administration,
and investigate MAC address learning.

## Technologies

- Cisco Catalyst 1000
- Cisco IOS
- VLANs and Switch Virtual Interfaces (SVIs)
- IPv4 and IPv6
- SSHv2
- Windows PowerShell
- MAC address tables

## Network Configuration

The management interface was configured on VLAN 99.

| Device | Interface | IPv4 Address |
|--------|-----------|--------------|
| Switch S1 | VLAN 99 | 192.168.1.2/24 |
| PC-A | Ethernet | 192.168.1.10/24 |

IPv6 connectivity was also configured and verified.

### Management VLAN Verification

The switch was configured with VLAN 99 as its management VLAN.

After assigning an active switch port to VLAN 99, the Switch
Virtual Interface (SVI) became operational, with both the
interface and line protocol reporting an up/up status.

The following screenshot shows the verification performed
on the physical Cisco Catalyst 1000 switch.

![VLAN 99 interface status](screenshots/vlan99-status.png)

### MAC Address Learning

The switch automatically learned the connected PC's
MAC address and associated it with the corresponding
interface in VLAN 99.

![Dynamic MAC learning](screenshots/mac-dynamic.png)

### Static MAC Address Configuration

I manually configured a static MAC address entry
(0050.56be.6c89) on VLAN 99, associating it with
interface Gi1/0/6.

I then verified that the static entry appeared in the
switch's MAC address table, alongside the dynamically
learned MAC address of the connected workstation.

![Static MAC entry](screenshots/mac-static.png)

## Troubleshooting: SSH Compatibility

### Issue Encountered

When connecting to the Cisco switch from Windows PowerShell,
the SSH client returned:

```text
Unable to negotiate with 192.168.1.2 port 22:
no matching key exchange method found.
```

### Investigation

The switch offered legacy SHA-1-based key exchange
algorithms that were not enabled by default in the
Windows OpenSSH client.

### Resolution

I successfully established the connection using:

```powershell
ssh -o KexAlgorithms=+diffie-hellman-group14-sha1 -o HostKeyAlgorithms=+ssh-rsa admin@192.168.1.2
```

The compatibility options applied only to that connection.

**Security note:** This was a temporary workaround in an
isolated educational lab. Enabling legacy cryptographic
algorithms is not recommended for production environments.

## Verification

| Verification | Result | Evidence |
|---|---|---|
| VLAN 99 SVI | Up/up | Screenshot |
| IPv4 connectivity | Successful | Lab notes |
| IPv6 connectivity | Successful | Lab notes |
| SSH remote access | Successful after troubleshooting | Lab notes |
| Dynamic MAC learning | Entry learned on Gi1/0/6 | Screenshot |
| Static MAC configuration | Entry present on Gi1/0/6 | Screenshot |

## Key Takeaways

This lab helped me understand how management VLANs and switch virtual interfaces work on physical Cisco equipment. I also gained experience troubleshooting SSH compatibility issues and examining how switches learn MAC addresses.

One of my main takeaways was learning to distinguish between network connectivity problems and authentication or cryptographic negotiation problems.

