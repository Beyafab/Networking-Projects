# README.md

# CCNA Lab: Default Routes, Loopbacks, and Standard ACLs

## Overview
This lab configures a serial link between R1 and R3, adds default routes, creates loopback interfaces on R3, and filters traffic with a standard ACL on R1.

## Topology
```
R1 (DCE) ----- Serial ----- R3
```

### Addressing
| Device | Interface | IP Address | Subnet Mask | Notes |
|---|---|---|---|---|
| R1 | Serial0/0 | 172.16.1.1 | 255.255.255.192 (/26) | DCE, clock rate 768000 |
| R3 | Serial0/1 | 172.16.1.2 | 255.255.255.192 (/26) | |
| R3 | Loopback10 | 10.10.10.3 | 255.255.255.128 (/25) | |
| R3 | Loopback20 | 10.20.20.3 | 255.255.255.240 (/28) | |
| R3 | Loopback30 | 10.30.30.3 | 255.255.255.248 (/29) | |

> **Note:** The provided transcript shows R1 configured with `172.16.1.2` and R3 also configured with `172.16.1.2`. This is a duplicate-IP typo. The working design uses R1 as `172.16.1.1` and R3 as `172.16.1.2`.

## Task 2: Basic Connectivity and Default Routes

### R1 Configuration
```cisco
conf t
int s0/0
 no shutdown
 clock rate 768000
 ip address 172.16.1.1 255.255.255.192
 exit
ip route 0.0.0.0 0.0.0.0 serial0/0 172.16.1.2
end
```

### R3 Configuration
```cisco
conf t
int s0/1
 ip address 172.16.1.2 255.255.255.192
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 serial0/1 172.16.1.1
int loo10
 ip address 10.10.10.3 255.255.255.128
 exit
int loo20
 ip address 10.20.20.3 255.255.255.240
 exit
int loo30
 ip address 10.30.30.3 255.255.255.248
 end
```

> **Note:** The transcript’s R3 default route references `serial0/0`, but the configured interface is `serial0/1`.

## Task 3: Verify Connectivity

Before applying the ACL, all pings should succeed.

```cisco
R3#ping 172.16.1.1
R3#ping 172.16.1.1 source loopback10
R3#ping 172.16.1.1 source loopback20
R3#ping 172.16.1.1 source loopback30
```

Expected result: `!!!!!` with 100% success.

## Task 4: Standard ACL on R1

### ACL Configuration
```cisco
R1#conf t
access-list 10 remark 'Permit From R3 Loopback10'
access-list 10 permit 10.10.10.0 0.0.0.127
access-list 10 remark 'Deny From R3 Loopback20'
access-list 10 deny 10.20.20.0 0.0.0.15
access-list 10 remark 'Permit From R3 Loopback30'
access-list 10 permit 10.30.30.0 0.0.0.7
int s0/0
 ip access-group 10 in
 end
```

### Verification
```cisco
R1#show ip access-lists
```

Expected ACL entries:
- Permit `10.10.10.0/25`
- Deny `10.20.20.0/28`
- Permit `10.30.30.0/29`
- All other traffic is implicitly denied.

### Ping Results After ACL
| Source | Destination | Result | Reason |
|---|---|---|---|
| Default (172.16.1.2) | 172.16.1.1 | `U.U.U` | Serial subnet not explicitly permitted |
| Loopback10 | 172.16.1.1 | `!!!!!` | Explicitly permitted |
| Loopback20 | 172.16.1.1 | `U.U.U` | Explicitly denied |
| Loopback30 | 172.16.1.1 | `!!!!!` | Explicitly permitted |

## Wildcard Mask Calculation
Wildcard mask = `255.255.255.255` − subnet mask.

| Subnet | Subnet Mask | Wildcard Mask |
|---|---|---|
| 10.10.10.0/25 | 255.255.255.128 | 0.0.0.127 |
| 10.20.20.0/28 | 255.255.255.240 | 0.0.0.15 |
| 10.30.30.0/29 | 255.255.255.248 | 0.0.0.7 |

## Key Concepts
- Standard ACLs match source IP addresses only.
- ACLs are processed top-down; the first match wins.
- There is an implicit deny at the end of every ACL.
- `U.U.U` usually means traffic was administratively prohibited by an ACL.
- Use `remark` entries to document ACL lines.
- Apply ACLs in the correct direction and on the correct interface.
- If traffic is not explicitly permitted, it is implicitly denied.

## Files
- `README.md` — Lab overview, commands, and verification.
- `tips_and_challengs.txt` — Tips, pitfalls, and challenges.
