# Network Security Policy

This document describes the access-control policy implemented in the **Small Company Network** project.

Network segmentation is implemented with VLANs and traffic between internal networks is controlled by Extended Access Control Lists on R1.

## Security Zones

The network contains four security zones:

| VLAN | Zone | Security Level |
|---:|---|---|
| 10 | ADMIN | High |
| 20 | STAFF | Medium |
| 30 | GUEST | Restricted |
| 40 | SERVER | Protected |

---

## Access Matrix

| Source | ADMIN | STAFF | GUEST | SERVER | Internet |
|---|---:|---:|---:|---:|---:|
| ADMIN | — | ✅ | ✅ | ✅ | ✅ |
| STAFF | ❌ | — | ❌ | ✅ | ✅ |
| GUEST | ❌ | ❌ | — | ❌ | ✅ |

Guest users have one exception:

```text
GUEST → DNS Server : ALLOWED
```

Only DNS traffic toward `192.168.40.10` is permitted.

---

## ADMIN Policy

ADMIN has the highest level of internal access.

Allowed:

```text
ADMIN → STAFF
ADMIN → GUEST
ADMIN → SERVER
ADMIN → Internet
```

The ADMIN VLAN is not restricted by an inbound ACL in this lab.

This allows administrators to perform network management and troubleshooting operations across internal networks.

---

## STAFF Policy

STAFF users are allowed to access company server resources and the Internet.

Allowed:

```text
STAFF → SERVER
STAFF → Internet
```

Blocked:

```text
STAFF → ADMIN
STAFF → GUEST
```

The STAFF ACL is applied inbound on:

```text
GigabitEthernet0/0.20
```

ICMP echo replies toward ADMIN are permitted so that ADMIN can initiate connectivity tests toward STAFF while STAFF remains unable to initiate Ping requests toward ADMIN.

---

## GUEST Policy

The GUEST VLAN is isolated from company internal networks.

Blocked:

```text
GUEST → ADMIN
GUEST → STAFF
GUEST → SERVER
```

Allowed:

```text
GUEST → DNS
GUEST → Internet
```

The GUEST ACL is applied inbound on:

```text
GigabitEthernet0/0.30
```

---

## Guest DNS Exception

The Guest network uses the internal DNS server:

```text
192.168.40.10
```

Only DNS traffic is permitted:

```text
UDP/53
TCP/53
```

This allows Guest clients to resolve domain names without giving them general access to the SERVER VLAN.

Example expected behavior:

```text
server.company.local → resolves to 192.168.40.10 ✅

Ping 192.168.40.10                           ❌
HTTP http://www.company.local                ❌
```

---

## ICMP Behavior

Blocked traffic is configured to behave like a silent firewall drop.

Instead of displaying:

```text
Destination host unreachable
```

clients receive:

```text
Request timed out.
```

This is configured using:

```cisco
no ip unreachables
```

on the restricted VLAN subinterfaces.

---

## STAFF ACL Logic

The STAFF policy follows this order:

```text
1. Allow required return ICMP traffic toward ADMIN.
2. Allow STAFF access to SERVER.
3. Deny STAFF access to ADMIN.
4. Deny STAFF access to GUEST.
5. Allow remaining traffic, including Internet access.
```

ACL processing is performed from top to bottom, and the first matching rule is applied.

---

## GUEST ACL Logic

The GUEST policy follows this order:

```text
1. Allow DHCP.
2. Allow DNS to 192.168.40.10.
3. Allow required ICMP echo replies toward ADMIN.
4. Deny access to ADMIN.
5. Deny access to STAFF.
6. Deny access to SERVER.
7. Allow remaining traffic toward the simulated Internet.
```

The final permit rule is intentionally placed after all internal deny statements.

---

## Network Address Translation

Internal devices use private IPv4 networks:

```text
192.168.10.0/24
192.168.20.0/24
192.168.30.0/24
192.168.40.0/24
```

PAT translates permitted internal connections to the R1 WAN address:

```text
203.0.113.1
```

This allows multiple internal hosts to share the same WAN address.

---

## Expected Security Tests

| Test | Expected Result |
|---|---|
| ADMIN → STAFF | Allowed |
| ADMIN → GUEST | Allowed |
| ADMIN → SERVER | Allowed |
| STAFF → ADMIN | Blocked |
| STAFF → GUEST | Blocked |
| STAFF → SERVER | Allowed |
| GUEST → ADMIN | Blocked |
| GUEST → STAFF | Blocked |
| GUEST → SERVER | Blocked |
| GUEST → DNS | Allowed |
| ADMIN → Internet | Allowed |
| STAFF → Internet | Allowed |
| GUEST → Internet | Allowed |

---

## Security Objectives

The implemented policy demonstrates:

- Network segmentation
- VLAN isolation
- Least-privilege access
- Extended ACL filtering
- Controlled DNS access
- Guest network isolation
- Protected server access
- Directional ICMP control
- WAN edge filtering
- NAT/PAT
- Basic firewall-style traffic control