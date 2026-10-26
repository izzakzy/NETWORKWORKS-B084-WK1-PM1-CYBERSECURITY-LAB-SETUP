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

This project documents a cybersecurity testing lab built with Oracle VirtualBox on an **Arch Linux** host, with Kali Linux as the attacking machine. The lab uses a VirtualBox **NAT Network** named `NAT` in the subnet `10.0.0.0/24`. Kali has the static IP `10.0.0.2/24` and full internet access, and the NAT Network keeps the lab separate from the physical network.

The guide below is written so another Arch Linux user can repeat the setup from scratch.

## Objectives

- Install VirtualBox and 7-Zip on Arch Linux.
- Create a NAT Network in `10.0.0.0/24`.
- Import Kali Linux as the attacking machine.
- Enable clipboard sharing and drag and drop.
- Share the host `Downloads` folder with the VM.
- Set the Kali IP address to `10.0.0.2/24`.
- Verify gateway, internet and DNS connectivity.
- Take a clean snapshot of the VM.

## Lab Environment

| Component | Configuration |
|---|---|
| Host | Lenovo ThinkPad T14s, AMD Ryzen 5 PRO, 8 GB RAM |
| Host OS | Arch Linux (standard `linux` kernel) |
| Hypervisor | Oracle VirtualBox 7.2.22 with DKMS host modules |
| Guest OS | Kali Linux 2026.2 |
| VM name | `kali-linux-2026.2-virtualbox-amd64` |
| VM resources | 3072 MB RAM, 2 CPUs |
| Network mode | NAT Network |
| NAT Network name | `NAT` |
| Network address | `10.0.0.0/24` |
| DHCP on the NAT Network | Disabled (static IPs are used) |
| Gateway | `10.0.0.1` |
| Kali IP address | `10.0.0.2/24` |
| DNS servers | `1.1.1.1`, `8.8.8.8` |
| Clipboard / drag and drop | Bidirectional |
| Shared folder | Host `~/Downloads` → Kali `/media/sf_downloads` |

## Lab Architecture

```
Arch Linux host (VirtualBox 7.2.22)
   │
   └── NAT Network "NAT"  (10.0.0.0/24, gateway 10.0.0.1, DHCP off)
          │
          └── Kali Linux  10.0.0.2/24 ──► internet (via NAT)

Shared folder:  host ~/Downloads  ◄──►  Kali /media/sf_downloads
```

Future target machines (Windows and Android VMs) can join the same NAT Network with their own static addresses.

---

## Setup Procedure

Run all host commands as your normal user. Use `sudo` only where shown, and never run VirtualBox itself as root.

### Step 1. Check the host network

The lab subnet must not overlap with the host's own network.

```bash
ip -4 -br a
```

If your Wi-Fi or Ethernet address starts with `10.0.0.`, the lab network would conflict with it. Addresses such as `192.168.x.x` are fine. The libvirt/QEMU bridge (`virbr0`, normally `192.168.122.0/24`) does not conflict.

### Step 2. Install VirtualBox and 7-Zip

Update the system and install the packages. VirtualBox needs kernel modules that match the running kernel, so the DKMS package is used together with the kernel headers.

```bash
uname -r                                   # note which kernel you run
sudo pacman -Syu
sudo pacman -S virtualbox virtualbox-host-dkms linux-headers p7zip xorg-xwayland
sudo usermod -aG vboxusers $USER
sudo reboot
```

`linux-headers` is for the standard Arch kernel. For other kernels, install the matching headers instead, such as `linux-lts-headers` or `linux-zen-headers`. `xorg-xwayland` is needed to run VirtualBox on a Wayland desktop.

After the reboot, verify the installation:

```bash
uname -r && pacman -Q linux-headers     # versions should match
dkms status                             # vboxhost should show as installed
sudo modprobe vboxdrv
VBoxManage --version
```

**Existing QEMU/KVM setup.** VirtualBox can be installed next to QEMU/KVM, and nothing needs to be removed. Do not run a QEMU/KVM VM and a VirtualBox VM at the same time. If VirtualBox refuses to start a VM because of a VT-x/AMD-V conflict with all QEMU VMs shut down, unload the KVM module temporarily:

```bash
sudo modprobe -r kvm_amd      # kvm_intel on Intel CPUs
```

### Step 3. Create the NAT Network

VirtualBox has no NAT Network by default, so create one named `NAT` with DHCP disabled:

```bash
VBoxManage natnetwork add --netname NAT --network "10.0.0.0/24" --enable --dhcp off
VBoxManage list natnetworks
```

Expected output:

```
Name:         NAT
Network:      10.0.0.0/24
Gateway:      10.0.0.1
DHCP Server:  No
```

In the GUI, the same network is under **File → Tools → Network Manager → NAT Networks**. A NAT Network is used instead of plain NAT so that several VMs on the same network can reach each other as well as the internet.

### Step 4. Download and import Kali Linux

1. Download the **VirtualBox 64-bit** image from the Virtual Machines tab at <https://kali.org/get-kali>. About 20 GB of free space is needed in `/home`, because VMs are stored in the user's home folder.
2. Extract it from the folder where it was saved:

   ```bash
   cd ~/Downloads
   7z x kali-linux-2026.2-virtualbox-amd64.7z
   ```

3. Start `VirtualBox`, choose **Machine → Add**, and select the extracted `.vbox` file.

The default Kali login is `kali` / `kali`.

### Step 5. Configure the virtual machine

With the VM powered off, open **Settings** and set the following. If **NAT Network** is missing from the *Attached to* list, switch VirtualBox to **Expert Mode** and make sure the network from Step 3 exists.

| Section | Setting |
|---|---|
| Network → Adapter 1 | Attached to **NAT Network**, Name `NAT` |
| General → Advanced | Shared Clipboard **Bidirectional**, Drag'n'Drop **Bidirectional** |
| System | Base Memory 3072 MB, Processors 2 |
| Shared Folders | Folder Path `/home/<user>/Downloads`, Folder Name `downloads`, **Auto-mount** and **Make Permanent** enabled |

The same settings from the terminal:

```bash
VM="kali-linux-2026.2-virtualbox-amd64"
VBoxManage modifyvm "$VM" --nic1 natnetwork --nat-network1 NAT
VBoxManage modifyvm "$VM" --clipboard-mode bidirectional --draganddrop bidirectional
VBoxManage modifyvm "$VM" --memory 3072 --cpus 2
VBoxManage sharedfolder add "$VM" --name downloads --hostpath "$HOME/Downloads" --automount
```

Check the saved configuration:

```bash
VBoxManage showvminfo "$VM" --machinereadable | grep -iE "^nic1|^nat-network1|sharedfolder"
```

Only the host `Downloads` folder is shared, not the whole home folder.

### Step 6. Set the static IP address in Kali

Start the VM, log in, and run in a Kali terminal:

```bash
nmcli con show                       # note the wired connection name

sudo nmcli con mod "Wired connection 1" \
  ipv4.method manual \
  ipv4.addresses 10.0.0.2/24 \
  ipv4.gateway 10.0.0.1 \
  ipv4.dns "1.1.1.1 8.8.8.8"

sudo nmcli con up "Wired connection 1"
sudo usermod -aG vboxsf kali         # allows access to the shared folder
```

The connection name may differ between systems, so use the name shown by `nmcli con show`. Log out and back in after the `usermod` command.

### Step 7. Verify the lab

Run these in Kali:

| Test | Command | Expected result |
|---|---|---|
| IP address | `ip a` | `eth0` has `10.0.0.2/24` |
| Default route | `ip route` | Default route via `10.0.0.1` |
| Gateway | `ping -c 3 10.0.0.1` | Replies received |
| Internet | `ping -c 3 8.8.8.8` | Replies received |
| DNS | `ping -c 3 google.com` | Name resolves, replies received |
| Shared folder | `ls /media/sf_downloads` | Host Downloads content listed |

Observed results:

```
eth0: inet 10.0.0.2/24
default via 10.0.0.1 dev eth0 proto static
PING google.com (142.250.198.174) ... replies received
```

An occasional `(DUP!)` line in ping output is a known quirk of the VirtualBox NAT Network and is harmless. Clipboard sharing can be tested by copying text between the host and Kali.

### Step 8. Take a snapshot

After all tests pass, take a clean snapshot in VirtualBox (**Machine → Take Snapshot**), or from the terminal:

```bash
VBoxManage snapshot "kali-linux-2026.2-virtualbox-amd64" take "clean-lab-setup"
VBoxManage snapshot "kali-linux-2026.2-virtualbox-amd64" list
```

The snapshot is the clean baseline of the lab. If a future exercise breaks the VM, it can be restored to this point.

---

## Problems Encountered and Solutions

| Problem | Solution |
|---|---|
| `error: target not found: virtualbox-host-modules-arch` | The prebuilt module package is no longer available under that name. Use `virtualbox-host-dkms` with `linux-headers` and confirm with `dkms status` |
| **NAT Network** missing from the adapter dropdown | The network had not been created and VirtualBox was in Basic Mode. Create it with `VBoxManage natnetwork add` and switch to Expert Mode |
| Kali gets a `10.0.2.x` address | Plain *NAT* mode was selected. Use **NAT Network** with the name `NAT` |
| `dkms status` is empty or shows an error | The headers do not match the running kernel. Install the matching headers, reboot, and run `sudo dkms autoinstall` |
| VM will not start (VT-x / AMD-V error) | Enable virtualization (SVM / VT-x) in the BIOS, and shut down any QEMU/KVM VMs. If needed, run `sudo modprobe -r kvm_amd` |
| Gateway pings work but `google.com` fails | DNS problem. Re-check `ipv4.dns` and run `sudo nmcli con up` again |
| Shared folder is empty or access is denied | Run `sudo usermod -aG vboxsf kali`, then log out and back in |
| Drag and drop does not work | Known limitation on Wayland hosts (VirtualBox runs through XWayland). The setting is enabled, and the shared folder is used for file transfer |

---

## What I Learned

- **NAT vs NAT Network:** plain NAT isolates each VM, while a NAT Network lets VMs on the same virtual network reach each other and the internet.
- **Static addressing:** configuring IP, gateway and DNS with NetworkManager, and why DHCP is off on this network.
- **Kernel modules on Arch:** VirtualBox needs modules matching the running kernel, which DKMS and the kernel headers provide.
- **VM integration:** shared folders, clipboard sharing and the `vboxsf` group connect the guest to the host.
- **Snapshots and documentation:** a clean snapshot gives a safe recovery point, and documenting problems makes the lab repeatable.

## Security and Ethical Use

This laboratory is intended strictly for education. Use Kali Linux and other security-testing tools only on systems and networks you own or have explicit permission to test. Keep experimental activity inside the isolated lab.

## Tools and Resources

- **7-Zip:** <https://7-zip.org/download.html>
- **VirtualBox:** <https://virtualbox.org/wiki/Downloads>
- **Kali Linux:** <https://kali.org/get-kali>
- [ArchWiki — VirtualBox](https://wiki.archlinux.org/title/VirtualBox)
- [Oracle VirtualBox User Manual — Networking](https://docs.oracle.com/en/virtualization/virtualbox/7.2/user/networkingdetails.html)
- [Kali Linux Documentation](https://www.kali.org/docs/)

## Author

**Izak** — [@izzakzy](https://github.com/izzakzy)

Cybersecurity lab setup using Arch Linux, VirtualBox, and Kali Linux.