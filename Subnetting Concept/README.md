# IPv4 Subnetting Calculator & Practical Guide 🌐

A comprehensive guide and reference implementation for **IPv4 Subnetting**, host address determination, and network address calculations based on practical Cisco Networking Academy lab scenarios.

---

## 📌 Overview

Understanding IPv4 subnetting and host address boundaries is fundamental to computer networking, network security, and infrastructure design. An IP address alone does not reveal network boundaries—it requires a **subnet mask** to divide the address into its network and host components.

This repository covers key formulas, step-by-step subnet calculation techniques, and worked lab exercises covering various subnetting masks (`/24`, `/27`, `/30`, and more).

---

## 🧮 Subnetting Core Formulas

| Calculation | Formula | Description |
| :--- | :--- | :--- |
| **Subnet Bits ($s$)** | $s = \text{New Mask Bits} - \text{Original Mask Bits}$ | Number of bits borrowed from host portion |
| **Subnets Created** | $2^s$ | Total subnets generated from borrowed bits |
| **Host Bits ($h$)** | $h = 32 - \text{Prefix Length}$ | Bits reserved for host addresses in the subnet |
| **Hosts per Subnet** | $2^h - 2$ | Total usable IP addresses (excluding Network & Broadcast IDs) |
| **Block Size (Magic Number)** | $256 - \text{Subnet Octet Value}$ | Step interval between adjacent subnet network addresses |

---

## 📊 Summary of Worked Lab Problems

Below is a breakdown of the subnetting calculations included in this repository:

| Host IP Address | Original Mask | New Subnet Mask | Subnet Bits ($s$) | Created Subnets | Host Bits ($h$) | Usable Hosts | Network Address | Usable Host Range | Broadcast Address |
| :--- | :--- | :--- | :---: | :---: | :---: | :---: | :--- | :--- | :--- |
| **192.168.200.139** | `255.255.255.0` | `255.255.255.224` (/27) | 3 | 8 | 5 | 30 | `192.168.200.128` | `192.168.200.129` – `.158` | `192.168.200.159` |
| **10.101.99.228** | `255.0.0.0` | `255.255.128.0` (/17) | 9 | 512 | 15 | 32,766 | `10.101.0.0` | `10.101.0.1` – `.127.254` | `10.101.127.255` |
| **172.22.32.12** | `255.255.0.0` | `255.255.224.0` (/19) | 3 | 8 | 13 | 8,190 | `172.22.32.0` | `172.22.32.1` – `.63.254` | `172.22.63.255` |
| **192.168.1.245** | `255.255.255.0` | `255.255.255.252` (/30) | 6 | 64 | 2 | 2 | `192.168.1.244` | `192.168.1.245` – `.246` | `192.168.1.247` |
| **128.107.0.55** | `255.255.0.0` | `255.255.255.0` (/24) | 8 | 256 | 8 | 254 | `128.107.0.0` | `128.107.0.1` – `.0.254` | `128.107.0.255` |
| **192.135.250.180** | `255.255.255.0` | `255.255.255.248` (/29) | 5 | 32 | 3 | 6 | `192.135.250.176` | `192.135.250.177` – `.182` | `192.135.250.183` |

---

## 📝 Key Concepts & Takeaways

### Why is the Subnet Mask Essential?
Merely looking at an IPv4 address provides incomplete information. The subnet mask defines the exact boundary between the network prefix and the individual host identifier. Without it, network devices cannot determine:
1. Whether a destination IP address resides on the **local subnet** or a **remote network**.
2. The total number of valid host IDs available within a network segment.
3. The specific network address (`.0` equivalent) and broadcast address (`.255` equivalent).

---

## 🛠️ Requirements & Resources
* **Operating System:** Any OS with a terminal/command prompt or Python support.
* **Optional Tools:** IPv4 address calculator, Wireshark, Packet Tracer.

---

## 📜 License
This repository is shared for educational purposes. Feel free to use and adapt these materials for CCNA study or networking practice.