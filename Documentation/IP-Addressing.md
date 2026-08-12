# IP Addressing Plan

This document describes the IPv4 addressing scheme used in the **Small Company Network** project.

## VLAN Networks

| VLAN | Name | Network | Subnet Mask | Default Gateway |
|---:|---|---|---|---|
| 10 | ADMIN | `192.168.10.0/24` | `255.255.255.0` | `192.168.10.1` |
| 20 | STAFF | `192.168.20.0/24` | `255.255.255.0` | `192.168.20.1` |
| 30 | GUEST | `192.168.30.0/24` | `255.255.255.0` | `192.168.30.1` |
| 40 | SERVER | `192.168.40.0/24` | `255.255.255.0` | `192.168.40.1` |

---

## Router R1

R1 performs inter-VLAN routing using Router-on-a-Stick.

| Interface | Purpose | IP Address |
|---|---|---|
| `G0/0` | Physical trunk toward SW1 | Unassigned |
| `G0/0.10` | VLAN 10 ADMIN Gateway | `192.168.10.1/24` |
| `G0/0.20` | VLAN 20 STAFF Gateway | `192.168.20.1/24` |
| `G0/0.30` | VLAN 30 GUEST Gateway | `192.168.30.1/24` |
| `G0/0.40` | VLAN 40 SERVER Gateway | `192.168.40.1/24` |
| `G0/1` | WAN connection to ISP | `203.0.113.1/30` |

---

## DHCP Addressing

R1 provides DHCP services for ADMIN, STAFF, and GUEST networks.

### ADMIN

```text
Network:          192.168.10.0/24
Gateway:          192.168.10.1
DNS Server:       192.168.40.10
Reserved Range:   192.168.10.1 - 192.168.10.20
DHCP Start:       192.168.10.21
```

### STAFF

```text
Network:          192.168.20.0/24
Gateway:          192.168.20.1
DNS Server:       192.168.40.10
Reserved Range:   192.168.20.1 - 192.168.20.20
DHCP Start:       192.168.20.21
```

### GUEST

```text
Network:          192.168.30.0/24
Gateway:          192.168.30.1
DNS Server:       192.168.40.10
Reserved Range:   192.168.30.1 - 192.168.30.20
DHCP Start:       192.168.30.21
```

---

## Internal Server

The company server uses a static IP address.

| Device | IP Address | Gateway | DNS |
|---|---|---|---|
| SRV1 | `192.168.40.10` | `192.168.40.1` | `192.168.40.10` |

SRV1 provides:

- DNS
- Internal HTTP

---

## Internal DNS Records

| Hostname | Address |
|---|---|
| `company.local` | `192.168.40.10` |
| `www.company.local` | `192.168.40.10` |
| `server.company.local` | `192.168.40.10` |

---

## WAN Network

R1 connects to the simulated ISP using a `/30` point-to-point network.

```text
WAN Network: 203.0.113.0/30
```

| Device | Interface | Address |
|---|---|---|
| R1 | `G0/1` | `203.0.113.1/30` |
| ISP | `G0/0` | `203.0.113.2/30` |

R1 default route:

```text
0.0.0.0/0 → 203.0.113.2
```

---

## Simulated Internet Network

```text
Network: 198.51.100.0/24
```

| Device | Address | Purpose |
|---|---|---|
| ISP | `198.51.100.1` | External network gateway |
| INTERNET-SRV | `198.51.100.10` | Simulated Internet server |

INTERNET-SRV configuration:

```text
IP Address:       198.51.100.10
Subnet Mask:      255.255.255.0
Default Gateway:  198.51.100.1
```

---

## Network Summary

```text
192.168.10.0/24   ADMIN
192.168.20.0/24   STAFF
192.168.30.0/24   GUEST
192.168.40.0/24   SERVER

203.0.113.0/30    R1 ↔ ISP

198.51.100.0/24   Simulated Internet
```