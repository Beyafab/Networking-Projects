# CCNA Lab: Configuring and Applying Standard Named ACLs

> **CCNA Practice Lab | Standard Named ACLs | Cisco IOS | IPv4**

## 📌 Overview

This lab demonstrates how to create and apply a **standard named Access Control List (ACL)** on a Cisco router to control traffic based on **source IPv4 addresses**.

The lab uses a point-to-point serial connection between **R1 and R3**, with three loopback interfaces configured on R3. An ACL is applied inbound on R1's serial interface to selectively allow or deny traffic from those loopback networks.

### What you'll practice

* Configuring Cisco router interfaces
* Configuring a DCE serial connection
* Assigning IPv4 addresses
* Configuring static default routes
* Creating loopback interfaces
* Creating a **standard named ACL**
* Using wildcard masks
* Applying an ACL to an interface
* Verifying ACL behavior
* Reading ACL match counters
* Troubleshooting connectivity after ACL deployment

---

## 🧩 Topology

```text
                 Serial Link
        172.16.1.0/26
   DCE                    DTE
┌─────────┐             ┌─────────┐
│   R1    │─────────────│   R3    │
│         │             │         │
│ S0/0    │             │ S0/1    │
│ .1      │             │ .2      │
└─────────┘             └────┬────┘
                             │
                  ┌──────────┼──────────┐
                  │          │          │
               Lo10        Lo20       Lo30
           10.10.10.3  10.20.20.3  10.30.30.3
```

---

## 🌐 IP Addressing

| Device | Interface  | IP Address   | Subnet Mask | Role             |
| ------ | ---------- | ------------ | ----------- | ---------------- |
| R1     | Serial0/0  | `172.16.1.1` | `/26`       | DCE              |
| R3     | Serial0/1  | `172.16.1.2` | `/26`       | DTE              |
| R3     | Loopback10 | `10.10.10.3` | `/25`       | Source Network 1 |
| R3     | Loopback20 | `10.20.20.3` | `/28`       | Source Network 2 |
| R3     | Loopback30 | `10.30.30.3` | `/29`       | Source Network 3 |

### ⚠️ Configuration Note

The original lab transcript contains interface/IP inconsistencies. The working topology used in this repository is:

```text
R1 Serial0/0 = 172.16.1.1/26
R3 Serial0/1 = 172.16.1.2/26
```

The R3 serial interface is therefore **Serial0/1** in this implementation.

---

# 1. Configure Hostnames

## R1

```cisco
enable
configure terminal
hostname R1
end
```

## R3

```cisco
enable
configure terminal
hostname R3
end
```

---

# 2. Configure the Serial Link

## R1

R1 is the **DCE** side and provides clocking.

```cisco
configure terminal
interface serial 0/0
 ip address 172.16.1.1 255.255.255.192
 clock rate 768000
 no shutdown
exit
end
```

## R3

```cisco
configure terminal
interface serial 0/1
 ip address 172.16.1.2 255.255.255.192
 no shutdown
exit
end
```

### Verify the interfaces

```cisco
show ip interface brief
```

The serial interfaces should be:

```text
up/up
```

---

# 3. Configure Static Routes

## R1

Configure a default route pointing toward R3:

```cisco
configure terminal
ip route 0.0.0.0 0.0.0.0 serial0/0 172.16.1.2
end
```

## R3

Configure a default route pointing toward R1:

```cisco
configure terminal
ip route 0.0.0.0 0.0.0.0 serial0/1 172.16.1.1
end
```

> **Note:** The static routes are primarily used to provide reachability between the two routers and their loopback networks.

Verify:

```cisco
show ip route
```

---

# 4. Configure R3 Loopback Interfaces

### Loopback10

```cisco
configure terminal
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
end
```

Verify:

```cisco
show ip interface brief
```

---

# 5. Verify Connectivity Before Applying the ACL

Before applying any ACL, make sure the network works normally.

From R3:

```cisco
ping 172.16.1.1
```

Test using each loopback as the source:

```cisco
ping 172.16.1.1 source loopback 10
ping 172.16.1.1 source loopback 20
ping 172.16.1.1 source loopback 30
```

### Expected result

All tests should succeed:

```text
!!!!!
```

This step is important.

If connectivity does not work **before** applying the ACL, troubleshoot routing and interfaces first rather than troubleshooting the ACL.

---

# 6. Create the Standard Named ACL

The ACL will implement the following policy:

| Source Network  | Action   |
| --------------- | -------- |
| `10.10.10.0/25` | ❌ Deny   |
| `10.20.20.0/28` | ✅ Permit |
| `10.30.30.0/29` | ❌ Deny   |
| `172.16.1.0/26` | ✅ Permit |

Create the ACL on R1:

```cisco
configure terminal

ip access-list standard LOOPBACK-10-30-ACL

 remark Deny Traffic From R3 Loopback10
 deny 10.10.10.0 0.0.0.127

 remark Permit Traffic From R3 Loopback20
 permit 10.20.20.0 0.0.0.15

 remark Deny Traffic From R3 Loopback30
 deny 10.30.30.0 0.0.0.7

 remark Permit Traffic From Serial Subnet
 permit 172.16.1.0 0.0.0.63

exit
```

---

# 7. Apply the ACL to R1

The ACL is applied **inbound** on R1's Serial0/0 interface:

```cisco
configure terminal

interface serial 0/0
 ip access-group LOOPBACK-10-30-ACL in

end
```

Verify:

```cisco
show running-config interface serial 0/0
```

You should see:

```text
ip access-group LOOPBACK-10-30-ACL in
```

---

# 8. Test the ACL

Run the same ping tests again.

### Directly from R3

```cisco
ping 172.16.1.1
```

Expected:

```text
!!!!!
```

The serial subnet is explicitly permitted.

### From Loopback10

```cisco
ping 172.16.1.1 source loopback 10
```

Expected:

```text
U.U.U
```

Traffic from `10.10.10.0/25` is denied.

### From Loopback20

```cisco
ping 172.16.1.1 source loopback 20
```

Expected:

```text
!!!!!
```

Traffic from `10.20.20.0/28` is permitted.

### From Loopback30

```cisco
ping 172.16.1.1 source loopback 30
```

Expected:

```text
U.U.U
```

Traffic from `10.30.30.0/29` is denied.

---

# 9. Expected Results

| Source       | Destination  | Expected Result | ACL Rule             |
| ------------ | ------------ | --------------: | -------------------- |
| `172.16.1.2` | `172.16.1.1` |       ✅ Allowed | Permit serial subnet |
| `10.10.10.3` | `172.16.1.1` |        ❌ Denied | Deny Loopback10      |
| `10.20.20.3` | `172.16.1.1` |       ✅ Allowed | Permit Loopback20    |
| `10.30.30.3` | `172.16.1.1` |        ❌ Denied | Deny Loopback30      |

---

# 10. Verify the ACL

### View the ACL

```cisco
show ip access-lists LOOPBACK-10-30-ACL
```

You should see something similar to:

```text
Standard IP access list LOOPBACK-10-30-ACL
    10 deny 10.10.10.0 0.0.0.127
    20 permit 10.20.20.0 0.0.0.15
    30 deny 10.30.30.0 0.0.0.7
    40 permit 172.16.1.0 0.0.0.63
```

The ACL will also show **match counters** after traffic has been generated.

For example:

```text
10 deny 10.10.10.0 0.0.0.127 (5 matches)
20 permit 10.20.20.0 0.0.0.15 (5 matches)
30 deny 10.30.30.0 0.0.0.7 (5 matches)
40 permit 172.16.1.0 0.0.0.63 (5 matches)
```

The exact counters will depend on how many packets you generate.

---

# 11. Useful Verification Commands

### Check interfaces

```cisco
show ip interface brief
```

### Check routing table

```cisco
show ip route
```

### Check ACL

```cisco
show ip access-lists LOOPBACK-10-30-ACL
```

### Check interface ACL configuration

```cisco
show running-config interface serial 0/0
```

### Check how the ACL is applied

```cisco
show ip interface serial 0/0
```

---

# 🧠 Key CCNA Concepts

## Standard ACLs

Standard ACLs filter traffic based on the **source IP address**.

```text
Source IP → ACL → Permit/Deny
```

Unlike extended ACLs, standard ACLs do not directly match:

* Destination IP
* TCP/UDP ports
* Specific Layer 4 protocols

---

## ACL Processing Order

ACL entries are processed from **top to bottom**.

The first matching statement is used.

Example:

```cisco
deny 10.10.10.0 0.0.0.127
permit 10.20.20.0 0.0.0.15
```

Once traffic matches one of these entries, Cisco does not continue checking the remaining entries.

---

## Implicit Deny

Every ACL has an implicit deny at the end.

Conceptually:

```cisco
deny any
```

Therefore, traffic that does not match an explicit permit statement is denied.

This is why ACL design requires careful consideration of what traffic needs to be allowed.

---

# 🎯 Standard ACL Placement

A standard ACL should generally be placed **close to the destination**.

Why?

Because standard ACLs only examine the source address.

If a standard ACL is placed too close to the source, it may unintentionally block that source from reaching other destinations.

In this lab, the ACL is applied inbound on R1's Serial0/0 interface to control access to R1.

---

# 🧮 Wildcard Masks

Wildcard masks are commonly used with Cisco ACLs.

A simple way to calculate a wildcard mask is:

```text
Wildcard Mask = 255.255.255.255 - Subnet Mask
```

| Network         | Subnet Mask       | Wildcard Mask |
| --------------- | ----------------- | ------------- |
| `10.10.10.0/25` | `255.255.255.128` | `0.0.0.127`   |
| `10.20.20.0/28` | `255.255.255.240` | `0.0.0.15`    |
| `10.30.30.0/29` | `255.255.255.248` | `0.0.0.7`     |
| `172.16.1.0/26` | `255.255.255.192` | `0.0.0.63`    |

### Quick memory trick

```text
0 = must match
1 = ignore
```

For example:

```cisco
10.10.10.0 0.0.0.127
```

means:

```text
10.10.10.x
```

where the last octet can vary from `0` to `127`.

---

# 🔍 Troubleshooting

If the expected results are not appearing, check the following.

### 1. Check interface status

```cisco
show ip interface brief
```

Make sure the serial interfaces are `up/up`.

### 2. Check routing

```cisco
show ip route
```

Make sure R1 and R3 have routes to the required networks.

### 3. Check the ACL

```cisco
show ip access-lists LOOPBACK-10-30-ACL
```

Check the source networks and wildcard masks carefully.

### 4. Check ACL application

```cisco
show ip interface serial 0/0
```

Confirm:

```text
Inbound access list is LOOPBACK-10-30-ACL
```

### 5. Check match counters

Generate traffic and then run:

```cisco
show ip access-lists LOOPBACK-10-30-ACL
```

If a rule has no matches, investigate the source address and traffic path.

---

# ⚠️ Common Mistakes

### Wrong wildcard mask

Incorrect:

```cisco
deny 10.10.10.0 255.255.255.128
```

Correct:

```cisco
deny 10.10.10.0 0.0.0.127
```

---

### Applying the ACL in the wrong direction

The lab requires:

```cisco
ip access-group LOOPBACK-10-30-ACL in
```

not:

```cisco
ip access-group LOOPBACK-10-30-ACL out
```

---

### Forgetting the implicit deny

If you only configure:

```cisco
deny 10.10.10.0 0.0.0.127
deny 10.30.30.0 0.0.0.7
```

then other traffic may also be denied because of the implicit deny.

---

### Testing before checking basic connectivity

Always establish this sequence:

```text
Interface
    ↓
IP addressing
    ↓
Routing
    ↓
Connectivity
    ↓
ACL
    ↓
ACL verification
```

This makes troubleshooting much faster.

---

# 📝 Lab Summary

This lab demonstrates how a **standard named ACL** can be used to control traffic based on source IPv4 addresses.

The final policy is:

```text
10.10.10.0/25  → DENY
10.20.20.0/28  → PERMIT
10.30.30.0/29  → DENY
172.16.1.0/26  → PERMIT
```

The ACL is applied inbound on R1's Serial0/0 interface:

```cisco
ip access-group LOOPBACK-10-30-ACL in
```

The lab reinforces several important CCNA concepts:

* Standard ACLs
* Named ACLs
* Wildcard masks
* ACL processing order
* Implicit deny
* ACL placement
* Interface ACL application
* ACL verification
* Network troubleshooting

---

## 🚀 Skills Practiced

`Cisco IOS` · `CCNA` · `IPv4` · `Standard ACL` · `Named ACL` · `Wildcard Masks` · `Static Routing` · `Serial Networking` · `Loopback Interfaces` · `Network Troubleshooting`

---

## 📚 Related Lab

**Lab 61: Restricting Inbound Telnet Access Using Extended ACLs**

That lab builds on these concepts by using an **extended named ACL** to filter Telnet traffic based on both source IP and TCP port.
