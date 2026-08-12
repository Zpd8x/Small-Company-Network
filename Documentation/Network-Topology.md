# Network Topology

This document describes the physical and logical topology of the **Small Company Network** project.

The lab represents a small business network distributed across two floors and connected to a simulated ISP and external Internet server.

## Topology Overview

```text
                         INTERNET-SRV
                         198.51.100.10
                               │
                               │
                              ISP
                         G0/1: 198.51.100.1
                         G0/0: 203.0.113.2
                               │
                               │
                         203.0.113.0/30
                               │
                               │
                         G0/1: 203.0.113.1
                              R1
                               │
                              G0/0
                               │
                         802.1Q TRUNK
                               │
                            Fa0/24
                              SW1
             ┌─────────────────┼─────────────────┐
             │                 │                 │
           ADMIN             STAFF             SERVER
          VLAN 10           VLAN 20           VLAN 40
          /      \          /      \              │
   ADMIN-PC1 ADMIN-PC2 STAFF-PC1 STAFF-PC2        SRV1
                               │
                               │
                         SW1 Fa0/23
                               │
                         802.1Q TRUNK
                               │
                         SW2 Fa0/24
                               │
                              SW2
                         ┌─────┴─────┐
                         │           │
                       STAFF       GUEST
                      VLAN 20     VLAN 30
                      /     \      /     \
                STAFF-PC3 STAFF-PC4 GUEST-PC1 GUEST-PC2
```

---

## Physical Layout

The internal network is divided into two floors.

### Floor 1

Contains:

```text
R1
SW1
ADMIN-PC1
ADMIN-PC2
STAFF-PC1
STAFF-PC2
SRV1
```

SW1 acts as the central internal switch.

---

### Floor 2

Contains:

```text
SW2
STAFF-PC3
STAFF-PC4
GUEST-PC1
GUEST-PC2
```

SW2 connects to SW1 through an 802.1Q trunk.

---

## Network Devices

| Device | Model / Type | Purpose |
|---|---|---|
| R1 | Cisco 2911 | Company edge router and inter-VLAN router |
| ISP | Cisco 2911 | Simulated ISP |
| SW1 | Cisco 2960-24TT | Core/internal switch |
| SW2 | Cisco 2960-24TT | Floor 2 access switch |
| SRV1 | Server-PT | DNS and internal HTTP server |
| INTERNET-SRV | Server-PT | Simulated external Internet server |

---

## End Devices

### Administration

```text
ADMIN-PC1
ADMIN-PC2
```

Both connect to SW1.

---

### Staff

```text
STAFF-PC1
STAFF-PC2
STAFF-PC3
STAFF-PC4
```

STAFF-PC1 and STAFF-PC2 connect to SW1.

STAFF-PC3 and STAFF-PC4 connect to SW2.

All four remain members of VLAN 20.

---

### Guest

```text
GUEST-PC1
GUEST-PC2
```

Both connect to SW2 and belong to VLAN 30.

---

## Internal Router Connection

R1 connects to SW1 using:

```text
R1 G0/0
   │
   │
SW1 Fa0/24
```

The link is an 802.1Q trunk carrying:

```text
VLAN 10
VLAN 20
VLAN 30
VLAN 40
```

R1 performs Router-on-a-Stick routing through:

```text
G0/0.10
G0/0.20
G0/0.30
G0/0.40
```

---

## Inter-Switch Connection

SW1 and SW2 are connected through:

```text
SW1 Fa0/23
     │
     │ TRUNK
     │
SW2 Fa0/24
```

Allowed VLANs:

```text
VLAN 20
VLAN 30
```

This allows STAFF devices on both floors to remain in the same VLAN.

---

## WAN Connection

R1 connects to the simulated ISP using:

```text
R1 G0/1
203.0.113.1/30
     │
     │
203.0.113.2/30
ISP G0/0
```

Network:

```text
203.0.113.0/30
```

R1 uses:

```text
203.0.113.2
```

as its next hop for the default route.

---

## Simulated Internet Segment

The ISP connects to the external server through:

```text
ISP G0/1
198.51.100.1
     │
     │
198.51.100.10
INTERNET-SRV
```

Network:

```text
198.51.100.0/24
```

This segment is used to test:

- Default routing
- WAN connectivity
- NAT
- PAT
- Internet-style HTTP access

---

## Traffic Flow — ADMIN to Server

```text
ADMIN-PC1
   │
VLAN 10
   │
SW1
   │
R1 G0/0.10
   │
Inter-VLAN Routing
   │
R1 G0/0.40
   │
SW1
   │
SRV1
```

Expected result:

```text
ALLOWED
```

---

## Traffic Flow — STAFF to Server

```text
STAFF-PC3
   │
SW2
   │
VLAN 20
   │
TRUNK
   │
SW1
   │
R1
   │
VLAN 40
   │
SRV1
```

Expected result:

```text
ALLOWED
```

---

## Traffic Flow — GUEST to Server

```text
GUEST-PC1
   │
VLAN 30
   │
SW2
   │
SW1
   │
R1
   │
ACL
   X
SRV1
```

Expected result:

```text
BLOCKED
```

DNS traffic remains permitted.

---

## Traffic Flow — GUEST to Internet

```text
GUEST-PC1
   │
VLAN 30
   │
SW2
   │
SW1
   │
R1
   │
ACL
   │
NAT/PAT
   │
ISP
   │
INTERNET-SRV
```

Expected result:

```text
ALLOWED
```

---

## NAT Boundary

R1 acts as the NAT boundary.

Internal side:

```text
192.168.10.0/24
192.168.20.0/24
192.168.30.0/24
192.168.40.0/24
```

External side:

```text
203.0.113.1
```

Multiple internal clients share the R1 WAN address through PAT.

---

## Topology Design Goals

The topology demonstrates:

- Two-floor office design
- Core and access switching
- VLAN segmentation
- VLAN extension across switches
- Router-on-a-Stick
- Inter-VLAN routing
- Dedicated server segment
- Guest isolation
- DHCP services
- DNS services
- HTTP services
- ACL-based security
- Simulated ISP
- Default routing
- NAT/PAT
- External connectivity