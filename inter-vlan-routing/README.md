# Lab 01: Inter-VLAN Routing (Router-on-a-Stick)

Cisco CCNA 200-301 | Cisco Packet Tracer

## Objective

Enable PCs in three VLANs to communicate with each other, with the switch, and with a server, using a single trunk link and one router. VLANs are separate broadcast domains, so the switch cannot forward traffic between them on its own. A router has to route between the VLAN subnets.

## Topology

![Topology](topology.png)

Summary: PC1/PC2/PC3 -> S1 -> (trunk G0/1) -> R1 -> (G0/0, /30 link) -> HQ -> Server

## Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|---|
| R1 | G0/0 | 172.17.25.2 | 255.255.255.252 | N/A |
| R1 | G0/1.10 | 172.17.10.1 | 255.255.255.0 | N/A |
| R1 | G0/1.20 | 172.17.20.1 | 255.255.255.0 | N/A |
| R1 | G0/1.30 | 172.17.30.1 | 255.255.255.0 | N/A |
| R1 | G0/1.88 | 172.17.88.1 | 255.255.255.0 | N/A |
| R1 | G0/1.99 | 172.17.99.1 | 255.255.255.0 | N/A |
| S1 | VLAN 99 | 172.17.99.10 | 255.255.255.0 | 172.17.99.1 |
| PC1 | NIC | 172.17.10.21 | 255.255.255.0 | 172.17.10.1 |
| PC2 | NIC | 172.17.20.22 | 255.255.255.0 | 172.17.20.1 |
| PC3 | NIC | 172.17.30.23 | 255.255.255.0 | 172.17.30.1 |
| Server | NIC | 172.17.50.254 | 255.255.255.0 | 172.17.50.1 |

## VLAN and Port Assignments

| VLAN | Name | Ports |
|---|---|---|
| 10 | Faculty/Staff | F0/11-17 |
| 20 | Students | F0/18-24 |
| 30 | Guest(Default) | F0/6-10 |
| 88 | Native | G0/1 |
| 99 | Management | VLAN 99 |

## Approach

1. **Router interfaces.** I configured R1 with one subinterface per VLAN (G0/1.10, .20, .30, .88, .99) and left the physical G0/1 without an IP address. `no shutdown` is only needed on the physical ports, because the subinterfaces come up with the parent.
2. **Switch configuration.** On S1 I created and named the VLANs, assigned the access ports, configured G0/1 as a static trunk with native VLAN 88, and set a management IP on VLAN 99 with a default gateway.
3. **Unused ports.** I shut down every port not assigned to a VLAN.
4. **End devices.** I set static IP addresses on the PCs and the server from the addressing table.
5. **Check and fix.** I ran the Packet Tracer Check Results and corrected the items it flagged (see Issues and Fixes).

Full device configurations: [configs/R1.txt](configs/R1.txt) and [configs/S1.txt](configs/S1.txt).

### Key Commands

S1 trunk and native VLAN:

```
interface g0/1
 switchport mode trunk
 switchport trunk native vlan 88
```

S1 management IP and default gateway:

```
interface vlan 99
 ip address 172.17.99.10 255.255.255.0
 no shutdown
ip default-gateway 172.17.99.1
```

R1 subinterface (VLAN 30 shown):

```
interface g0/1.30
 encapsulation dot1Q 30
 ip address 172.17.30.1 255.255.255.0
```

R1 native VLAN subinterface:

```
interface g0/1.88
 encapsulation dot1Q 88 native
 ip address 172.17.88.1 255.255.255.0
```

R1 route to the server network:

```
ip route 172.17.50.0 255.255.255.0 172.17.25.1
```

## Concepts I Had to Clarify

**SVI vs. subinterface.** I initially mixed these up. They are different constructs on different devices:

| | Device | Purpose |
|---|---|---|
| SVI (`interface vlan 99`) | Switch | The switch's own management IP |
| Subinterface (`interface g0/1.30`) | Router | Default gateway for a VLAN, configured with `encapsulation dot1Q` |

**Encapsulation on the switch.** The switch does not need an `encapsulation` command per VLAN. Tagging is handled by the trunk port (`switchport mode trunk`). The router needs `encapsulation dot1Q` on each subinterface because its physical port is not VLAN-aware.

**Default gateway on a Layer 2 switch.** A Layer 2 switch does not route, so it uses `ip default-gateway` for its own management traffic. A Layer 3 switch would use `ip routing` and `ip route` instead.

**Next hop for the server route.** The addressing table does not state it, so I derived it. R1's G0/0 is on a /30, which has two usable addresses. R1 uses .2, so the other end must be .1 (172.17.25.1). The server is not on a directly connected network, so traffic must leave through that link toward HQ.

Assumption to confirm: HQ's IP address is not listed in the table. I verify it with `ping 172.17.25.1` from R1.

## Issues and Fixes

Packet Tracer's Check Results flagged five items on my first attempt.

| # | Issue | Cause | Fix |
|---|---|---|---|
| 1 | R1 G0/1.88 native VLAN incorrect | Subinterface was not marked native | `encapsulation dot1Q 88 native` |
| 2 | S1 default gateway incorrect | Not configured | `ip default-gateway 172.17.99.1` |
| 3 | S1 G0/2 port status incorrect | Spare gigabit port left up | `interface g0/2`, then `shutdown` |
| 4 | VLAN 10 name incorrect | Did not match the table exactly | `name Faculty/Staff` |
| 5 | VLAN 30 name incorrect | Did not match the table exactly | `name Guest(Default)` |

I also initially applied `native` to the VLAN 99 subinterface. The lab specifies VLAN 88 as the native VLAN, so I moved it.

## Lessons Learned

- The native VLAN must match on both ends of the trunk.
- Names are checked literally, so capitalization, spaces, and punctuation matter.
- "Unused ports" includes the gigabit ports, not just the FastEthernet ones.
- Read the addressing and VLAN tables fully before configuring. Three of the five issues came from not checking them closely enough.

## Verification

Commands:

```
S1# show vlan brief
S1# show interfaces trunk
S1# show ip interface brief
R1# show ip interface brief
R1# show ip route
```

| Test | Result |
|---|---|
| Each PC pings its default gateway | Pending |
| PC to PC across VLANs | Pending |
| PCs and R1 ping S1 (172.17.99.10) | Pending |
| All devices ping the server (172.17.50.254) | Pending |

Update the results after testing and add screenshots to the `screenshots/` folder.

## Summary

Router-on-a-stick allows devices in different VLANs to communicate over a single physical link. The switch port is a 802.1Q trunk that tags each frame with its VLAN ID. On the router, each subinterface is tied to one VLAN with `encapsulation dot1Q` and carries an IP address that serves as that VLAN's default gateway. The router receives a tagged frame, routes it to the destination subnet, and sends it back out the same link with the destination VLAN's tag. The trade-off is that all inter-VLAN traffic shares one link, which is why larger networks typically use a Layer 3 switch instead.

[Back to all labs](../README.md)