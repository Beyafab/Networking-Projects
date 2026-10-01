# Lab 61: Restricting Inbound Telnet Access Using Extended ACLs

## Overview

This lab demonstrates how to use an **extended named Access Control List (ACL)** to control inbound Telnet access to a Cisco router.

The lab focuses on applying an ACL directly to the **VTY lines** using the `access-class` command. This allows the router administrator to control which source hosts are permitted or denied when attempting to establish Telnet sessions with the router.

This is an important CCNA-level networking skill because VTY access is used for remote management of Cisco devices.

---

## Objectives

By completing this lab, you will learn how to:

* Configure Cisco router hostnames.
* Configure a point-to-point serial connection.
* Configure a DCE clock rate.
* Assign IP addresses to serial interfaces.
* Configure a static default route.
* Configure loopback interfaces.
* Configure Telnet access on VTY lines.
* Create an extended named ACL.
* Permit and deny Telnet traffic based on source IP addresses.
* Apply an ACL to VTY lines using `access-class`.
* Test Telnet access using a specific source interface.
* Verify ACL matches using `show ip access-lists`.

---

## Topology

```text
                  Serial 0/0
          172.16.1.1/26
              ┌─────────┐
              │   R1    │
              │         │
              └────┬────┘
                   │
                   │ 172.16.1.0/26
                   │
              ┌────┴────┐
              │   R3    │
              │         │
              └─────────┘
                │   │   │
                │   │   │
              Lo10 Lo20 Lo30
             .3    .3    .3

        10.10.10.3
        10.20.20.3
        10.30.30.3
```

### Addressing

| Device | Interface  | IP Address | Mask            |
| ------ | ---------- | ---------- | --------------- |
| R1     | S0/0       | 172.16.1.1 | 255.255.255.192 |
| R3     | S0/0       | 172.16.1.2 | 255.255.255.192 |
| R3     | Loopback10 | 10.10.10.3 | 255.255.255.128 |
| R3     | Loopback20 | 10.20.20.3 | 255.255.255.240 |
| R3     | Loopback30 | 10.30.30.3 | 255.255.255.248 |

---

# Configuration

## 1. Configure Hostnames

### R1

```cisco
enable
configure terminal
hostname R1
```

### R3

```cisco
enable
configure terminal
hostname R3
```

---

## 2. Configure the Serial Connection

### R1

R1 is the DCE side of the serial connection.

```cisco
configure terminal
interface serial 0/0
ip address 172.16.1.1 255.255.255.192
clock rate 2000000
no shutdown
end
```

### R3

```cisco
configure terminal
interface serial 0/0
ip address 172.16.1.2 255.255.255.192
no shutdown
end
```

### Test the Serial Connection

From R1:

```cisco
ping 172.16.1.2
```

From R3:

```cisco
ping 172.16.1.1
```

Both tests should succeed.

---

# 3. Configure R1

## Static Default Route

Configure R1 to forward unknown destinations toward R3.

```cisco
configure terminal
ip route 0.0.0.0 0.0.0.0 serial 0/0 172.16.1.2
```

## Configure Telnet Access

```cisco
line vty 0 4
password CISCO
login
exit
```

---

# 4. Configure R3 Loopback Interfaces

### Loopback10

```cisco
interface loopback 10
ip address 10.10.10.3 255.255.255.128
exit
```

### Loopback20

```cisco
interface loopback 20
ip address 10.20.20.3 255.255.255.240
exit
```

### Loopback30

```cisco
interface loopback 30
ip address 10.30.30.3 255.255.255.248
exit
```

---

# 5. Test Connectivity

From R1, test each loopback:

```cisco
ping 10.10.10.3
ping 10.20.20.3
ping 10.30.30.3
```

All three addresses should be reachable.

---

# 6. Create the Extended Named ACL

The objective is:

| Source     | Telnet Access |
| ---------- | ------------- |
| 10.10.10.3 | Permit        |
| 10.20.20.3 | Deny          |
| 10.30.30.3 | Permit        |

Create the ACL on R1:

```cisco
configure terminal

ip access-list extended TELNET-IN

remark Permit Telnet From Host 10.10.10.3
permit tcp host 10.10.10.3 any eq 23

remark Deny Telnet From Host 10.20.20.3
deny tcp host 10.20.20.3 any eq 23

remark Permit Telnet From Host 10.30.30.3
permit tcp host 10.30.30.3 any eq 23

exit
```

---

# 7. Apply the ACL to the VTY Lines

```cisco
line vty 0 4
access-class TELNET-IN in
end
```

The important command is:

```cisco
access-class TELNET-IN in
```

This applies the ACL to inbound connections attempting to access the router through the VTY lines.

---

# 8. Test Telnet Access

From R3, test each loopback as the source.

### Loopback10

```cisco
telnet 172.16.1.1 /source-interface loopback 10
```

Expected result:

```text
Trying 172.16.1.1 ... Open
User Access Verification
Password:
```

**Result: ALLOWED**

---

### Loopback20

```cisco
telnet 172.16.1.1 /source-interface loopback 20
```

Expected result:

```text
Trying 172.16.1.1 ...
% Connection refused by remote host
```

**Result: DENIED**

---

### Loopback30

```cisco
telnet 172.16.1.1 /source-interface loopback 30
```

Expected result:

```text
Trying 172.16.1.1 ... Open
User Access Verification
Password:
```

**Result: ALLOWED**

---

# 9. Verify ACL Matches

Run:

```cisco
show ip access-lists TELNET-IN
```

Expected output:

```text
Extended IP access list TELNET-IN
    10 permit tcp host 10.10.10.3 any eq telnet
    20 deny tcp host 10.20.20.3 any eq telnet
    30 permit tcp host 10.30.30.3 any eq telnet
```

The ACL should also display match counters after the Telnet tests.

For example:

```text
10 permit tcp host 10.10.10.3 any eq telnet (2 matches)
20 deny tcp host 10.20.20.3 any eq telnet (1 match)
30 permit tcp host 10.30.30.3 any eq telnet (2 matches)
```

The match counters provide evidence that the ACL rules are actually being hit.

---

# Important Concept: `access-class` vs Interface ACL

One of the most important concepts in this lab is understanding where the ACL is applied.

### `access-class`

```cisco
line vty 0 4
access-class TELNET-IN in
```

This controls access to the router's **VTY lines**.

It is specifically useful for controlling remote management connections such as:

* Telnet
* SSH

### Interface ACL

An ACL applied to an interface looks like:

```cisco
interface serial 0/0
ip access-group ACL-NAME in
```

This controls traffic entering or leaving a network interface.

These two configurations serve different purposes.

---

# Key Commands

| Command                   | Purpose                                 |
| ------------------------- | --------------------------------------- |
| `show ip interface brief` | Check interface status and IP addresses |
| `show running-config`     | View current configuration              |
| `show ip route`           | View routing table                      |
| `show ip access-lists`    | View ACLs and match counters            |
| `show access-lists`       | Display configured ACLs                 |
| `ping`                    | Test IP connectivity                    |
| `telnet`                  | Test remote VTY access                  |
| `access-class`            | Apply an ACL to VTY lines               |
| `ip access-list extended` | Create a named extended ACL             |

---

# What This Lab Teaches

The main lesson is that **remote management access can be controlled separately from ordinary transit traffic**.

In this lab, R1 receives Telnet connection attempts from three different source addresses:

```text
10.10.10.3  → Permit
10.20.20.3  → Deny
10.30.30.3  → Permit
```

The ACL identifies the source address and TCP destination port 23 before deciding whether the VTY connection is allowed.

---

# Security Note

Telnet sends credentials and session data without encryption. In real production networks, **SSH should generally be used instead of Telnet** for secure remote management.

This lab uses Telnet because the purpose is to practice ACL and VTY-line configuration.

---

# Lab Result

The completed configuration demonstrates that:

* R1 and R3 can communicate through the serial link.
* R1 can reach all three R3 loopback addresses.
* R1 accepts Telnet from `10.10.10.3`.
* R1 rejects Telnet from `10.20.20.3`.
* R1 accepts Telnet from `10.30.30.3`.
* ACL hit counters can be used to verify the rules.

## Skills Practiced

**CCNA Networking | Cisco IOS | Extended ACLs | VTY Security | Telnet | TCP | IPv4 Routing | Serial Interfaces | Loopback Interfaces | Network Troubleshooting**
