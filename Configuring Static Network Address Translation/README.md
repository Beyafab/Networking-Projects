# CCNA Lab: Static Network Address Translation (NAT) Configuration & Verification

A hands-on Cisco CCNA preparation lab covering the configuration, operational logic, and verification of **Static Network Address Translation (Static NAT)** using Cisco IOS commands.

---

## 📌 Overview

Static NAT provides a **1-to-1 permanent mapping** between a private IP address (RFC 1918) and a public IP address. It allows external, untrusted network hosts to initiate connections to internal private hosts without exposing internal IP routing topologies or internal private addresses.

- **Certification Level:** Cisco CCNA
- **Difficulty Rating:** 8 / 10
- **Target Completion Time:** $\le 10\text{ minutes}$

---

## 🗺️ Topology & Addressing Scheme

```text
[ R1 ] (Outside Host)
  │
  │ Serial 0/0 (DCE: 192.168.254.1/24)
  ▼
[ R3 ] (NAT Border Router)
  │ Serial 0/0 (Outside NAT: 192.168.254.2/24)
  │ FastEthernet 0/0 (Inside NAT: 10.2.2.2/27)
  ▼
[ Sw1 ] (Inside Host / Switch SVI)
    FastEthernet 0/2 (VLAN 50: 10.2.2.4/27)
```

### IP Addressing Table

| Device | Interface | IP Address | Subnet Mask | NAT Role / Function |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | Serial 0/0 | `192.168.254.1` | `255.255.255.0` | Outside Host |
| **R3** | Serial 0/0 | `192.168.254.2` | `255.255.255.0` | `ip nat outside` |
| **R3** | FastEthernet 0/0 | `10.2.2.2` | `255.255.255.224` | `ip nat inside` (Default GW) |
| **Sw1** | Interface VLAN 50 | `10.2.2.4` | `255.255.255.224` | Inside Local Host |
| **NAT Static** | *Virtual Mapping* | `192.168.254.4` | N/A | Inside Global Address |

---

## 📋 Task Summary

1. **Hostnames:** Configure device hostnames on `R1`, `R3`, and `Sw1`.
2. **Serial & Clocking:** Configure `R1` S0/0 as DCE with `clock rate 256000` and assign IP address `192.168.254.1/24`. Configure `R3` S0/0 with `192.168.254.2/24`.
3. **VLAN & Security:** Create `VLAN 50` (`NAT_VLAN`) on `Sw1`, assign `Fa0/2` to `VLAN 50`, and enable Telnet access with password `CISCO`.
4. **SVI & Gateways:** Configure `Sw1` SVI `VLAN 50` (`10.2.2.4/27`) and set `ip default-gateway 10.2.2.2`. Configure `R3` `Fa0/0` with `10.2.2.2/27`.
5. **Baseline Verification:** Confirm baseline reachability. Verify that `Sw1` cannot ping `R1` prior to NAT setup.
6. **Static NAT Configuration:** Designate NAT inside/outside interfaces on `R3` and create static translation between `10.2.2.4` and `192.168.254.4`.
7. **End-to-End Verification:** Verify ping from `Sw1` to `R1`, and test Telnet connection from `R1` to `192.168.254.4`.

---

## ⚙️ Configuration Commands

### Task 3: Switch Access & VTY Setup
```cisco
Sw1# configure terminal
Sw1(config)# vlan 50
Sw1(config-vlan)# name NAT_VLAN
Sw1(config-vlan)# exit
Sw1(config)# interface FastEthernet 0/2
Sw1(config-if)# switchport mode access
Sw1(config-if)# switchport access vlan 50
Sw1(config-if)# no shutdown
Sw1(config-if)# exit
Sw1(config)# line vty 0 15
Sw1(config-line)# password CISCO
Sw1(config-line)# login
Sw1(config-line)# end
```

### Task 4: SVI & Router Gateway Setup
```cisco
! Switch SVI and Gateway
Sw1# configure terminal
Sw1(config)# interface vlan 1
Sw1(config-if)# shutdown
Sw1(config-if)# exit
Sw1(config)# interface vlan 50
Sw1(config-if)# no shutdown
Sw1(config-if)# ip address 10.2.2.4 255.255.255.224
Sw1(config-if)# exit
Sw1(config)# ip default-gateway 10.2.2.2
Sw1(config)# end

! Router Interface Configuration
R3# configure terminal
R3(config)# interface FastEthernet 0/0
R3(config-if)# no shutdown
R3(config-if)# ip address 10.2.2.2 255.255.255.224
R3(config-if)# end
```

### Task 6: Static NAT Configuration (R3)
```cisco
R3# configure terminal
R3(config)# interface FastEthernet 0/0
R3(config-if)# ip nat inside
R3(config-if)# exit
R3(config)# interface Serial 0/0
R3(config-if)# ip nat outside
R3(config-if)# exit
R3(config)# ip nat inside source static 10.2.2.4 192.168.254.4
R3(config)# end
```

---

## 🔍 Verification & Diagnostics

### 1. Check NAT Translation Table
```cisco
R3# show ip nat translations
Pro Inside global    Inside local    Outside local Outside global
--- 192.168.254.4    10.2.2.4        ---           ---
```

### 2. End-to-End ICMP Connectivity (Sw1 to R1)
```cisco
Sw1# ping 192.168.254.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.254.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/6/8 ms
```

### 3. Remote Management Test (R1 to Sw1 via Public Mapping)
```cisco
R1# telnet 192.168.254.4
Trying 192.168.254.4 ... Open

User Access Verification

Password: 
Sw1#
```

### 4. Inspect NAT Statistics & Performance
```cisco
R3# show ip nat statistics
Total active translations: 1 (1 static, 0 dynamic; 0 extended)
Outside interfaces:
  Serial0/0
Inside interfaces:
  FastEthernet0/0
Hits: 53  Misses: 0
Expired translations: 0
```

> **Note:** Because Static NAT uses permanent 1-to-1 mappings, translation entries do not time out or expire automatically from the NAT table.