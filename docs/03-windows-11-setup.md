# Windows 11 Endpoint Setup (`BTL-WIN11`)

This document describes the installation and baseline configuration of the Windows 11 endpoint used in the **Blue Team Home SOC Lab**.

`BTL-WIN11` acts as the primary monitored Windows workstation and is used for Windows Event Log analysis, Sysmon telemetry, Wazuh monitoring, authentication testing, PowerShell analysis, and SOC investigation exercises.

---

# 1. Overview

The Windows endpoint provides a realistic workstation for generating and analysing security telemetry.

The endpoint is used to practise:

- Windows Event Log analysis
- User authentication monitoring
- Process execution analysis
- PowerShell activity
- Sysmon telemetry
- Wazuh endpoint monitoring
- Alert triage
- Threat hunting
- Incident investigation
- Detection validation

The VM runs **Windows 11 Pro ARM64** because the lab host uses Apple Silicon.

---

# 2. Virtual Machine Configuration

The Windows VM was created in VMware Fusion using the following configuration:

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

# 3. Why Windows 11 ARM64?

The lab host uses Apple Silicon, which uses the ARM64 architecture.

Therefore, the Windows guest uses:

```text
Windows 11 ARM64
```

rather than the traditional x86-64 version of Windows.

Windows 11 ARM provides sufficient functionality for the current Blue Team lab, including:

- Windows Event Viewer
- PowerShell
- Sysmon ARM64
- Wazuh Agent
- Windows Security logs
- Networking tools
- Endpoint telemetry generation
- SOC investigation exercises

---

# 4. Windows Installation Media

The official Windows 11 ARM64 installation image can be obtained from Microsoft:

https://www.microsoft.com/software-download/windows11arm64

Use the ARM64 version when creating the virtual machine on Apple Silicon.

---

# 5. Firmware Configuration

The Windows VM uses:

```text
UEFI
```

Legacy BIOS is not used.

---

# 6. Secure Boot

Secure Boot was enabled for the Windows endpoint:

```text
Secure Boot: Enabled
```

This reflects the configuration commonly found on modern Windows systems.

---

# 7. Trusted Platform Module (TPM)

A virtual Trusted Platform Module was enabled:

```text
TPM: Enabled
```

VMware Fusion may require VM encryption to support a virtual TPM.

For this lab, only the files required to support the TPM were encrypted rather than unnecessarily encrypting the entire virtual machine.

---

# 8. Windows Installation

Create the VM in VMware Fusion using the Windows 11 ARM64 installation image.

During installation:

1. Boot from the Windows 11 ARM64 installation media.
2. Select the required language and regional settings.
3. Install Windows 11.
4. Complete the Windows Out-of-Box Experience.
5. Allow Windows to complete its initial configuration.
6. Sign in to the desktop.

---

# 9. Windows Edition

The installed edition is:

```text
Windows 11 Pro
```

Windows 11 Pro provides additional functionality useful for IT and security lab environments.

---

# 10. Initial Windows Updates

After installation, Windows Update was used to bring the endpoint up to date.

Navigate to:

```text
Settings
→ Windows Update
→ Check for updates
```

Install available:

- Security updates
- Cumulative updates
- Defender updates
- Driver updates where appropriate

Restart the VM when required.

Repeat the update process until no important updates remain.

---

# 11. Endpoint Naming

The Windows endpoint was renamed to:

```text
BTL-WIN11
```

This provides a clear and consistent hostname for:

- Wazuh
- Event logs
- Network analysis
- Screenshots
- Investigation reports
- SIEM searches

---

## Rename Using PowerShell

Open PowerShell as Administrator:

```powershell
Rename-Computer -NewName "BTL-WIN11" -Restart
```

The computer automatically restarts after the command executes.

---

## Verify the Hostname

After restart:

```powershell
hostname
```

Expected result:

```text
BTL-WIN11
```

Alternatively:

```powershell
$env:COMPUTERNAME
```

Expected:

```text
BTL-WIN11
```

---

# 12. Lab User Accounts

Separate accounts were created to support authentication and security-monitoring exercises.

The primary lab accounts are:

| Account | Role |
|---|---|
| `labadmin` | Local Administrator |
| `testuser` | Standard User |

The endpoint may also contain system or built-in Windows accounts.

Passwords are intentionally excluded from this repository.

> Never commit lab passwords, credentials, recovery keys, tokens, or authentication secrets to GitHub.

---

# 13. Administrator Account

`labadmin` is used when administrative privileges are required.

Examples include:

- Installing Sysmon
- Installing the Wazuh Agent
- Editing Wazuh configuration
- Managing Windows services
- Performing administrative PowerShell operations
- Configuring the endpoint

---

# 14. Standard User Account

`testuser` is configured as a standard Windows user.

This account can later be used for:

- Authentication testing
- Failed logon generation
- User privilege comparisons
- Event Log analysis
- SOC investigation scenarios

Using separate administrator and standard-user accounts makes the lab more representative of a real enterprise endpoint.

---

# 15. Verify Local Users

Local users can be viewed with PowerShell:

```powershell
Get-LocalUser
```

Or:

```cmd
net user
```

---

# 16. Verify Administrator Membership

To inspect the local Administrators group:

```powershell
Get-LocalGroupMember -Group "Administrators"
```

Alternatively:

```cmd
net localgroup Administrators
```

Verify that:

```text
labadmin
```

has the required administrative privileges.

---

# 17. Windows Networking

The VM uses VMware Fusion NAT networking.

During the initial lab deployment, the endpoint received an address similar to:

```text
172.16.106.131
```

The exact address may change because DHCP is being used.

---

## Check Network Configuration

```cmd
ipconfig
```

For more detailed information:

```cmd
ipconfig /all
```

Useful information includes:

- IPv4 address
- Subnet mask
- Default gateway
- DNS servers
- Network adapter information
- DHCP status

---

# 18. Test Basic Connectivity

Test the local TCP/IP stack:

```cmd
ping 127.0.0.1
```

Test external name resolution and connectivity:

```powershell
Test-NetConnection example.com -Port 443
```

Check DNS resolution:

```powershell
Resolve-DnsName example.com
```

These commands were later also used to generate security telemetry after Sysmon was installed.

---

# 19. Windows Event Viewer

Windows Event Viewer is one of the primary tools used throughout this lab.

Open it by searching:

```text
Event Viewer
```

or using:

```cmd
eventvwr.msc
```

---

# 20. Important Windows Logs

Several Windows Event Logs are particularly useful for Blue Team investigations.

```text
Event Viewer
└── Windows Logs
    ├── Application
    ├── Security
    ├── Setup
    └── System
```

The **Security** log is particularly important for authentication and security-related activity.

---

# 21. Windows Security Log

Navigate to:

```text
Event Viewer
→ Windows Logs
→ Security
```

This log contains events relating to:

- Authentication
- Account activity
- Security policy
- Privilege use
- Logon sessions
- Audit events

---

# 22. Authentication Telemetry

Two important Windows authentication events were tested during the initial lab setup.

| Event ID | Description |
|---:|---|
| `4624` | Successful Logon |
| `4625` | Failed Logon |

These events provide important telemetry for SOC investigations.

---

# 23. Event ID 4624 — Successful Logon

Windows Security Event ID:

```text
4624
```

means:

```text
An account was successfully logged on.
```

This event can contain information such as:

- Account name
- Account domain
- Logon type
- Source workstation
- Source IP
- Authentication package
- Logon process
- Timestamp

---

# 24. Event ID 4625 — Failed Logon

Windows Security Event ID:

```text
4625
```

means:

```text
An account failed to log on.
```

This event can contain information such as:

- Attempted username
- Failure reason
- Status code
- Sub-status code
- Logon type
- Source IP
- Source workstation
- Authentication package
- Timestamp

Repeated 4625 events can be particularly useful when investigating:

- Password guessing
- Brute-force attempts
- Misconfigured services
- Stale credentials
- User login problems

---

# 25. Filtering Security Events

In Event Viewer:

```text
Windows Logs
→ Security
→ Filter Current Log...
```

Enter an Event ID such as:

```text
4624
```

or:

```text
4625
```

Multiple Event IDs can also be used when appropriate.

---

# 26. Logon Types

Windows authentication events contain a field called:

```text
Logon Type
```

The logon type describes **how the account attempted to authenticate**.

During initial local authentication testing, the lab observed:

```text
Logon Type 2
```

which represents:

```text
Interactive Logon
```

This is typically associated with a user logging directly into the Windows system.

---

# 27. Useful Logon Types

Some common Windows logon types include:

| Logon Type | Description |
|---:|---|
| `2` | Interactive |
| `3` | Network |
| `4` | Batch |
| `5` | Service |
| `7` | Unlock |
| `8` | NetworkCleartext |
| `9` | NewCredentials |
| `10` | RemoteInteractive |
| `11` | CachedInteractive |

Understanding logon types helps analysts determine how an authentication event occurred.

---

# 28. Example Authentication Investigation Logic

A single failed login may simply represent a typing mistake.

For example:

```text
4625
    │
    ▼
Single failed password
    │
    ▼
Likely user error
```

However, repeated failures can require further investigation:

```text
4625
4625
4625
4625
4625
  │
  ▼
Repeated failures
  │
  ▼
Investigate source
```

An especially interesting pattern might be:

```text
4625
4625
4625
4625
4625
  │
  ▼
4624
```

This could indicate repeated failed attempts followed by successful authentication.

The analyst would then investigate:

- Username
- Source IP
- Source workstation
- Logon type
- Failure reason
- Timestamp
- Authentication method
- Activity after successful authentication

---

# 29. Windows PowerShell

PowerShell is used extensively throughout the lab for:

- System administration
- Network testing
- Process inspection
- Service management
- Telemetry generation
- Security investigation

Useful baseline commands include:

```powershell
whoami
```

```powershell
hostname
```

```powershell
ipconfig
```

```powershell
Get-Process
```

```powershell
Get-Service
```

```powershell
Resolve-DnsName example.com
```

```powershell
Test-NetConnection example.com -Port 443
```

These commands later become useful for both administration and telemetry-generation exercises.

---

# 30. Windows Defender

Microsoft Defender remains available on the Windows endpoint.

The lab is primarily designed for defensive monitoring, so endpoint security controls should generally remain enabled unless a specific controlled exercise requires otherwise.

Do not disable security controls simply to make testing easier.

---

# 31. Display Scaling on Apple Silicon

When Windows was initially launched through VMware Fusion on the Mac's Retina display, interface elements appeared very small.

Windows display scaling was adjusted through:

```text
Settings
→ System
→ Display
→ Scale
```

Useful values may include:

```text
150%
175%
200%
```

The recommended display resolution can generally remain enabled while scaling is adjusted for readability.

---

# 32. Windows Baseline Validation

Before installing additional security tooling, verify:

- [x] Windows boots successfully
- [x] Windows Update completed
- [x] Hostname configured as `BTL-WIN11`
- [x] `labadmin` account available
- [x] `testuser` account available
- [x] Administrator privileges verified
- [x] VMware NAT networking operational
- [x] Internet connectivity operational
- [x] DNS resolution operational
- [x] Event Viewer operational
- [x] Windows Security log accessible
- [x] Event ID 4624 observed
- [x] Event ID 4625 observed

---

# 33. Security Baseline

At this stage, the endpoint provides the following native telemetry:

```text
Windows
   │
   ├── Security Log
   │      ├── 4624 Successful Logon
   │      └── 4625 Failed Logon
   │
   ├── System Log
   │
   ├── Application Log
   │
   └── PowerShell / OS Telemetry
```

The next stage expands this visibility significantly using Microsoft Sysmon.

---

# 34. Why Native Windows Logs Are Not Enough

Windows provides substantial native security telemetry.

However, SOC analysts often require deeper visibility into activity such as:

- Process creation
- Parent/child process relationships
- Command-line arguments
- Network connections
- DNS queries
- Process access
- File creation
- Image/DLL loading

This is where Sysmon becomes valuable.

The telemetry architecture evolves from:

```text
Windows Activity
      │
      ▼
Windows Event Logs
```

to:

```text
Windows Activity
      │
      ├───────────────┐
      ▼               ▼
Windows Logs        Sysmon
      │               │
      └───────┬───────┘
              ▼
       Security Monitoring
```

---

# 35. ARM64 Security Tool Considerations

Because `BTL-WIN11` runs Windows 11 ARM64, software compatibility should be verified before installing security tools.

Windows 11 ARM can run many traditional Windows applications through compatibility/emulation functionality, but this does not guarantee that every security tool will behave identically to its x86-64 equivalent.

For this lab:

```text
Microsoft Sysmon ARM64
```

is used natively.

The Wazuh Windows Agent is also used for endpoint monitoring and has been validated within this lab environment.

---

# 36. Malware Safety

`BTL-WIN11` is primarily a:

```text
Monitoring
+
Telemetry
+
Detection
+
Investigation
```

endpoint.

It is **not currently intended for uncontrolled real-malware execution**.

Future malware-analysis exercises should use appropriately isolated infrastructure designed specifically for that purpose.

---

# 37. Snapshot Strategy

After the Windows baseline was established and Sysmon was later installed successfully, a snapshot was created.

The documented recovery point represents:

```text
Windows - Sysmon Baseline
```

This provides a known-working state for future experiments.

Snapshots should be created before major configuration changes or security exercises.

---

# 38. Shutdown

Windows should be shut down normally when the VM is not required.

From the Windows interface:

```text
Start
→ Power
→ Shut down
```

Or from Command Prompt:

```cmd
shutdown /s /t 0
```

This preserves host resources when performing theory, documentation, or work that does not require the endpoint.

---

# 39. Current Endpoint Role

Within the lab architecture:

```text
                     BTL-WIN11
                         │
                         │
              Windows Workstation
                         │
             ┌───────────┴───────────┐
             │                       │
             ▼                       ▼
      Windows Event Logs           Sysmon
             │                       │
             └───────────┬───────────┘
                         │
                         ▼
                    Wazuh Agent
                         │
                         ▼
                    BTL-WAZUH
                         │
                         ▼
                 SOC Investigation
```

---

# 40. Skills Practised

The Windows endpoint setup provided hands-on experience with:

- Windows 11 deployment
- ARM64 virtualisation
- UEFI
- Secure Boot
- Virtual TPM
- Windows administration
- Local user management
- Administrator vs standard-user accounts
- Windows networking
- PowerShell
- Windows Event Viewer
- Security Event Logs
- Authentication telemetry
- Event ID 4624
- Event ID 4625
- Logon types
- Endpoint baseline creation
- VM snapshot strategy

---

# 41. Setup Result

The Windows endpoint baseline was successfully completed.

```text
Windows 11 ARM64
        │
        ▼
System Updated
        │
        ▼
Hostname: BTL-WIN11
        │
        ▼
Lab Accounts Created
        │
        ▼
Networking Verified
        │
        ▼
Event Viewer Verified
        │
        ▼
Authentication Logs Verified
        │
        ▼
Ready for Sysmon
```

---

# 42. Next Step

The next stage installs and validates Microsoft Sysmon to provide enhanced endpoint telemetry.

Continue with:

```text
docs/04-sysmon-setup.md
```

The following stage then connects the endpoint to the central Wazuh SIEM:

```text
docs/07-wazuh-windows-agent.md
```

---

## References

- [Windows 11 ARM64 Download](https://www.microsoft.com/software-download/windows11arm64)
- [Microsoft Windows Security Auditing](https://learn.microsoft.com/windows/security/threat-protection/auditing/basic-security-audit-policies)
- [Microsoft Event 4624 Documentation](https://learn.microsoft.com/windows/security/threat-protection/auditing/event-4624)
- [Microsoft Event 4625 Documentation](https://learn.microsoft.com/windows/security/threat-protection/auditing/event-4625)
- [Microsoft PowerShell Documentation](https://learn.microsoft.com/powershell/)
- [Microsoft Sysmon Documentation](https://learn.microsoft.com/sysinternals/downloads/sysmon)
