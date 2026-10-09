<div align="center">

# 🔀 Lab 01: Inter-VLAN Routing

### My first router-on-a-stick build: what I did, what confused me, and what I got wrong

![Cert](https://img.shields.io/badge/Cisco-CCNA%20200--301-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Tool](https://img.shields.io/badge/Tool-Packet%20Tracer-orange?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-Network%20Access-blueviolet?style=for-the-badge)

</div>

---

## 🎯 The goal

Get PCs in three different VLANs talking to each other, to the switch, and to a server, using **one trunk link** and **one router**. VLANs are separate broadcast domains, so the switch can't connect them by itself. A router has to do it.

---

## 🗺️ Topology

![Topology](topology.png)

```mermaid
graph LR
    PC1["💻 PC1<br/>VLAN 10"] --- S1
    PC2["💻 PC2<br/>VLAN 20"] --- S1
    PC3["💻 PC3<br/>VLAN 30"] --- S1
    S1(["🔌 S1"]) ==="trunk G0/1<br/>native VLAN 88"=== R1
    R1{{"🌐 R1"}} ---|"G0/0 · /30"| HQ(("☁️ HQ"))
    HQ --- SRV["🖥️ Server"]
```

## 📋 What I was given

<details>
<summary><b>Addressing table</b></summary>

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

</details>

<details>
<summary><b>VLAN and port table</b></summary>

| VLAN | Name | Ports |
|---|---|---|
| 10 | Faculty/Staff | F0/11-17 |
| 20 | Students | F0/18-24 |
| 30 | Guest(Default) | F0/6-10 |
| 88 | Native | G0/1 |
| 99 | Management | VLAN 99 |

</details>

---

## 🧭 How I worked through it

1. **Started on R1.** I began by figuring out how to put the addresses on the interfaces. The router uses subinterfaces (`G0/1.10`, `.20`, and so on), one per VLAN, with the physical G0/1 left without an IP.
2. **Brought the interfaces up.** I learned that `no shutdown` is only needed on the physical ports, because the subinterfaces come up with the parent.
3. **Moved to S1.** I created the VLANs, assigned the access ports, set up the trunk on G0/1, and gave the switch a management IP on VLAN 99 with a default gateway.
4. **Shut down unused ports.** The lab said every port not assigned to a VLAN had to be disabled.
5. **Configured the PCs and server** with static IPs from the table.
6. **Ran Packet Tracer's Check Results** and fixed what it flagged (see below).

The full device configs are in [`configs/R1.txt`](configs/R1.txt) and [`configs/S1.txt`](configs/S1.txt).

<details>
<summary><b>Key commands I used</b></summary>

**S1: trunk and native VLAN**
```
interface g0/1
 switchport mode trunk
 switchport trunk native vlan 88
```

**S1: management IP and gateway**
```
interface vlan 99
 ip address 172.17.99.10 255.255.255.0
 no shutdown
ip default-gateway 172.17.99.1
```

**R1: one subinterface per VLAN (VLAN 30 shown)**
```
interface g0/1.30
 encapsulation dot1Q 30
 ip address 172.17.30.1 255.255.255.0
```

**R1: native VLAN subinterface**
```
interface g0/1.88
 encapsulation dot1Q 88 native
 ip address 172.17.88.1 255.255.255.0
```

**R1: route to the server network**
```
ip route 172.17.50.0 255.255.255.0 172.17.25.1
```

</details>

---

## 🤔 What confused me

These were the questions I actually got stuck on.

**1. SVI or subinterface?**
I mixed these up at first. I thought I gave an SVI an IP and then tagged it with `dot1Q`. They're different things on different devices:

| | Where | What it's for |
|---|---|---|
| **SVI** (`interface vlan 99`) | Switch | The switch's own management IP |
| **Subinterface** (`interface g0/1.30`) | Router | The gateway for a VLAN, with `encapsulation dot1Q` |

**2. Does the switch need `encapsulation` too?**
No. The switch only deals with tagging on the trunk port (`switchport mode trunk`). Nothing is set per VLAN. The router is the one that needs `encapsulation dot1Q` on each subinterface, because its physical port isn't VLAN-aware.

**3. How can a switch have a "default route"?**
A Layer 2 switch doesn't route, so it uses `ip default-gateway` for its own management traffic. A Layer 3 switch would use `ip routing` and `ip route` instead.

**4. Where did the next hop `172.17.25.1` come from?**
Nothing in the table states it, so I worked it out. R1's G0/0 is a `/30`, which has only two usable addresses. R1 is `.2`, so the other end has to be `.1`. The server isn't on any network R1 is directly connected to, so traffic has to leave through that link, toward the HQ cloud.

> 📌 **Assumption I still need to confirm:** the table doesn't list HQ's IP. I check it with `ping 172.17.25.1` from R1.

---

## 🐛 What I got wrong

Packet Tracer's Check Results flagged five problems on my first pass.

| # | What was wrong | Why | How I fixed it |
|---|---|---|---|
| 1 | R1 G0/1.88 native VLAN | I hadn't marked the subinterface as native | `encapsulation dot1Q 88 native` |
| 2 | S1 default gateway | I never set it | `ip default-gateway 172.17.99.1` |
| 3 | S1 G0/2 still up | I forgot the spare gigabit port when shutting down unused ports | `interface g0/2`, then `shutdown` |
| 4 | VLAN 10 name | It didn't match the table exactly | `name Faculty/Staff` |
| 5 | VLAN 30 name | Same problem | `name Guest(Default)` |

I also first put `native` on the VLAN 99 subinterface. The lab says VLAN 88 is the native VLAN, so I had to move it.

### 🎓 What I took away

- 🔁 The native VLAN has to match on **both** ends of the trunk.
- 🔤 Names are checked literally, so capitalization, spaces and punctuation matter.
- 🚪 "Unused ports" includes the gigabit ports, not just the FastEthernet ones.
- 📖 **Read the tables first.** Three of my five mistakes came from not checking them closely enough.

---

## ✅ Did it work?

| Test | Result |
|---|---|
| Each PC pings its own gateway | ⬜ |
| PC to PC across VLANs | ⬜ |
| PCs and R1 ping S1 (172.17.99.10) | ⬜ |
| Everything pings the server (172.17.50.254) | ⬜ |

<!-- Replace ⬜ with ✅ once each test passes, and add screenshots to screenshots/ -->

---

## 💬 How I'd explain it in an interview

> *✏️ Rewrite this in your own words before you commit it.*

Router-on-a-stick lets devices in different VLANs communicate over a single physical link. The switch port is a trunk that tags each frame with its VLAN ID. On the router, I create one subinterface per VLAN, tell each one which tag to expect with `encapsulation dot1Q`, and give it an IP that acts as that VLAN's default gateway. The router receives a tagged frame, routes it to the destination VLAN's subnet, and sends it back out the same link with the new tag. The trade-off is that all inter-VLAN traffic shares one link, which is why larger networks use a Layer 3 switch instead.

---

<div align="center">

⬅️ [Back to all labs](../README.md)

</div>