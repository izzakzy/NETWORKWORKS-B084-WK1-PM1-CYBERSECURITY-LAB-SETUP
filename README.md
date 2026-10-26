# NetworkWorks B084 WK1 PM1 — Cybersecurity Lab Setup

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

## Overview

This project documents the setup of a cybersecurity lab using Oracle VirtualBox on an Arch Linux host, with Kali Linux as the guest operating system.

The lab uses a VirtualBox NAT Network named `nat` to provide network connectivity for the Kali Linux virtual machine while keeping the lab environment separate from a direct bridged connection to the physical network.

## Objectives

- Install and configure VirtualBox on Arch Linux.
- Configure a VirtualBox NAT Network named `nat`.
- Set up a Kali Linux virtual machine.
- Configure the virtual machine to use the `nat` network.
- Verify network connectivity from Kali Linux.
- Document the setup process and troubleshooting steps.

## Lab Environment

| Component | Configuration |
|---|---|
| Host device | Lenovo ThinkPad T14s |
| Host operating system | Arch Linux |
| Processor | AMD Ryzen 5 PRO |
| Memory | 8 GB RAM |
| Virtualization software | Oracle VirtualBox |
| Guest operating system | Kali Linux |
| Network mode | NAT Network |
| NAT Network name | `nat` |
| Example network subnet | `10.0.0.0/24` |

*Note: The subnet is an example. Use a subnet that does not conflict with your local network, VPN, or other virtual networks.*

## 1. Install VirtualBox on Arch Linux

Update the system packages:

```bash
sudo pacman -Syu
```

Install VirtualBox and its DKMS host modules:

```bash
sudo pacman -S virtualbox virtualbox-host-dkms
```

Install the kernel headers matching your installed kernel. For example:

```bash
sudo pacman -S linux-headers
```

For other kernels, install the corresponding headers, such as `linux-lts-headers` or `linux-zen-headers`.

Load the VirtualBox kernel module:

```bash
sudo modprobe vboxdrv
```

Verify the installation:

```bash
VBoxManage --version
```

If the module fails to load, check that the installed kernel headers match the running kernel and consult the ArchWiki VirtualBox guide.

## 2. Configure the VirtualBox NAT Network

The NAT Network will be named `nat`.

### Check existing NAT Networks

```bash
VBoxManage list natnets
```

### Create the NAT Network

If a network named `nat` does not already exist, create it with an example subnet and DHCP enabled:

```bash
VBoxManage natnetwork add \
  --netname "nat" \
  --network "10.0.0.0/24" \
  --enable \
  --dhcp on
```

### Start the NAT Network

```bash
VBoxManage natnetwork start --netname "nat"
```

### Verify the configuration

```bash
VBoxManage list natnets
```

Confirm that the network named `nat` is listed and enabled.

If `nat` already exists, do not create a duplicate. Enable and configure the existing network instead:

```bash
VBoxManage natnetwork modify \
  --netname "nat" \
  --enable \
  --dhcp on
```

Then start it if it is not already running:

```bash
VBoxManage natnetwork start --netname "nat"
```

## 3. Configure the Kali Linux Virtual Machine

In the VirtualBox graphical interface:

1. Select the Kali Linux virtual machine.
2. Open **Settings**.
3. Select **Network**.
4. Open **Adapter 1**.
5. Enable the network adapter.
6. Set **Attached to** to **NAT Network**.
7. Select `nat` as the network name.
8. Save the changes.

Start the Kali Linux virtual machine.

## 4. Verify Network Connectivity in Kali Linux

Open a terminal in Kali Linux.

### Check the network interfaces

```bash
ip addr
```

Confirm that the active interface has an IP address assigned by DHCP.

### Check the routing table

```bash
ip route
```

Confirm that an appropriate default route is present.

### Test external IP connectivity

```bash
ping -c 4 1.1.1.1
```

### Test DNS resolution

```bash
ping -c 4 google.com
```

Successful responses indicate that IP connectivity and DNS resolution are working. If one test fails, investigate the relevant network configuration before assuming the entire network is unavailable.

## 5. Screenshots

The following screenshots document the lab setup. Keep these image files in the repository alongside this README.

### Screenshot 1 — Lab Setup Title

![Lab setup title](1-screenshot-title-image.png)

### Screenshot 2 — VirtualBox Network Settings

![VirtualBox network settings](2-screenshot-network-settings-1.png)

### Screenshot 3 — Kali Linux Virtual Machine

![Kali Linux virtual machine](3-screenshot-kali-linux.png)

### Screenshot 4 — Kali Linux Network Settings

![Kali Linux network settings](4-screenshot-kali-network-settings.png)

## 6. Troubleshooting

### VirtualBox kernel module fails to load

Check the running kernel:

```bash
uname -r
```

Ensure the matching kernel headers are installed, then review the ArchWiki instructions for your kernel and VirtualBox package configuration.

### NAT Network `nat` is not listed

```bash
VBoxManage list natnets
```

If it does not exist, create it using the commands in Section 2.

### Kali Linux does not receive an IP address

- Confirm that Adapter 1 is enabled.
- Confirm that **Attached to** is set to **NAT Network**.
- Confirm that the selected network is `nat`.
- Check that DHCP is enabled for the NAT Network.
- Restart the NAT Network and Kali VM if necessary.

### IP connectivity works, but domain names do not resolve

Test an IP address and a domain name separately. If IP connectivity works but DNS resolution fails, investigate the guest's DNS configuration and VirtualBox network settings.

### Network subnet conflicts

The example subnet `10.0.0.0/24` may not be suitable for every environment. Choose a non-conflicting subnet if it overlaps with your local network, VPN, or another virtual network.

## 7. Learning Outcomes

This lab provides practical experience with:

- Installing virtualization software on Arch Linux.
- Understanding VirtualBox NAT Network configuration.
- Connecting a Kali Linux guest to a virtual network.
- Checking IP addressing, routes, and DNS resolution.
- Troubleshooting basic virtual machine networking.
- Documenting a repeatable cybersecurity lab environment.

## Security Note

Use Kali Linux and other security-testing tools only on systems and networks you own or have explicit permission to test. Keep experimental activities within an authorized lab environment.

## References

- [ArchWiki — VirtualBox](https://wiki.archlinux.org/title/VirtualBox)
- [Oracle VirtualBox User Manual — Networking](https://docs.oracle.com/en/virtualization/virtualbox/7.2/user/networkingdetails.html)
- [Kali Linux Documentation](https://www.kali.org/docs/)

## Author

**Izak** — [@izzakzy](https://github.com/izzakzy)

Cybersecurity lab setup using Arch Linux, VirtualBox, and Kali Linux.
