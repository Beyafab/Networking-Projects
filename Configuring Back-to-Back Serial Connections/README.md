# 🔌 Configuring Back-to-Back Serial Connections (Lab 20)

A fundamental Cisco IOS lab demonstrating how to establish a direct point-to-point Serial link between two routers without an external CSU/DSU by configuring DCE clocking.

## 📋 Lab Overview

* **Certification Level:** CCENT / CCNA
* **Difficulty Rating:** 3/10
* **Target Completion Time:** 10 minutes
* **Objective:** Bring up a back-to-back Serial connection between two Cisco routers (`R1` and `R2`), identify the DCE/DTE roles, configure a `256 Kbps` clock rate, and verify Layer 3 IP connectivity.

### Why This Matters
In a production WAN environment, a router's Serial interface (DTE) receives its synchronization clock signal from a service provider's CSU/DSU (DCE). When connecting two routers directly in a home lab or testing environment, one router's interface acts as the **DCE** and must be manually configured to provide clocking to the **DTE** side so the link protocol can transition to `up/up`.

---

## 🏗️ Topology & Addressing Plan

### Connection Diagram
```text
[ R1 ] (DTE)  <-------- Serial0/0 (172.30.100.0/30) -------->  (DCE) [ R2 ]
```

### IP Addressing Table

| Device | Interface | Role | Clock Rate | IPv4 Address | Subnet Mask |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **R1** | `Serial0/0` | DTE | *Received from R2* | `172.30.100.1` | `255.255.255.252` (`/30`) |
| **R2** | `Serial0/0` | DCE | `256000` bps (`256 Kbps`) | `172.30.100.2` | `255.255.255.252` (`/30`) |

---

## 🚀 Step-by-Step Configuration

### 1. Configure Router Hostnames
Assign descriptive hostnames to both routers:
```ios
! On R1
Router> enable
Router# configure terminal
Router(config)# hostname R1

! On R2
Router> enable
Router# configure terminal
Router(config)# hostname R2
```

### 2. Enable Serial Interfaces & Identify DCE/DTE
Bring up `Serial0/0` on both devices using `no shutdown`. Note that the line protocol will remain `down` until clocking is configured.
```ios
! On R1 & R2
interface Serial0/0
 no shutdown
```

Verify which router has the DCE end of the serial cable attached:
```ios
R2# show controllers serial 0/0
! Expected Output: "DCE V.35, no clock"
```

### 3. Configure Clock Rate on the DCE Interface (R2)
Set the clock rate to `256000` bps (`256 Kbps`) on `R2`'s `Serial0/0` interface:
```ios
R2(config)# interface Serial0/0
R2(config-if)# clock rate 256000
```

### 4. Configure IPv4 Addressing
Assign the `/30` point-to-point IP addresses to each router's Serial interface:
```ios
! On R1
R1(config)# interface Serial0/0
R1(config-if)# ip address 172.30.100.1 255.255.255.252

! On R2
R2(config)# interface Serial0/0
R2(config-if)# ip address 172.30.100.2 255.255.255.252
```

---

## ✅ Verification & Testing

1. **Verify Clocking Detection:**
   * **On R2 (DCE):** Run `show controllers serial 0/0` to confirm `DCE V.35, clock rate 256000`.
   * **On R1 (DTE):** Run `show controllers serial 0/0` to confirm `DTE V.35 TX and RX clocks detected`.

2. **Verify Interface Status (`up/up`):**
   ```ios
   R1# show ip interface brief
   Interface     IP-Address      OK? Method Status   Protocol
   Serial0/0     172.30.100.1    YES manual up       up
   ```

3. **Test End-to-End Connectivity:**
   ```ios
   R1# ping 172.30.100.2
   ! Expected Output: Success rate is 100 percent (5/5)
   ```