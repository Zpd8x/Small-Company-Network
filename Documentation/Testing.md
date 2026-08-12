# Network Testing & Verification

This document contains the final testing and verification procedures for the **Small Company Network** project.

The purpose of these tests is to confirm that VLANs, trunks, inter-VLAN routing, DHCP, DNS, HTTP services, ACL security, WAN connectivity, and NAT/PAT are working correctly.

## Test Environment

Internal networks:

```text
ADMIN   → 192.168.10.0/24
STAFF   → 192.168.20.0/24
GUEST   → 192.168.30.0/24
SERVER  → 192.168.40.0/24
```

WAN:

```text
R1  → 203.0.113.1
ISP → 203.0.113.2
```

Simulated Internet:

```text
INTERNET-SRV → 198.51.100.10
```

---

## 1. Interface Verification

### R1

Command:

```cisco
show ip interface brief
```

Expected interfaces:

```text
G0/0       → unassigned       up/up

G0/0.10    → 192.168.10.1     up/up
G0/0.20    → 192.168.20.1     up/up
G0/0.30    → 192.168.30.1     up/up
G0/0.40    → 192.168.40.1     up/up

G0/1       → 203.0.113.1      up/up
```

Result:

```text
PASS
```

---

## 2. VLAN Verification

### SW1

Command:

```cisco
show vlan brief
```

Expected:

```text
VLAN 10 ADMIN
Fa0/1
Fa0/2

VLAN 20 STAFF
Fa0/3
Fa0/4

VLAN 40 SERVER
Fa0/5
```

### SW2

Command:

```cisco
show vlan brief
```

Expected:

```text
VLAN 20 STAFF
Fa0/1
Fa0/2

VLAN 30 GUEST
Fa0/3
Fa0/4
```

Result:

```text
PASS
```

---

## 3. Trunk Verification

### SW1

Command:

```cisco
show interfaces trunk
```

Expected:

```text
Fa0/23 → trunking
Allowed VLANs: 20,30

Fa0/24 → trunking
Allowed VLANs: 10,20,30,40
```

### SW2

Expected:

```text
Fa0/24 → trunking
Allowed VLANs: 20,30
```

Result:

```text
PASS
```

---

## 4. Router-on-a-Stick Verification

Command:

```cisco
show running-config
```

Expected R1 subinterfaces:

```cisco
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0

interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
```

Result:

```text
PASS
```

---

## 5. Routing Verification

Command:

```cisco
show ip route
```

Expected connected networks:

```text
192.168.10.0/24
192.168.20.0/24
192.168.30.0/24
192.168.40.0/24
203.0.113.0/30
```

Expected default route:

```text
S* 0.0.0.0/0 via 203.0.113.2
```

Result:

```text
PASS
```

---

## 6. DHCP Verification

Command on R1:

```cisco
show ip dhcp binding
```

Expected address ranges:

```text
ADMIN → 192.168.10.x
STAFF → 192.168.20.x
GUEST → 192.168.30.x
```

Expected DHCP information on each client:

```text
Subnet Mask
Default Gateway
DNS Server
```

Expected DNS server:

```text
192.168.40.10
```

Result:

```text
PASS
```

---

## 7. Internal Server Verification

SRV1 configuration:

```text
IP Address:       192.168.40.10
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.40.1
DNS Server:       192.168.40.10
```

Services:

```text
DNS  → ON
HTTP → ON
```

Result:

```text
PASS
```

---

## 8. DNS Verification

From ADMIN:

```text
ping server.company.local
```

Expected resolution:

```text
server.company.local → 192.168.40.10
```

From STAFF:

```text
ping server.company.local
```

Expected:

```text
DNS resolution successful
Ping successful
```

From GUEST:

```text
ping server.company.local
```

Expected:

```text
server.company.local resolves to 192.168.40.10

Request timed out.
```

This confirms:

```text
DNS access       → ALLOWED
Direct ICMP      → BLOCKED
```

Result:

```text
PASS
```

---

## 9. Internal HTTP Verification

From ADMIN:

```text
http://www.company.local
```

Expected:

```text
ACCESS ALLOWED
```

From STAFF:

```text
http://www.company.local
```

Expected:

```text
ACCESS ALLOWED
```

From GUEST:

```text
http://www.company.local
```

Expected:

```text
ACCESS BLOCKED
```

Result:

```text
PASS
```

---

## 10. ADMIN Connectivity Tests

From ADMIN-PC1:

```text
ping <STAFF-PC-IP>
```

Expected:

```text
SUCCESS
```

Test Guest:

```text
ping <GUEST-PC-IP>
```

Expected:

```text
SUCCESS
```

Test server:

```text
ping 192.168.40.10
```

Expected:

```text
SUCCESS
```

Test Internet server:

```text
ping 198.51.100.10
```

Expected:

```text
SUCCESS
```

Result:

```text
PASS
```

---

## 11. STAFF Security Tests

From STAFF:

### STAFF → ADMIN

```text
ping <ADMIN-PC-IP>
```

Expected:

```text
Request timed out.
```

### STAFF → GUEST

```text
ping <GUEST-PC-IP>
```

Expected:

```text
Request timed out.
```

### STAFF → SERVER

```text
ping 192.168.40.10
```

Expected:

```text
SUCCESS
```

### STAFF → INTERNET

```text
ping 198.51.100.10
```

Expected:

```text
SUCCESS
```

Result:

```text
PASS
```

---

## 12. GUEST Security Tests

From GUEST-PC1:

### Gateway

```text
ping 192.168.30.1
```

Expected:

```text
SUCCESS
```

### GUEST → ADMIN

```text
ping <ADMIN-PC-IP>
```

Expected:

```text
Request timed out.
```

### GUEST → STAFF

```text
ping <STAFF-PC-IP>
```

Expected:

```text
Request timed out.
```

### GUEST → SERVER

```text
ping 192.168.40.10
```

Expected:

```text
Request timed out.
```

### DNS

```text
ping server.company.local
```

Expected:

```text
Name resolves to 192.168.40.10
Ping request times out
```

### Internet

```text
ping 198.51.100.10
```

Expected:

```text
SUCCESS
```

Result:

```text
PASS
```

---

## 13. ACL Verification

Command:

```cisco
show access-lists
```

Expected ACLs:

```text
STAFF_IN
GUEST_IN
```

Packet counters should increase when traffic matches permit or deny rules.

Verify application:

```cisco
show ip interface GigabitEthernet0/0.20
show ip interface GigabitEthernet0/0.30
```

Expected:

```text
G0/0.20
Inbound access list is STAFF_IN

G0/0.30
Inbound access list is GUEST_IN
```

Result:

```text
PASS
```

---

## 14. Silent Drop Verification

Restricted VLAN subinterfaces use:

```cisco
no ip unreachables
```

Blocked Ping requests should display:

```text
Request timed out.
```

instead of:

```text
Destination host unreachable.
```

Result:

```text
PASS
```

---

## 15. WAN Verification

From R1:

```text
ping 203.0.113.2
```

Expected:

```text
SUCCESS
```

Then:

```text
ping 198.51.100.1
```

Expected:

```text
SUCCESS
```

Finally:

```text
ping 198.51.100.10
```

Expected:

```text
SUCCESS
```

Result:

```text
PASS
```

---

## 16. Default Route Verification

Command:

```cisco
show ip route
```

Expected:

```text
Gateway of last resort is 203.0.113.2
```

and:

```text
S* 0.0.0.0/0 via 203.0.113.2
```

Result:

```text
PASS
```

---

## 17. NAT/PAT Verification

Generate traffic from multiple internal clients:

```text
ADMIN → 198.51.100.10
STAFF → 198.51.100.10
GUEST → 198.51.100.10
```

Then on R1:

```cisco
show ip nat translations
```

Expected behavior:

```text
192.168.10.x
192.168.20.x
192.168.30.x

        ↓

203.0.113.1
```

Multiple private addresses should share the R1 WAN address using PAT.

Additional command:

```cisco
show ip nat statistics
```

Expected:

```text
Inside:
G0/0.10
G0/0.20
G0/0.30
G0/0.40

Outside:
G0/1
```

Result:

```text
PASS
```

---

## Final Test Matrix

| Test | Expected Result | Status |
|---|---|---|
| VLAN Segmentation | Working | PASS |
| Trunk SW1 ↔ SW2 | Working | PASS |
| Trunk SW1 ↔ R1 | Working | PASS |
| Router-on-a-Stick | Working | PASS |
| Inter-VLAN Routing | Working | PASS |
| DHCP ADMIN | Working | PASS |
| DHCP STAFF | Working | PASS |
| DHCP GUEST | Working | PASS |
| DNS | Working | PASS |
| Internal HTTP | Working | PASS |
| ADMIN → STAFF | Allowed | PASS |
| ADMIN → GUEST | Allowed | PASS |
| ADMIN → SERVER | Allowed | PASS |
| STAFF → ADMIN | Blocked | PASS |
| STAFF → GUEST | Blocked | PASS |
| STAFF → SERVER | Allowed | PASS |
| GUEST → ADMIN | Blocked | PASS |
| GUEST → STAFF | Blocked | PASS |
| GUEST → SERVER | Blocked | PASS |
| GUEST → DNS | Allowed | PASS |
| WAN R1 ↔ ISP | Working | PASS |
| Default Route | Working | PASS |
| NAT/PAT | Working | PASS |
| ADMIN → Internet | Allowed | PASS |
| STAFF → Internet | Allowed | PASS |
| GUEST → Internet | Allowed | PASS |

---

## Final Result

```text
ALL NETWORK TESTS PASSED
```

The Small Company Network successfully demonstrates:

- VLAN segmentation
- Multi-switch trunking
- Router-on-a-Stick
- Inter-VLAN routing
- DHCP
- DNS
- HTTP
- Extended ACLs
- Directional access control
- Guest isolation
- Silent packet filtering
- Static routing
- WAN connectivity
- NAT/PAT
- Simulated Internet access