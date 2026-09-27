# Static Routing

## 📌 Lab Overview
This lab demonstrates the configuration of IPv4 static routes on Cisco routers to achieve full network reachability. It covers two methods of static routing: **Next-Hop** (specifying the next router's IP) and **Exit-Interface** (specifying the local outgoing interface).

## 🎯 Objectives
1. Configure routers with IP addresses and passwords (Enable: `ccna`, Access: `cisco`).
2. Configure static routes (Next-hop and Exit-interface) to provide network reachability.
3. Verify connectivity by pinging between Cloud-PC, PC-2, and PC-3.

## 🗺️ Topology & IP Addressing

| Device | Interface | IP Address | Subnet Mask | Description |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | FastEthernet0/0 | 192.168.1.1 | 255.255.255.0 | LAN (Cloud-PC) |
| **R1** | Serial2/0 | 12.12.12.1 | 255.255.255.0 | WAN Link to R2 |
| **R2** | Serial2/0 | 12.12.12.2 | 255.255.255.0 | WAN Link to R1 |
| **R2** | FastEthernet0/0 | 23.23.23.2 | 255.255.255.0 | WAN Link to R3 |
| **R2** | FastEthernet1/1 | 192.168.2.1 | 255.255.255.0 | LAN (PC-2) |
| **R3** | FastEthernet0/0 | 23.23.23.3 | 255.255.255.0 | WAN Link to R2 |
| **R3** | FastEthernet1/1 | 192.168.3.1 | 255.255.255.0 | LAN (PC-3) |

> **Note:** The Cloud-PC is a bridged connection on R1's LAN (192.168.1.0/24). PC-2 and PC-3 act as end hosts on their respective LANs.

## 🛠️ Key Concepts Covered
*   **Next-Hop Static Routing:** `ip route [destination] [mask] [next-hop-ip]`
*   **Exit-Interface Static Routing:** `ip route [destination] [mask] [local-interface]`
*   **Verification:** `show ip route`, `ping`, `traceroute`.

## 🚀 How to Use
1. Open Cisco Packet Tracer or GNS3 and build the topology as described above.
2. Apply the configurations found in `commands.md`.
3. Verify connectivity by pinging between the Cloud-PC, PC-2, and PC-3.
