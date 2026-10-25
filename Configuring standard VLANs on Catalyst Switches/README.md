# Lab 1: Configuring Standard VLANs on Catalyst Switches

## 🎯 Lab Objective
The objective of this lab exercise is to learn and understand how to configure standard VLANs (1–1001) on Cisco Catalyst IOS switches, as well as how to validate configurations using verification commands.

## 📌 Metadata
* **Certification Target:** CCENT / CCNA
* **Lab Difficulty:** 4 / 10
* **Time Limit:** 10 minutes
* **Topics Covered:** Standard VLAN Creation, Port Assignment, Interface Configuration

---

## 💡 Lab Purpose
VLAN configuration is a fundamental skill. VLANs allow you to segment your network into multiple, smaller broadcast domains. As a Cisco engineer, as well as in the Cisco CCNA exam, you will be expected to know how to configure VLANs on Cisco switches.

---

## 📝 Tasks

* **Task 1:** In preparation for VLAN configuration, configure a hostname on `Sw1` as well as the following standard VLANs:
  * VLAN 10: `SALES`
  * VLAN 20: `MANAGERS`
  * VLAN 30: `ENGINEERS`
  * VLAN 40: `SUPPORT`
* **Task 2:** Configure ports `FastEthernet0/5` – `FastEthernet0/8` as access ports and assign them to the specified VLANs.
* **Task 3:** Verify your VLAN configuration using relevant `show` commands in Cisco IOS.

---

## 🛠️ Configuration & Verification

### Task 1: Hostname & VLAN Creation
```text
Switch# configure terminal
Switch(config)# hostname Sw1
Sw1(config)# vlan 10
Sw1(config-vlan)# name SALES
Sw1(config-vlan)# exit
Sw1(config)# vlan 20
Sw1(config-vlan)# name MANAGERS
Sw1(config-vlan)# exit
Sw1(config)# vlan 30
Sw1(config-vlan)# name ENGINEERS
Sw1(config-vlan)# exit
Sw1(config)# vlan 40
Sw1(config-vlan)# name SUPPORT
Sw1(config-vlan)# exit
```

> **Note:** By default, Cisco switches operate in **VTP Server Mode**, which allows standard-range VLANs (1–1005) to be configured without modifying VTP settings. Use `show vtp status` to check the current operating mode.

### Task 2: Access Port Assignment
```text
Sw1(config)# interface FastEthernet0/5
Sw1(config-if)# switchport mode access
Sw1(config-if)# switchport access vlan 10
Sw1(config-if)# exit

Sw1(config)# interface FastEthernet0/6
Sw1(config-if)# switchport mode access
Sw1(config-if)# switchport access vlan 20
Sw1(config-if)# exit

Sw1(config)# interface FastEthernet0/7
Sw1(config-if)# switchport mode access
Sw1(config-if)# switchport access vlan 30
Sw1(config-if)# exit

Sw1(config)# interface FastEthernet0/8
Sw1(config-if)# switchport mode access
Sw1(config-if)# switchport access vlan 40
Sw1(config-if)# exit
```

### Task 3: Verification
Verify that the ports are properly assigned to their respective VLANs:
```text
Sw1# show vlan brief
```

---

## 🔍 Verification Commands Summary
| Command | Purpose |
| :--- | :--- |
| `show vlan brief` | Displays concise list of active VLANs and assigned ports |
| `show vtp status` | Checks switch VTP operating mode and domain settings |
| `show interface FastEthernet0/5 switchport` | Displays operational switchport mode and access VLAN details |