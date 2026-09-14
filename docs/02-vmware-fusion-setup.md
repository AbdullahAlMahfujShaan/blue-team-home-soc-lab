# VMware Fusion Lab Setup

This document describes the installation and configuration of **VMware Fusion** as the virtualisation platform for the Blue Team Home SOC Lab.

The environment is hosted on an **Apple Silicon Mac**, which requires ARM64-compatible guest operating systems and introduces several architectural considerations when building a cybersecurity lab.

---

# 1. Overview

VMware Fusion provides the virtualisation layer for the Blue Team Home SOC Lab.

The current environment contains three primary virtual machines:

```text
Apple Silicon Mac
       │
       ▼
VMware Fusion
       │
       ├── BTL-WIN11
       │     └── Windows 11 Pro ARM64
       │
       ├── BTL-WAZUH
       │     └── Ubuntu Server ARM64
       │
       └── BTL-KALI
             └── Kali Linux ARM64
```

The virtual machines communicate through a VMware NAT network.

---

# 2. Host Platform

The lab is hosted on an Apple Silicon Mac.

## Host Specifications

| Component | Configuration |
|---|---|
| Processor | Apple M2 Max |
| Architecture | ARM64 / Apple Silicon |
| Memory | 32 GB |
| Virtualisation | VMware Fusion |
| Guest Firmware | UEFI |

---

# 3. Apple Silicon Considerations

Apple Silicon uses the ARM64 architecture.

This means guest operating systems should also use ARM64-compatible versions when running natively through VMware Fusion.

Examples include:

```text
Windows 11 ARM64
Ubuntu Server ARM64
Kali Linux ARM64
```

Traditional x86-64 virtual machines cannot simply be treated as native ARM64 VMware guests.

This is particularly important when building cybersecurity labs because some security tools and pre-built virtual machines are distributed only for x86-64 systems.

---

# 4. Guest Architecture

The following architectures were selected:

| Virtual Machine | Architecture |
|---|---|
| `BTL-WIN11` | ARM64 |
| `BTL-WAZUH` | ARM64 |
| `BTL-KALI` | ARM64 |

This allows all three virtual machines to run efficiently on the Apple Silicon host.

---

# 5. VMware Fusion Installation

Install VMware Fusion from the official Broadcom / VMware distribution.

After installation:

1. Launch VMware Fusion.
2. Allow any required macOS permissions.
3. Confirm VMware Fusion can create and run virtual machines.
4. Download ARM64 installation media for each guest operating system.

---

# 6. Required Installation Media

The lab requires ARM64-compatible operating system images.

## Windows

Use:

```text
Windows 11 ARM64
```

Official download:

https://www.microsoft.com/software-download/windows11arm64

---

## Ubuntu Server

Use an ARM64 Ubuntu Server image.

Example:

```text
Ubuntu Server 24.04 ARM64
```

Official download:

https://ubuntu.com/download/server/arm

---

## Kali Linux

Use:

```text
Kali Linux ARM64 / Apple Silicon
```

Official download:

https://www.kali.org/get-kali/

Kali also provides documentation specifically for VMware on Apple Silicon:

https://www.kali.org/docs/virtualization/install-vmware-silicon-host/

---

# 7. Virtual Machine Naming Convention

Each VM uses a descriptive name based on its role.

```text
BTL-WIN11
BTL-WAZUH
BTL-KALI
```

This naming convention makes systems easier to identify during:

- SIEM investigations
- Network analysis
- Event correlation
- Screenshots
- Documentation
- Incident reports

---

# 8. Windows VM Configuration

The Windows endpoint was configured as:

| Setting | Value |
|---|---|
| VM Name | `BTL-WIN11` |
| Operating System | Windows 11 Pro |
| Architecture | ARM64 |
| vCPU | 4 |
| RAM | 8 GB |
| Disk | 64 GB |
| Firmware | UEFI |
| Secure Boot | Enabled |
| TPM | Enabled |
| Networking | VMware NAT |

---

# 9. Windows Firmware

Windows 11 ARM uses:

```text
UEFI
```

Legacy BIOS is not used for this VM.

---

# 10. Windows Secure Boot

Secure Boot was enabled for `BTL-WIN11`.

```text
Secure Boot: Enabled
```

This more closely resembles a modern Windows workstation configuration.

---

# 11. Windows TPM

A virtual Trusted Platform Module was enabled for the Windows VM.

```text
TPM: Enabled
```

VMware Fusion may require encryption to support a virtual TPM.

Only the files required for TPM support were encrypted rather than unnecessarily encrypting the entire VM.

---

# 12. Kali VM Configuration

The Kali Linux workstation was configured as:

| Setting | Value |
|---|---|
| VM Name | `BTL-KALI` |
| Hostname | `btl-kali` |
| Operating System | Kali Linux |
| Architecture | ARM64 |
| Guest Type | Debian 13.x 64-bit ARM |
| Desktop | Xfce |
| vCPU | 2 |
| RAM | 4 GB |
| Disk | ~50–60 GB |
| Firmware | UEFI |
| Secure Boot | Disabled |
| TPM | Not Required |
| Networking | VMware NAT |

Kali does not require the Windows-specific TPM configuration.

---

# 13. Wazuh Server VM Configuration

The Wazuh server was configured as:

| Setting | Value |
|---|---|
| VM Name | `BTL-WAZUH` |
| Hostname | `btl-wazuh` |
| Operating System | Ubuntu Server |
| Architecture | ARM64 |
| vCPU | 4 |
| RAM | 8 GB |
| Disk | 80 GB |
| Firmware | UEFI |
| Secure Boot | Not Required |
| TPM | Not Required |
| Networking | VMware NAT |

Ubuntu Server was selected instead of Ubuntu Desktop because the Wazuh server does not require a local graphical desktop environment.

This reduces unnecessary resource usage.

---

# 14. Resource Allocation

Current VM resource allocation:

| Virtual Machine | vCPU | RAM | Disk |
|---|---:|---:|---:|
| `BTL-WIN11` | 4 | 8 GB | 64 GB |
| `BTL-WAZUH` | 4 | 8 GB | 80 GB |
| `BTL-KALI` | 2 | 4 GB | ~50–60 GB |
| **Total** | **10 vCPU** | **20 GB** | **~194–204 GB** |

The host contains 32 GB RAM, allowing all three VMs to operate when required.

However, VMs should only be powered on when needed.

Example:

```text
Theory / Documentation
        │
        ▼
All VMs OFF
```

```text
Windows Event Analysis
        │
        ▼
BTL-WIN11 ON
```

```text
SIEM Investigation
        │
        ▼
BTL-WIN11
+
BTL-WAZUH
```

```text
Full Security Exercise
        │
        ▼
BTL-WIN11
+
BTL-WAZUH
+
BTL-KALI
```

---

# 15. VMware Networking

The lab currently uses VMware Fusion's NAT networking mode.

Conceptually:

```text
                    Internet
                       │
                       ▼
                Apple Silicon Mac
                       │
                       ▼
                  VMware NAT
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
     BTL-WIN11     BTL-WAZUH    BTL-KALI
```

During the initial build, VMware assigned addresses from:

```text
172.16.106.0/24
```

Example addresses observed during deployment:

```text
BTL-WIN11  → 172.16.106.131
BTL-KALI   → 172.16.106.132
BTL-WAZUH  → 172.16.106.134
```

These addresses are assigned through DHCP and may change.

Documentation therefore uses placeholders such as:

```text
<WAZUH-SERVER-IP>
<WINDOWS-ENDPOINT-IP>
<KALI-IP>
```

when reproducibility is more important than documenting a temporary address.

---

# 16. Why NAT Was Selected

NAT provides a simple starting network architecture.

It allows the VMs to:

- Communicate with each other
- Reach the Internet for updates
- Download required packages
- Communicate with the Wazuh server
- Remain behind the VMware virtual networking layer

For the initial SOC lab, this provides sufficient connectivity without introducing unnecessary networking complexity.

---

# 17. Future Network Isolation

The current NAT architecture is appropriate for the initial Blue Team lab.

However, future security-testing exercises may require additional isolation.

A future architecture may introduce:

```text
Internet
    │
    ▼
VMware NAT
    │
    ├────────────── Management / Updates
    │
    ▼
┌──────────────────────┐
│ Isolated Lab Network │
└──────────┬───────────┘
           │
     ┌─────┼─────┐
     │     │     │
     ▼     ▼     ▼
 Windows Wazuh  Kali
```

This would provide greater control when generating potentially suspicious traffic or performing controlled security exercises.

---

# 18. VMware Tools

VMware guest integration tools were installed where appropriate.

---

## Kali Linux

The following packages were installed:

```bash
sudo apt install open-vm-tools open-vm-tools-desktop
```

These packages improve integration between Kali and VMware Fusion.

Benefits can include:

- Improved display handling
- Clipboard integration
- Guest/host interaction
- Better VMware compatibility

---

## Ubuntu Server

Ubuntu Server does not require the desktop integration package because it operates without a graphical desktop environment.

---

# 19. Retina Display Considerations

Apple Silicon Macs commonly use Retina displays.

When initially running Windows inside VMware Fusion, interface elements may appear extremely small because of the high display resolution.

For Windows, display scaling can be adjusted through:

```text
Settings
→ System
→ Display
→ Scale
```

Example scaling values:

```text
150%
175%
200%
```

The recommended Windows resolution should generally be retained while scaling is adjusted for readability.

---

# 20. Kali Display Scaling

Kali Xfce may also require scaling adjustments on a Retina display.

Display or DPI settings can be adjusted through the Xfce appearance/display configuration if interface elements appear too small.

The goal is to adjust scaling rather than unnecessarily lowering the VM's native display quality.

---

# 21. VM Snapshots

Snapshots are used before significant configuration changes.

This provides recovery points if:

- A configuration breaks
- A security exercise damages a VM
- A package update causes problems
- A lab exercise needs to be repeated
- A clean state is required

---

# 22. Current Snapshot Strategy

## Windows

A snapshot was created after:

- Windows installation
- Windows Update
- Sysmon installation
- Sysmon configuration
- Sysmon telemetry validation

This snapshot represents the:

```text
Windows - Sysmon Baseline
```

---

## Kali Linux

A clean snapshot was created after:

- Kali installation
- System updates
- VMware Tools installation
- Basic configuration

This snapshot represents:

```text
00 - Clean Kali
```

---

## Ubuntu Server

A snapshot was created after:

- Ubuntu Server installation
- System updates
- OpenSSH installation
- Basic system configuration

The snapshot was created **before Wazuh installation**.

This represents:

```text
00 - Clean Ubuntu Server
```

---

# 23. Snapshot Limitations

In the current VMware Fusion interface, snapshot renaming was not available through the expected workflow.

Snapshots are therefore tracked based on:

- Creation order
- VM state
- Documentation
- Known configuration point

The repository documents what each snapshot represents so that recovery points remain understandable.

---

# 24. Recommended Snapshot Workflow

Before major lab changes:

```text
Known Working State
       │
       ▼
Create Snapshot
       │
       ▼
Make Configuration Change
       │
       ▼
Test
       │
   ┌───┴────┐
   │        │
 Success   Failure
   │        │
   ▼        ▼
Continue   Restore
```

Snapshots should not replace proper backups, but they are extremely useful for repeatable lab exercises.

---

# 25. Shutting Down the Lab

Virtual machines should be shut down properly when they are no longer required.

---

## Windows

Use the normal Windows shutdown process.

Alternatively:

```cmd
shutdown /s /t 0
```

---

## Kali Linux

```bash
sudo poweroff
```

---

## Ubuntu / Wazuh

```bash
sudo poweroff
```

---

# 26. Lab Startup Order

For normal SIEM exercises, a useful startup sequence is:

```text
1. Start BTL-WAZUH
        │
        ▼
2. Wait for Wazuh services
        │
        ▼
3. Start BTL-WIN11
        │
        ▼
4. Confirm Wazuh Agent is Active
        │
        ▼
5. Start BTL-KALI if required
```

`BTL-KALI` does not need to run during normal Windows/SIEM investigations.

---

# 27. Architecture Limitations

Apple Silicon provides excellent performance for ARM64 virtual machines, but there are some cybersecurity-lab limitations.

Some security tools and pre-built virtual machines are designed specifically for:

```text
x86-64 / AMD64
```

These may not operate natively inside ARM64 VMware Fusion guests.

Examples can include certain:

- Malware-analysis environments
- Legacy Windows tools
- Pre-built security appliances
- x86-only forensic utilities
- x86 virtual appliances

This should be considered when expanding the lab.

---

# 28. Malware Analysis Consideration

The current Windows ARM64 VM is designed for:

- Windows telemetry
- Sysmon
- Event logging
- Wazuh monitoring
- PowerShell analysis
- Detection exercises

It is **not intended to be the primary environment for detonating real malware**.

Future malware-analysis work may use a separate isolated x86-64 system or suitable hosted laboratory environment.

This separation reduces risk and improves tool compatibility.

---

# 29. Current VMware Lab Status

| Component | Status |
|---|---|
| VMware Fusion | ✅ Installed |
| Apple Silicon ARM64 support | ✅ Verified |
| UEFI guests | ✅ Working |
| Windows 11 ARM64 | ✅ Working |
| Windows Secure Boot | ✅ Enabled |
| Windows TPM | ✅ Enabled |
| Kali ARM64 | ✅ Working |
| Ubuntu Server ARM64 | ✅ Working |
| VMware NAT | ✅ Working |
| VM-to-VM communication | ✅ Working |
| Internet connectivity | ✅ Working |
| Kali VMware Tools | ✅ Installed |
| Retina display handling | ✅ Configured |
| VM snapshots | ✅ Created |

---

# 30. Current Lab Architecture

```text
┌───────────────────────────────────────────────────────────┐
│                  Apple M2 Max Mac                         │
│                     32 GB RAM                             │
│                                                           │
│                    VMware Fusion                          │
│                                                           │
│  ┌────────────────┐  ┌────────────────┐  ┌─────────────┐ │
│  │   BTL-WIN11    │  │   BTL-WAZUH    │  │  BTL-KALI   │ │
│  │                │  │                │  │             │ │
│  │ Windows 11     │  │ Ubuntu Server  │  │ Kali Linux  │ │
│  │ ARM64          │  │ ARM64          │  │ ARM64       │ │
│  │                │  │                │  │             │ │
│  │ 4 vCPU         │  │ 4 vCPU         │  │ 2 vCPU      │ │
│  │ 8 GB RAM       │  │ 8 GB RAM       │  │ 4 GB RAM    │ │
│  │ 64 GB Disk     │  │ 80 GB Disk     │  │ ~50-60 GB   │ │
│  │                │  │                │  │             │ │
│  │ Sysmon         │  │ Wazuh Manager  │  │ Security    │ │
│  │ Wazuh Agent    │  │ Wazuh Indexer  │  │ Testing     │ │
│  │                │  │ Wazuh Dashboard│  │             │ │
│  └───────┬────────┘  └───────▲────────┘  └──────┬──────┘ │
│          │                   │                   │        │
│          └───────────────────┼───────────────────┘        │
│                              │                            │
│                       VMware NAT                          │
│                      172.16.106.0/24                      │
└───────────────────────────────────────────────────────────┘
```

---

# 31. Next Steps

With the virtualisation platform operational, the remaining project documentation covers:

```text
03-windows-11-setup.md
        ↓
04-sysmon-setup.md
        ↓
05-kali-linux-setup.md
        ↓
06-wazuh-server-setup.md
        ↓
07-wazuh-windows-agent.md
        ↓
08-sysmon-to-wazuh.md
        ↓
SOC Investigations
```

The virtualisation layer now provides the foundation for all future Blue Team exercises.

---

## References

- [VMware Fusion Documentation](https://techdocs.broadcom.com/us/en/vmware-cis/desktop-hypervisors/fusion-pro.html)
- [Windows 11 ARM64 Download](https://www.microsoft.com/software-download/windows11arm64)
- [Ubuntu Server ARM Download](https://ubuntu.com/download/server/arm)
- [Kali Linux Downloads](https://www.kali.org/get-kali/)
- [Kali VMware Apple Silicon Guide](https://www.kali.org/docs/virtualization/install-vmware-silicon-host/)
