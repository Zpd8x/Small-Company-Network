# VLAN Design

This document describes the VLAN architecture used in the **Small Company Network** project.

VLANs are used to logically separate company departments, reduce broadcast domains, and enforce security policies between different groups of users.

## VLAN Summary

| VLAN ID | Name | Department | Network |
|---:|---|---|---|
| 10 | ADMIN | Administration | `192.168.10.0/24` |
| 20 | STAFF | Employees | `192.168.20.0/24` |
| 30 | GUEST | Guest Users | `192.168.30.0/24` |
| 40 | SERVER | Internal Servers | `192.168.40.0/24` |

---

## VLAN 10 — ADMIN

```text
VLAN ID: 10
Name: ADMIN
Network: 192.168.10.0/24
Gateway: 192.168.10.1
```

Connected devices:

```text
ADMIN-PC1
ADMIN-PC2
```

Switch ports:

```text
SW1 Fa0/1 → ADMIN-PC1
SW1 Fa0/2 → ADMIN-PC2
```

ADMIN devices have the highest level of access in the network.

They are allowed to initiate connections toward:

```text
STAFF
GUEST
SERVER
Internet
```

---

## VLAN 20 — STAFF

```text
VLAN ID: 20
Name: STAFF
Network: 192.168.20.0/24
Gateway: 192.168.20.1
```

Connected devices:

```text
STAFF-PC1
STAFF-PC2
STAFF-PC3
STAFF-PC4
```

STAFF devices are distributed across both switches.

### SW1

```text
Fa0/3 → STAFF-PC1
Fa0/4 → STAFF-PC2
```

### SW2

```text
Fa0/1 → STAFF-PC3
Fa0/2 → STAFF-PC4
```

VLAN 20 therefore demonstrates how a single VLAN can extend across multiple switches using an 802.1Q trunk.

---

## VLAN 30 — GUEST

```text
VLAN ID: 30
Name: GUEST
Network: 192.168.30.0/24
Gateway: 192.168.30.1
```

Connected devices:

```text
GUEST-PC1
GUEST-PC2
```

Switch ports:

```text
SW2 Fa0/3 → GUEST-PC1
SW2 Fa0/4 → GUEST-PC2
```

The GUEST VLAN is intentionally restricted.

Guest devices cannot initiate connections toward:

```text
ADMIN
STAFF
SERVER
```

They are allowed to:

```text
Use DNS
Access the simulated Internet
```

---

## VLAN 40 — SERVER

```text
VLAN ID: 40
Name: SERVER
Network: 192.168.40.0/24
Gateway: 192.168.40.1
```

Connected device:

```text
SRV1
```

Switch port:

```text
SW1 Fa0/5 → SRV1
```

SRV1 uses the static address:

```text
192.168.40.10
```

Services hosted by SRV1:

```text
DNS
HTTP
```

---

## Access Port Configuration

End devices connect to switch access ports.

Example:

```cisco
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 description ADMIN-PC1
```

This configuration assigns the port exclusively to VLAN 10.

---

## SW1 VLAN Assignment

| Interface | Device | VLAN |
|---|---|---:|
| `Fa0/1` | ADMIN-PC1 | 10 |
| `Fa0/2` | ADMIN-PC2 | 10 |
| `Fa0/3` | STAFF-PC1 | 20 |
| `Fa0/4` | STAFF-PC2 | 20 |
| `Fa0/5` | SRV1 | 40 |
| `Fa0/23` | SW2 | Trunk |
| `Fa0/24` | R1 | Trunk |

---

## SW2 VLAN Assignment

| Interface | Device | VLAN |
|---|---|---:|
| `Fa0/1` | STAFF-PC3 | 20 |
| `Fa0/2` | STAFF-PC4 | 20 |
| `Fa0/3` | GUEST-PC1 | 30 |
| `Fa0/4` | GUEST-PC2 | 30 |
| `Fa0/24` | SW1 | Trunk |

---

## SW1 ↔ SW2 Trunk

The inter-switch trunk is:

```text
SW1 Fa0/23
       │
       │ 802.1Q
       │
SW2 Fa0/24
```

Allowed VLANs:

```text
20
30
```

Configuration:

```cisco
interface FastEthernet0/23
 switchport mode trunk
 switchport trunk allowed vlan 20,30
 description TRUNK_TO_SW2
```

On SW2:

```cisco
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 20,30
 description TRUNK_TO_SW1
```

---

## SW1 ↔ R1 Trunk

The link toward R1 carries all internal VLANs.

```text
R1 G0/0
   │
   │ 802.1Q
   │
SW1 Fa0/24
```

Allowed VLANs:

```text
10
20
30
40
```

Configuration on SW1:

```cisco
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 description TRUNK_TO_R1
```

R1 handles the VLAN tags using subinterfaces.

---

## Router Subinterfaces

| Subinterface | VLAN | Gateway |
|---|---:|---|
| `G0/0.10` | 10 | `192.168.10.1` |
| `G0/0.20` | 20 | `192.168.20.1` |
| `G0/0.30` | 30 | `192.168.30.1` |
| `G0/0.40` | 40 | `192.168.40.1` |

Example:

```cisco
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

---

## VLAN Verification

Useful commands:

```cisco
show vlan brief
show interfaces trunk
show interfaces status
```

Expected VLAN membership on SW1:

```text
VLAN 10 → Fa0/1, Fa0/2
VLAN 20 → Fa0/3, Fa0/4
VLAN 40 → Fa0/5
```

Expected VLAN membership on SW2:

```text
VLAN 20 → Fa0/1, Fa0/2
VLAN 30 → Fa0/3, Fa0/4
```

---

## Design Objectives

The VLAN design provides:

- Department separation
- Reduced broadcast domains
- Controlled inter-department communication
- Guest isolation
- Dedicated server network
- Multi-switch VLAN extension
- 802.1Q trunking
- Centralized inter-VLAN routing