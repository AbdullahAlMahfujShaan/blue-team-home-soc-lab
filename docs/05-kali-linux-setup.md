# Kali Linux Workstation Setup (`BTL-KALI`)

This document details the installation, virtual machine hardware allocation, desktop environment configuration, and initial package setup for the Kali Linux security workstation (`BTL-KALI`) running on Apple Silicon via VMware Fusion.

---

## Hardware & Virtual Machine Configuration

The virtual machine was deployed using the ARM64 installer/architecture with the following hardware specifications:

| Parameter | Configuration |
| :--- | :--- |
| **Hostname** | `btl-kali` |
| **Architecture** | ARM64 |
| **Guest OS Type** | Debian 13.x ARM64 |
| **vCPU** | 2 Cores |
| **RAM** | 4 GB |
| **Storage** | ~50--60 GB Virtual Disk |
| **Network Adapter** | VMware NAT |
| **Desktop Environment** | Xfce |

---

## Operating System Updates & Package Maintenance

After completing the base OS installation, run the following commands to update system repositories, upgrade all packages, and remove redundant dependencies:

```bash
# Update package lists, perform full distribution upgrade, and clean unused packages
sudo apt update
sudo apt full-upgrade -y
sudo apt autoremove -y
```

VMware Integration & Desktop Tools
----------------------------------

To enable seamless resolution scaling, dynamic display resize, and clipboard integration within VMware Fusion on Apple Silicon, install the Open VM Tools suite:

Bash

```
# Install open-vm-tools and desktop integration utilities
sudo apt install open-vm-tools open-vm-tools-desktop -y
```

System Shutdown & Snapshot Readiness
------------------------------------

Once packages are fully updated and virtualization tools are operational, perform a clean shutdown before capturing a baseline snapshot in VMware Fusion:

Bash

```
# Clean power-off command
sudo poweroff
```

References & Official Resources
-------------------------------

-   **Official Media Downloads:** [Kali Linux Downloads](https://www.kali.org/get-kali/)

-   **Virtualization Documentation:** [Kali VMware Apple Silicon Guide](https://www.google.com/search?q=https://www.kali.org/docs/virtualization/install-vmware-guest-apple-silicon/)
