# Windows 11 Endpoint Setup (`BTL-WIN11`)

This document details the deployment, hardware allocation, security configuration, user provisioning, and initial setup of the primary target endpoint (`BTL-WIN11`) used for generating security telemetry.

---

## Hardware & Virtual Machine Configuration

The virtual machine is hosted on Apple Silicon using VMware Fusion with the following specifications:

| Parameter | Configuration |
| :--- | :--- |
| **Guest OS** | Windows 11 Pro ARM64 |
| **Firmware** | UEFI |
| **Secure Boot** | Enabled |
| **TPM Support** | Enabled (Virtual TPM) |
| **vCPU** | 4 Cores |
| **RAM** | 8 GB |
| **Storage** | 64 GB NVMe / Virtual Disk |
| **Network Adapter** | VMware NAT |

---

## Operating System Installation & Provisioning

1. **Virtual Machine Creation:**
   * Created a new VMware Fusion VM using the Windows 11 ARM64 ISO.
   * Configured Virtual TPM and enabled Secure Boot in VM settings to satisfy Windows 11 deployment requirements.

2. **System Updates & Drivers:**
   * Completed initial Windows Setup (OOBE).
   * Installed VMware Tools for ARM64 for full display resolution support and driver integration.
   * Executed Windows Update to install the latest security updates and patches.

---

## User Account Provisioning

To simulate proper privilege segregation and enable multi-account telemetry testing, two dedicated lab accounts were created:

| Username | Account Role | Purpose |
| :--- | :--- | :--- |
| `labadmin` | Local Administrator | System management, tool installation, and operational tasks |
| `testuser` | Standard User | Low-privileged execution and user-context activity generation |

> **Security Note:** Passwords for both accounts follow lab security standards and are managed locally. No plain-text passwords or hashes are committed to this repository.

---

## Hostname Configuration

The hostname was updated to match lab documentation standards (`BTL-WIN11`) using PowerShell:

```powershell
# Execute from an elevated PowerShell prompt
Rename-Computer -NewName "BTL-WIN11" -Restart

```

Network & Connectivity Verification
-----------------------------------

Upon reboot, verified network connectivity and DHCP IP assignment within the VMware NAT subnet (`172.16.106.0/24`):

PowerShell

```
Get-NetIPAddress -AddressFamily IPv4 | Select-Interface -Property IPAddress, InterfaceAlias

```

-   **Hostname:** `BTL-WIN11`

-   **Network Interface:** VMware Virtual Ethernet Adapter (NAT)
