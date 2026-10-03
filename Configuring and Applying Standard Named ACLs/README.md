# README.md

# CCNA Lab: Configuring and Applying Standard Named ACLs

## Overview
This lab focuses on creating and applying **standard named ACLs** to filter traffic based on source IP addresses. Standard ACLs should be placed close to the destination. The lab uses a serial link between R1 and R3, loopback interfaces on R3, and a named ACL on R1 to control inbound traffic from R3.

## Topology
R1 (DCE) ----- Serial ----- R3

text

### Addressing
| Device | Interface   | IP Address  | Subnet Mask       | Notes                     |
|--------|-------------|-------------|-------------------|---------------------------|
| R1     | Serial0/0   | 172.16.1.1  | 255.255.255.192 /26 | DCE, clock rate 768000    |
| R3     | Serial0/1   | 172.16.1.2  | 255.255.255.192 /26 |                           |
| R3     | Loopback10  | 10.10.10.3  | 255.255.255.128 /25 |                           |
| R3     | Loopback20  | 10.20.20.3  | 255.255.255.240 /28 |                           |
| R3     | Loopback30  | 10.30.30.3  | 255.255.255.248 /29 |                           |

> **Note:** The provided transcript shows R1 Serial0/0 with `172.16.1.2` and R3 Serial0/1 also with `172.16.1.2`. This is a duplicate IP typo. The working design uses **R1 = 172.16.1.1** and **R3 = 172.16.1.2**. Verification pings from R3 to 172.16.1.1 confirm this.

## Task 1: Configure Hostnames
Configure hostnames **R1** and **R3** as shown in the topology.

## Task 2: Basic Configuration and Default Routes

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
R3 Configuration
cisco
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
Note: The transcript’s R3 default route references serial0/0, but the configured interface is serial0/1.

Task 3: Verify Connectivity (Before ACL)
cisco
R3#ping 172.16.1.1
R3#ping 172.16.1.1 source loopback10
R3#ping 172.16.1.1 source loopback20
R3#ping 172.16.1.1 source loopback30
All pings should succeed with !!!!!.

Task 4: Configure and Apply Standard Named ACL
ACL Configuration on R1
cisco
R1#conf t
ip access-list standard LOOPBACK-10-30-ACL
 remark 'Deny Traffic From R3 Loopback10'
 deny 10.10.10.0 0.0.0.127
 remark 'Permit Traffic From R3 Loopback20'
 permit 10.20.20.0 0.0.0.15
 remark 'Deny Traffic From R3 Loopback30'
 deny 10.30.30.0 0.0.0.7
 remark 'Permit Traffic From Serial0/0 Subnet'
 permit 172.16.1.0 0.0.0.63
 exit
int s0/0
 ip access-group LOOPBACK-10-30-ACL in
 end
Verification
cisco
R1#show ip access-lists LOOPBACK-10-30-ACL
R1#show running-config interface serial 0/0
R1#show ip interface serial 0/0
Expected ping results after ACL:

Source	Destination	Result	Reason
Default (172.16.1.2)	172.16.1.1	!!!!!	Serial subnet explicitly permitted
Loopback10	172.16.1.1	U.U.U	Explicitly denied
Loopback20	172.16.1.1	!!!!!	Explicitly permitted
Loopback30	172.16.1.1	U.U.U	Explicitly denied
Wildcard Mask Calculation
Wildcard mask = 255.255.255.255 − subnet mask.

Subnet	Subnet Mask	Wildcard Mask
10.10.10.0/25	255.255.255.128	0.0.0.127
10.20.20.0/28	255.255.255.240	0.0.0.15
10.30.30.0/29	255.255.255.248	0.0.0.7
172.16.1.0/26	255.255.255.192	0.0.0.63
Key Concepts
Standard ACLs filter based on source IP address only.

Standard ACLs should be applied as close to the destination as possible.

Named ACLs use ip access-list standard <name> and allow remark statements for documentation.

ACLs are processed top-down; the first match wins.

There is an implicit deny at the end of every ACL. If traffic is not explicitly permitted, it is denied.

U.U.U typically indicates traffic was administratively prohibited by an ACL.

Apply ACLs inbound or outbound on an interface with ip access-group <name> in|out.

Use show ip access-lists and show ip interface <intf> to verify.
