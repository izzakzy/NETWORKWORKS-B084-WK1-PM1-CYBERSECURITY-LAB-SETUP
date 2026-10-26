# 🔐 Cybersecurity Lab Setup

<p align="center">
  <img src="https://img.shields.io/badge/Skill-Cybersecurity-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/Ver-VirtualBox-0070C0?style=flat-square&labelColor=000000" />
  <img src="https://img.shields.io/badge/Kali%20Linux-Lab-E87500?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/Skill-Linux-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/Host-Arch%20Linux-1793D1?style=flat-square&labelColor=000000&logo=archlinux&logoColor=white" />
  <img src="https://img.shields.io/badge/Penetration%20Testing-C00000?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/Skill-Virtualization-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/GitHub-404040?style=flat-square&labelColor=0070C0&logo=github&logoColor=white" />
  <img src="https://img.shields.io/badge/Kali%20Linux-404040?style=flat-square&labelColor=C00000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/NetworkWalks-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/Ethical%20Hacking-E87500?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/Author-Izak-C00000?style=flat-square" />
</p>

## 📌 Overview

This project documents the setup of a cybersecurity lab using Oracle VirtualBox and Kali Linux. The lab provides a virtual environment for practicing Linux commands, understanding network configuration, and learning cybersecurity fundamentals.

## 🖥️ Lab Environment

![Cybersecurity Lab Setup](1-screenshot-title-image.png)

My lab runs on a Lenovo ThinkPad T14s with Arch Linux as the host operating system.

| Component | Specification |
|---|---|
| Host OS | Arch Linux |
| Virtualization | Oracle VirtualBox |
| Guest OS | Kali Linux |

## ⚙️ VirtualBox Network Configuration

![VirtualBox Network Settings](2-screenshot-network-settings-1.png)

VirtualBox provides the virtual networking environment for the Kali Linux machine. The network adapter should be configured according to the lab requirements so the virtual machine can communicate through the selected virtual network.

## 🐉 Kali Linux Virtual Machine

![Kali Linux Virtual Machine](3-screenshot-kali-linux.png)

Kali Linux is the guest operating system used for the lab. It provides a Linux environment for practicing system administration, networking, and authorized cybersecurity testing.

## 🌐 Network Verification

![Kali Linux Network Settings](4-screenshot-kali-network-settings.png)

The following commands help verify network interfaces, routing, and connectivity.

**Check IP addresses:**

```bash
ip addr
```

**View the routing table:**

```bash
ip route
```

**Test network connectivity:**

```bash
ping -c 4 1.1.1.1
```

**Test DNS resolution:**

```bash
ping -c 4 google.com
```

These checks help determine whether the virtual machine has an IP address, a route to the network, and working connectivity.

## 🎯 Learning Objectives

- Understand virtualization using VirtualBox.
- Set up and run a Kali Linux virtual machine.
- Learn basic virtual network configuration.
- Inspect IP addresses and routing information.
- Test network connectivity and DNS resolution.
- Build a foundation in cybersecurity lab practices.

## 🔐 Security Note

This lab is intended for educational purposes and authorized security testing only. Always obtain permission before testing systems or networks.

## 👤 Author

**izak**

GitHub: [@izzakzy](https://github.com/izzakzy)

---

*Cybersecurity Lab Setup — Virtualization, Linux Networking, and Security Fundamentals.*
