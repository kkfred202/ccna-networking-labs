<div align="center">

# 🧪 CCNA Networking Labs

### Hands-on Cisco labs, documented with configs, diagrams and the mistakes I made along the way

![Cert](https://img.shields.io/badge/Cisco-CCNA%20200--301-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Tool](https://img.shields.io/badge/Tool-Packet%20Tracer-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-In%20progress-yellow?style=for-the-badge)

![Labs](https://img.shields.io/badge/Labs%20done-1-success?style=flat-square)
![Planned](https://img.shields.io/badge/Labs%20planned-6-lightgrey?style=flat-square)

</div>

---

## 👋 About this repo

This is where I keep the **Packet Tracer labs** I build while studying for the **Cisco CCNA 200-301**.

Every lab has its own folder with the same ingredients: the **topology**, the **addressing and VLAN tables**, the **full device configs**, the **verification output**, and a **"What went wrong"** section. The mistakes are on purpose, because that's where the real learning happens.

---

## 🧪 Labs

| # | Lab | Topics | Status |
|---|---|---|---|
| 01 | [Inter-VLAN Routing](inter-vlan-routing/) | VLANs · 802.1Q trunking · native VLAN · router-on-a-stick · securing unused ports | ✅ Done |

### 🔜 Planned

| Lab | Topics | Status |
|---|---|---|
| Basic switch and router setup | Hostnames · passwords · SSH · banners | ⬜ |
| STP | Root bridge · PortFast · BPDU Guard | ⬜ |
| OSPF | Single-area OSPFv2 · neighbors · cost | ⬜ |
| DHCP and NAT | DHCP server and relay · PAT | ⬜ |
| ACLs | Standard and extended ACLs | ⬜ |
| Port security | Sticky MAC · violation modes | ⬜ |

> The planned list will change as I work through the exam topics.

---

## 🧰 Skills practiced so far

![VLANs](https://img.shields.io/badge/VLANs-✔-success?style=flat-square)
![Trunking](https://img.shields.io/badge/802.1Q%20Trunking-✔-success?style=flat-square)
![Subinterfaces](https://img.shields.io/badge/Router%20Subinterfaces-✔-success?style=flat-square)
![IP addressing](https://img.shields.io/badge/IPv4%20Addressing-✔-success?style=flat-square)
![Static routing](https://img.shields.io/badge/Static%20Routing-✔-success?style=flat-square)
![Port hardening](https://img.shields.io/badge/Unused%20Port%20Hardening-✔-success?style=flat-square)

---

## 📝 How each lab is documented

```
🗺️ Topology  →  📋 Tables  →  🛠️ Step-by-step config  →  ✅ Verification  →  🐛 What went wrong  →  🎓 Lessons learned
```

Each lab README answers the same questions:
**What is the goal? · How does it work? · How did I configure it? · How did I test it? · What broke and how did I fix it?**

---

## 🗂️ Repository structure

```
ccna-networking-labs/
├── README.md                  ← you are here
└── inter-vlan-routing/
    ├── README.md              ← full write-up
    ├── topology.png
    ├── configs/
    │   ├── R1.txt
    │   └── S1.txt
    └── screenshots/
```

Every new lab gets its own folder with the same layout.

---

## 🛠️ Tools

| | |
|---|---|
| 🧪 **Labs** | Cisco Packet Tracer |
| 🎯 **Goal** | Cisco CCNA 200-301 |
| 📖 **Docs** | Markdown + Mermaid diagrams on GitHub |

---

## 🔎 Running a lab yourself

1. Open the lab folder and read its `README.md`.
2. Build the topology in Packet Tracer using the addressing table.
3. Paste the configs from `configs/` onto each device, or type them step by step.
4. Run the verification commands and compare with the screenshots.

---

<div align="center">

⭐ *If you're also studying for the CCNA, feel free to follow along!*

</div>