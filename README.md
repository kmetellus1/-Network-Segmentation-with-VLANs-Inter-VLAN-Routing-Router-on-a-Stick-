# 🧱 Network Segmentation with VLANs & Inter-VLAN Routing (Router-on-a-Stick)

![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Networking](https://img.shields.io/badge/Networking-Layer%202%20%26%203-0A66C2?style=for-the-badge)
![VLANs](https://img.shields.io/badge/Focus-VLANs%20%2F%20Segmentation-2EA44F?style=for-the-badge)
![Security](https://img.shields.io/badge/Lens-Network%20Security-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Complete-success?style=for-the-badge)

> A hands-on lab that splits a single switch into isolated networks using **VLANs**, then re-connects them in a controlled way using **router-on-a-stick** inter-VLAN routing. Built and verified in Cisco Packet Tracer, framed through a **network-security lens**: segmentation as a defensive control.

---

## 📖 Overview

In a default network, every device on a switch can talk to every other device. That's convenient — and a security problem. If one machine is compromised, an attacker can move freely to every other host on that flat network.

This project demonstrates how to fix that with **segmentation**:

- I divided one physical switch into **two isolated logical networks** (VLANs) that cannot reach each other by default.
- I then enabled **controlled communication** between them by forcing all cross-VLAN traffic through a router using the **router-on-a-stick** design.
- I **verified isolation and connectivity** with targeted ping tests and the device's own routing/VLAN tables.

The result is a small but realistic model of how enterprises keep, for example, a Guest network away from a Finance network — while still allowing the specific traffic that's supposed to flow.

---

## 🎯 Skills Demonstrated

- **VLAN creation & port assignment** (Layer 2 segmentation)
- **802.1Q trunking** to carry multiple VLANs over a single link
- **Inter-VLAN routing** via router subinterfaces (router-on-a-stick)
- **IPv4 addressing & gateway design** across multiple subnets
- **Connectivity verification & troubleshooting** using `show` commands and `ping`
- **Security reasoning** — segmentation, blast-radius reduction, and VLAN-hopping awareness

---

## 🛠️ Tools & Technologies

| Category | Used |
|---|---|
| Simulation | Cisco Packet Tracer |
| Switching | Cisco 2960 Switch |
| Routing | Cisco 2911 Router |
| Protocols / Standards | 802.1Q (dot1Q), IPv4, ICMP |
| Concepts | VLANs, Trunking, Inter-VLAN Routing, Network Segmentation |

---

## 🗺️ Network Topology

```
   PC0 (Sales) ──┐
   PC1 (Sales) ──┤
                 ├──[ Switch0 ]══ 802.1Q Trunk ══[ Router0  Gig0/0 ]
   PC2 (IT)    ──┘
```

<img width="2560" height="1297" alt="Network Segmentation " src="https://github.com/user-attachments/assets/8efd8653-5921-4ce8-abd1-682051af8508" />


### VLAN & Addressing Plan

| VLAN | Name | Network | Gateway (Router) | Members |
|:---:|:---|:---|:---|:---|
| 10 | SALES | 192.168.10.0/24 | 192.168.10.1 | PC0, PC1 |
| 20 | IT | 192.168.20.0/24 | 192.168.20.1 | PC2 |

The two VLANs live on **different subnets**, so they require a router to communicate — that's what makes the segmentation meaningful.

---

## ⚙️ Implementation

### 1. Create VLANs and assign access ports (Switch0)
Each PC port is locked to exactly one VLAN.

```bash
vlan 10
 name SALES
vlan 20
 name IT
interface range fa0/1 - 2
 switchport mode access
 switchport access vlan 10
interface fa0/3
 switchport mode access
 switchport access vlan 20
```

### 2. Configure the trunk to the router (Switch0)
The single link to the router must carry **both** VLANs.

```bash
interface gig0/1
 switchport mode trunk
```

### 3. Router-on-a-stick: one subinterface per VLAN (Router0)
The router splits its one physical port into virtual interfaces, each acting as a VLAN's gateway. The `dot1Q` tag **must match** the switch's VLAN ID.

```bash
interface gig0/0
 no shutdown
interface gig0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
interface gig0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

### 4. Address the hosts
Each PC's default gateway points to **its own VLAN's** router subinterface.

| Device | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC1 | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 |
| PC2 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |

<img width="912" height="674" alt="PC0 conf" src="https://github.com/user-attachments/assets/ca282d97-761a-40a9-8d94-ef451c2ff6d5" />
<img width="909" height="676" alt="PC1 config" src="https://github.com/user-attachments/assets/329ce74a-55cb-4d01-abe7-a8466d52b796" />
<img width="900" height="676" alt="PC2 config" src="https://github.com/user-attachments/assets/e3f6334a-8e74-485d-9bb7-a48095205af5" />



---

## ✅ Verification & Results

| Test | Command | Expected | Why it matters |
|---|---|---|---|
| Same-VLAN reachability | `ping 192.168.10.11` from PC0 | ✅ Success | Confirms Layer 2 works within a VLAN (no router involved) |
| Cross-VLAN reachability | `ping 192.168.20.10` from PC0 | ✅ Success | Proves inter-VLAN routing is working through the router |
| VLAN membership | `show vlan brief` (switch) | Ports mapped to VLAN 10 / 20 | Confirms segmentation at Layer 2 |
| Routing table | `show ip route` (router) | Both networks shown as `C` (connected) | Confirms the router knows both subnets |

The key result: **the cross-VLAN ping only succeeds because of the router**. Remove the subinterfaces and that traffic dies — which is exactly the controlled-communication behavior we want.

<img width="898" height="676" alt="Ping test " src="https://github.com/user-attachments/assets/0995ec4b-3846-4a5f-b720-1ca1423fcf65" />
<img width="898" height="799" alt="switch config" src="https://github.com/user-attachments/assets/d0f0c69d-b61a-477b-a5fc-a84920946dd0" />
<img width="899" height="794" alt="Router config" src="https://github.com/user-attachments/assets/1557d7b4-fd0c-48f6-ab10-f850f86760bb" />



---

## 🛡️ Why This Matters for Security

VLAN segmentation is one of the most common and effective **defensive controls** in enterprise networks:

- **Limits blast radius** — if an attacker compromises one segment, isolation stops them from freely pivoting to others.
- **Enforces least-access by design** — networks like Guest, Finance, and IT are kept apart unless traffic is explicitly allowed to cross.
- **Foundation for further controls** — once segments exist, you can layer ACLs/firewall rules on the router to permit *only* the traffic that should flow.

It's also worth knowing the offensive side: **VLAN hopping** (via switch spoofing or double-tagging) is a real attack class. That's precisely why hardening guidance recommends disabling automatic trunking (DTP) and shutting down unused ports — defenses that only make sense once you understand how the segmentation itself works.

> This lab is the thing a SOC/Blue Team analyst spends their time defending. Building it bottom-up makes that defense tangible.

---

## 💡 Key Takeaways

- A switch alone gives you connectivity **within** a network; a router is required to move traffic **between** networks — even when they share one switch.
- The `dot1Q` VLAN tag on the router subinterface must exactly match the VLAN ID on the switch — the most common point of failure.
- `no shutdown` on the **physical** interface is required for its subinterfaces to function.
- Segmentation isn't just a networking convenience — it's a core security architecture decision.

---

## 🚀 Possible Extensions

- Add **ACLs** on the router to allow only specific traffic between VLANs (e.g., let IT reach Sales, but not the reverse).
- Introduce a **DHCP** server per VLAN for automatic addressing.
- Add a **Syslog server** and forward device logs — the first step toward SIEM-based monitoring.
- Implement **switchport hardening** (port security, disable DTP) to defend against VLAN hopping.

---

## 📂 Repository Structure

```
.
├── README.md
├── /screenshots          # Packet Tracer captures referenced above
└── inter-vlan-lab.pkt    # The Packet Tracer project file
```

---

## 👤 Author

**Kirkland Metellus** — IT Support Technician | Aspiring SOC Analyst
🔗 GitHub: [github.com/kmetellus1](https://github.com/kmetellus1)

*Part of an ongoing hands-on networking & security lab series.*
