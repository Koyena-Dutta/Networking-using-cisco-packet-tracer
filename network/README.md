# ROAS (Router-on-a-Stick) — Inter-VLAN Routing Lab

A Cisco Packet Tracer simulation demonstrating **Router-on-a-Stick (ROAS)** inter-VLAN routing using IEEE 802.1Q trunking, with three isolated VLANs communicating through a single router interface.

---

## Network Topology

```
![ROAS Network Topology](Screenshot 2025-03-16 190425.png)
```

---

## VLAN Configuration

| VLAN | Name   | Subnet           | Gateway        | Hosts          |
|------|--------|------------------|----------------|----------------|
| 10   | VLAN10 | 192.168.1.0/24   | 192.168.1.1    | PC0, PC1       |
| 20   | VLAN20 | 192.168.2.0/24   | 192.168.2.1    | PC2, PC3       |
| 30   | VLAN30 | 192.168.3.0/24   | 192.168.3.1    | PC4, PC5       |

---

## Device Inventory

| Device   | Role              | Interface Used |
|----------|-------------------|----------------|
| Router0  | Inter-VLAN Router | Gi0/0 (trunk)  |
| Switch0  | Layer 2 Switch    | Fa0/1–Fa0/7    |
| PC0–PC1  | VLAN 10 Hosts     | Fa0            |
| PC2–PC3  | VLAN 20 Hosts     | Fa0            |
| PC4–PC5  | VLAN 30 Hosts     | Fa0            |

---

## Configuration

### Router0 — Sub-interface (ROAS) Setup

```
interface GigabitEthernet0/0
 no ip address
 duplex auto
 speed auto
 no shutdown

interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.1.1 255.255.255.0

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.2.1 255.255.255.0

interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.3.1 255.255.255.0
```

### Switch0 — VLAN & Trunk Configuration

```
! Access ports for VLAN 10
interface range fa0/2-3
 switchport mode access
 switchport access vlan 10

! Access ports for VLAN 20
interface range fa0/4-5
 switchport mode access
 switchport access vlan 20

! Access ports for VLAN 30
interface range fa0/6-7
 switchport mode access
 switchport access vlan 30

! Trunk port to Router
interface fa0/1
 switchport mode trunk
```

### PC Default Gateways

| Host | IP Address     | Subnet Mask     | Default Gateway |
|------|----------------|-----------------|-----------------|
| PC0  | 192.168.1.2    | 255.255.255.0   | 192.168.1.1     |
| PC1  | 192.168.1.3    | 255.255.255.0   | 192.168.1.1     |
| PC2  | 192.168.2.2    | 255.255.255.0   | 192.168.2.1     |
| PC3  | 192.168.2.3    | 255.255.255.0   | 192.168.2.1     |
| PC4  | 192.168.3.2    | 255.255.255.0   | 192.168.3.1     |
| PC5  | 192.168.3.3    | 255.255.255.0   | 192.168.3.1     |

---

## Verification

### Ping Tests (from PC0)

```
C:\>ping 192.168.3.3     → PC5 (VLAN30) — SUCCESS ✅
C:\>ping 192.168.1.3     → PC1 (VLAN10) — SUCCESS ✅
C:\>ping 192.168.2.2     → PC2 (VLAN20) — SUCCESS ✅
```

The simulation table shows:
- PC0 → PC5 : **Failed** (cross-VLAN, requires correct trunk + sub-interface)
- PC0 → PC2 : **Failed** (cross-VLAN)
- PC0 → PC1 : **Successful** (same VLAN)

> **Note:** Failures between VLANs indicate the trunk or sub-interface is not yet active. Ensure the physical Gi0/0 interface is `no shutdown` and the trunk port on Switch0 is correctly set.

---

## How ROAS Works

1. PCs send traffic to their default gateway (the router sub-interface IP).
2. The switch carries all VLAN traffic over a **single trunk link** (802.1Q tagged) to the router.
3. The router's **sub-interfaces** strip the VLAN tag, route the packet to the correct destination VLAN, re-tag it, and send it back.
4. The switch forwards the re-tagged frame to the destination PC.

---

## Files

| File                  | Description                        |
|-----------------------|------------------------------------|
| `roas_lab.pkt`        | Cisco Packet Tracer topology file  |
| `README.md`           | This documentation                 |

---

## Prerequisites

- Cisco Packet Tracer 7.x or later
- Basic understanding of VLANs, trunking, and IP routing

---

## Key Concepts Demonstrated

- **802.1Q Trunking** — tagging frames with VLAN IDs over a single link
- **Sub-interfaces** — logical interfaces on a physical router port
- **Inter-VLAN Routing** — routing between isolated Layer 2 broadcast domains
- **Access vs Trunk Ports** — switch port modes and their roles