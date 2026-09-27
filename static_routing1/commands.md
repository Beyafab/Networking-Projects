# Static Routing - Configuration Commands

## 1. Initial Router Configuration (Passwords)

**On R1, R2, and R3:**
```cisco
enable
configure terminal
hostname R1/R2/R3


** R1 Router **
R1(config)# ip route 23.23.23.0 255.255.255.0 12.12.12.2
R1(config)# ip route 192.168.2.0 255.255.255.0 12.12.12.2
R1(config)# ip route 192.168.3.0 255.255.255.0 12.12.12.2

** R2 Router **
R2(config)# ip route 192.168.1.0 255.255.255.0 12.12.12.1
R2(config)# ip route 192.168.3.0 255.255.255.0 23.23.23.3

** R3 Router **

R3(config)# ip route 12.12.12.0 255.255.255.0 23.23.23.2
R3(config)# ip route 192.168.1.0 255.255.255.0 23.23.23.2
R3(config)# ip route 192.168.2.0 255.255.255.0 23.23.23.2

** Verification Commands **

R1# show ip route static | begin Gateway
R2# show ip route static | begin Gateway
R3# show ip route static | begin Gateway
