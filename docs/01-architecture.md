# Blue Team Home SOC Lab Architecture

This document describes the architecture of the **Blue Team Home SOC Lab**, including the host system, virtual machines, networking, telemetry pipeline, security monitoring components, and planned future expansion.

---

# 1. Architecture Overview

The lab is designed as a small Security Operations Center (SOC) environment for practising:

- Windows security monitoring
- Endpoint telemetry collection
- SIEM operations
- Sysmon analysis
- Alert triage
- Threat hunting
- Incident investigation
- Network analysis
- Detection engineering
- Blue Team workflows

The environment is hosted on an **Apple Silicon Mac** using VMware Fusion.

The current architecture consists of three primary virtual machines:

1. `BTL-WIN11` — monitored Windows endpoint
2. `BTL-WAZUH` — SIEM / security monitoring server
3. `BTL-KALI` — security testing and network analysis workstation

---

# 2. High-Level Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                    Apple Silicon Mac                        │
│                                                             │
│                    VMware Fusion                            │
│                                                             │
│   ┌────────────────┐     ┌────────────────────────────┐     │
│   │   BTL-WIN11    │     │         BTL-WAZUH          │     │
│   │                │     │                            │     │
│   │ Windows 11     │────▶│ Ubuntu Server ARM64        │     │
│   │ ARM64          │     │                            │     │
│   │                │     │ Wazuh Manager              │     │
│   │ Sysmon         │     │ Wazuh Indexer              │     │
│   │ Wazuh Agent    │     │ Wazuh Dashboard            │     │
│   └────────────────┘     └────────────────────────────┘     │
│           ▲                         ▲                        │
│           │                         │                        │
│           │                         │                        │
│   ┌───────┴────────┐                │                        │
│   │    BTL-KALI    │────────────────┘                        │
│   │                │                                         │
│   │ Kali Linux     │                                         │
│   │ ARM64          │                                         │
│   │                │                                         │
│   │ Testing /      │                                         │
│   │ Analysis       │                                         │
│   └────────────────┘                                         │
│                                                             │
│                VMware NAT Network                           │
└─────────────────────────────────────────────────────────────┘
```

---

# 3. Host System

The lab runs on an Apple Silicon workstation.

## Host Configuration

| Component | Configuration |
|---|---|
| Platform | Apple Silicon |
| Architecture | ARM64 |
| Virtualisation | VMware Fusion |
| Host RAM | 32 GB |
| Networking | VMware NAT |
| Guest Firmware | UEFI |

Because the host uses Apple Silicon, the virtual machines primarily use ARM64-compatible operating systems.

---

# 4. Virtual Machine Architecture

## 4.1 `BTL-WIN11`

`BTL-WIN11` acts as the primary monitored endpoint.

It represents a Windows workstation that would exist inside an enterprise environment.

### Configuration

| Component | Configuration |
|---|---|
| Hostname | `BTL-WIN11` |
| Operating System | Windows 11 Pro |
| Architecture | ARM64 |
| vCPU | 4 |
| RAM | 8 GB |
| Disk | 64 GB |
| Firmware | UEFI |
| Secure Boot | Enabled |
| TPM | Enabled |
| Network | VMware NAT |

### Security Components

The endpoint currently contains:

- Windows Event Logging
- Microsoft Sysmon
- Wazuh Windows Agent

### Lab Accounts

Two local accounts are used for testing:

```text
labadmin   - Administrator
testuser   - Standard User
```

Passwords and credentials are intentionally excluded from this repository.

---

# 5. `BTL-WAZUH`

`BTL-WAZUH` provides centralized security monitoring and SIEM functionality.

### Configuration

| Component | Configuration |
|---|---|
| Hostname | `btl-wazuh` |
| Operating System | Ubuntu Server |
| Architecture | ARM64 |
| vCPU | 4 |
| RAM | 8 GB |
| Disk | 80 GB |
| Network | VMware NAT |

### Wazuh Components

The server uses an all-in-one Wazuh deployment containing:

```text
Wazuh Manager
Wazuh Indexer
Wazuh Dashboard
```

The server receives security telemetry from monitored endpoints and applies detection rules to identify potentially suspicious activity.

---

# 6. `BTL-KALI`

`BTL-KALI` is the security testing and investigation workstation.

It will be used to generate controlled activity against lab systems and practise network/security analysis.

### Configuration

| Component | Configuration |
|---|---|
| Hostname | `btl-kali` |
| Operating System | Kali Linux |
| Architecture | ARM64 |
| Desktop | Xfce |
| vCPU | 2 |
| RAM | 4 GB |
| Disk | ~50–60 GB |
| Network | VMware NAT |

### Intended Uses

`BTL-KALI` will be used for:

- Network reconnaissance
- Controlled security testing
- Traffic generation
- Network analysis
- DNS testing
- Authentication testing
- Detection validation
- Blue Team investigation exercises

All testing is performed only against systems within the authorised lab environment.

---

# 7. Current Network Architecture

All virtual machines currently communicate through the VMware NAT network.

During the initial deployment, the lab used the following subnet:

```text
172.16.106.0/24
```

Example addresses observed during the build were:

```text
BTL-WIN11  → 172.16.106.131
BTL-KALI   → 172.16.106.132
BTL-WAZUH  → 172.16.106.134
```

These addresses are assigned using DHCP and may change.

For this reason, documentation uses placeholders such as:

```text
<WAZUH-SERVER-IP>
<WINDOWS-ENDPOINT-IP>
<KALI-IP>
```

where appropriate.

---

# 8. Network Diagram

```text
                         Internet
                            │
                            │
                     VMware NAT
                            │
              ┌─────────────┴─────────────┐
              │                           │
       Apple Silicon Host                 │
              │                           │
        VMware Fusion                     │
              │                           │
      172.16.106.0/24                     │
              │                           │
     ┌────────┼───────────────┐           │
     │        │               │           │
     ▼        ▼               ▼           │
 BTL-WIN11  BTL-WAZUH      BTL-KALI      │
     │        ▲               │           │
     │        │               │           │
     └────────┴───────────────┘           │
           Lab Communication             │
```

---

# 9. Windows Telemetry Architecture

The Windows endpoint generates several types of telemetry.

```text
┌───────────────────────┐
│      BTL-WIN11        │
│                       │
│   User / OS Activity  │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│ Windows Event Logging │
└───────────┬───────────┘
            │
            ├────────────────────┐
            │                    │
            ▼                    ▼
┌─────────────────┐     ┌──────────────────┐
│ Security Events │     │ Microsoft Sysmon │
│                 │     │                  │
│ 4624            │     │ Event ID 1       │
│ 4625            │     │ Event ID 3       │
│ etc.            │     │ Event ID 10      │
│                 │     │ Event ID 22      │
└────────┬────────┘     └─────────┬────────┘
         │                        │
         └────────────┬───────────┘
                      │
                      ▼
               ┌─────────────┐
               │ Wazuh Agent │
               └──────┬──────┘
                      │
                      ▼
                  BTL-WAZUH
```

---

# 10. Sysmon Telemetry

Microsoft Sysmon provides enhanced endpoint visibility.

Sysmon telemetry is stored locally in:

```text
Applications and Services Logs
└── Microsoft
    └── Windows
        └── Sysmon
            └── Operational
```

The event channel is:

```text
Microsoft-Windows-Sysmon/Operational
```

The following Sysmon events have been observed and/or investigated during the lab:

| Event ID | Description |
|---:|---|
| `1` | Process Creation |
| `3` | Network Connection |
| `7` | Image Loaded |
| `10` | Process Access |
| `11` | File Create |
| `22` | DNS Query |

---

# 11. Wazuh Collection Architecture

The Wazuh Agent installed on `BTL-WIN11` collects security telemetry from Windows.

Relevant sections of the agent configuration include:

```xml
<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

This creates the following collection pipeline:

```text
Windows Security Events ──────┐
                              │
                              ▼
                         Wazuh Agent
                              │
                              │
Sysmon Operational Events ────┘
                              │
                              ▼
                         BTL-WAZUH
```

---

# 12. Detection Pipeline

The complete monitoring pipeline currently looks like:

```text
User / System Activity
          │
          ▼
Windows / Sysmon Telemetry
          │
          ▼
Windows Event Channels
          │
          ▼
Wazuh Agent
          │
          ▼
Wazuh Manager
          │
          ▼
Decoders / Detection Rules
          │
          ▼
Does activity match detection logic?
          │
      ┌───┴────┐
      │        │
     YES       NO
      │        │
      ▼        ▼
    Alert    No Alert
      │
      ▼
SOC Investigation
      │
      ▼
┌─────────────┬──────────────┐
│             │              │
▼             ▼              ▼
Benign     Suspicious     Malicious
                              │
                              ▼
                           Incident
```

---

# 13. Telemetry vs Detection vs Alert

A major learning objective of this lab is understanding the distinction between raw security telemetry and actionable alerts.

## Telemetry

Telemetry represents activity recorded by systems or security tools.

Example:

```text
Sysmon Event ID: 1
Image: C:\Windows\System32\whoami.exe
CommandLine: whoami
```

---

## Detection

Detection rules analyse telemetry for behaviour that may be interesting or suspicious.

For example:

```text
PowerShell
      │
      ▼
Creates suspicious file
      │
      ▼
Detection rule matches
```

---

## Alert

When detection logic matches activity, the SIEM may generate an alert for analyst review.

```text
Telemetry
    │
    ▼
Detection Rule
    │
    ▼
Alert
```

---

## Investigation

The analyst then determines whether the alert represents legitimate or malicious activity.

```text
Alert
  │
  ▼
Triage
  │
  ▼
Collect Evidence
  │
  ▼
Correlate Activity
  │
  ▼
Determine Verdict
```

An important principle demonstrated by the lab is:

> **An event is not necessarily an alert, and an alert is not necessarily an incident.**

---

# 14. Example Investigation Flow

The first documented investigation involved OneDrive accessing Windows Explorer.

```text
OneDrive.exe
     │
     │ Process Access
     ▼
Explorer.EXE
```

Sysmon recorded:

```text
Event ID 10 - Process Access
```

Wazuh generated:

```text
Rule ID: 92910
Severity: Level 12
Possible Process Injection
```

The analyst investigation examined:

```text
SourceImage
TargetImage
GrantedAccess
CallTrace
SourceUser
TargetUser
```

The call trace contained legitimate-looking OneDrive modules including:

```text
FileSyncClient.dll
FileSyncEvents.dll
FileSyncHost.DLL
```

The activity was ultimately assessed as:

```text
Likely Benign
```

This demonstrated the importance of investigating context rather than relying only on alert severity.

The full investigation is documented at:

```text
investigations/001-onedrive-explorer-process-access.md
```

---

# 15. Current Architecture Status

The following components are operational:

| Component | Status |
|---|---|
| VMware Fusion | ✅ Operational |
| Windows 11 VM | ✅ Operational |
| Kali Linux VM | ✅ Operational |
| Ubuntu Server VM | ✅ Operational |
| Microsoft Sysmon | ✅ Operational |
| Windows Event Logging | ✅ Operational |
| Wazuh Manager | ✅ Operational |
| Wazuh Indexer | ✅ Operational |
| Wazuh Dashboard | ✅ Operational |
| Wazuh Windows Agent | ✅ Active |
| Windows Security Log Collection | ✅ Configured |
| Sysmon Log Collection | ✅ Configured |
| Wazuh Alerting | ✅ Verified |
| SOC Investigation Workflow | ✅ Tested |

---

# 16. VM Snapshots

Snapshots are used to provide recovery points before major configuration changes.

Current recovery points include:

### Windows

```text
Windows - Sysmon Baseline
```

Represents the Windows endpoint after successful Sysmon installation and validation.

### Kali Linux

```text
00 - Clean Kali
```

Represents the clean Kali installation after initial updates and VMware integration.

### Ubuntu / Wazuh

```text
00 - Clean Ubuntu Server
```

Represents the Ubuntu Server state before Wazuh installation.

Additional snapshots may be created before future security-testing scenarios.

---

# 17. Current Security Boundary

At the current stage, the environment uses VMware NAT networking.

The lab is intended exclusively for:

- Defensive security education
- Authorised testing
- Security monitoring
- Detection validation
- SOC investigations

Testing must remain limited to systems owned and controlled within the lab.

---

# 18. Planned Architecture Expansion

The current architecture provides the foundation for future Blue Team exercises.

Planned additions include:

```text
                     ┌──────────────┐
                     │   Internet   │
                     └──────┬───────┘
                            │
                      VMware NAT
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
        BTL-WIN11       BTL-WAZUH      BTL-KALI
             │              ▲              │
             │              │              │
             │         Telemetry            │
             └──────────────┘              │
             ▲                             │
             │                             │
             └──── Controlled Testing ─────┘

                     Future Components

                    ┌──────────────┐
                    │    Splunk    │
                    │   Practice   │
                    └──────────────┘

                    ┌──────────────┐
                    │  Wireshark   │
                    │ PCAP Analysis│
                    └──────────────┘

                    ┌──────────────┐
                    │    Sigma     │
                    │  Detection   │
                    └──────────────┘
```

Potential future capabilities include:

- Controlled Kali-to-Windows security testing
- Failed authentication simulations
- Brute-force detection exercises
- PowerShell investigations
- DNS investigations
- Network connection investigations
- Wireshark / PCAP analysis
- Splunk / SPL training
- Sigma detection engineering
- Threat intelligence enrichment
- Phishing analysis
- Incident-response exercises

---

# 19. Resource Allocation

Current VM resource allocation:

| System | vCPU | RAM | Disk |
|---|---:|---:|---:|
| `BTL-WIN11` | 4 | 8 GB | 64 GB |
| `BTL-WAZUH` | 4 | 8 GB | 80 GB |
| `BTL-KALI` | 2 | 4 GB | ~50–60 GB |
| **Total** | **10 vCPU** | **20 GB** | **~194–204 GB** |

Because the host contains 32 GB RAM, not all VMs need to remain powered on when they are not required for an exercise.

For example:

```text
Theory / Documentation
        ↓
All VMs OFF

Windows Event Practice
        ↓
BTL-WIN11 ON

SIEM Investigation
        ↓
BTL-WIN11 + BTL-WAZUH ON

Controlled Security Exercise
        ↓
BTL-WIN11 + BTL-WAZUH + BTL-KALI ON
```

---

# 20. Repository Architecture

The project documentation is organised as:

```text
blue-team-home-soc-lab/
│
├── README.md
│
├── docs/
│   ├── 01-architecture.md
│   ├── 02-vmware-fusion-setup.md
│   ├── 03-windows-11-setup.md
│   ├── 04-sysmon-setup.md
│   ├── 05-kali-linux-setup.md
│   ├── 06-wazuh-server-setup.md
│   ├── 07-wazuh-windows-agent.md
│   ├── 08-sysmon-to-wazuh.md
│   ├── 09-troubleshooting.md
│   └── 10-snapshots-and-recovery.md
│
├── investigations/
│   ├── README.md
│   ├── 001-onedrive-explorer-process-access.md
│   └── investigation-template.md
│
├── cheatsheets/
│   ├── windows-event-ids.md
│   ├── sysmon-event-ids.md
│   ├── powershell.md
│   ├── linux.md
│   ├── wazuh-searches.md
│   └── networking.md
│
├── learning/
│   ├── soc-fundamentals.md
│   ├── mitre-attack.md
│   ├── windows-logging.md
│   └── btl1-preparation-roadmap.md
│
└── screenshots/
    ├── architecture/
    ├── windows/
    ├── sysmon/
    ├── wazuh/
    └── investigations/
```

---

# 21. Architecture Goals

The architecture is designed around four primary goals:

### 1. Visibility

Generate detailed endpoint telemetry using Windows Event Logs and Sysmon.

### 2. Centralisation

Forward endpoint telemetry to Wazuh for centralised security monitoring.

### 3. Detection

Use SIEM detection rules to identify potentially suspicious activity.

### 4. Investigation

Develop the ability to analyse alerts, correlate evidence, determine scope, and classify activity as benign or malicious.

The overall learning workflow is:

```text
BUILD
  ↓
GENERATE TELEMETRY
  ↓
DETECT
  ↓
INVESTIGATE
  ↓
DOCUMENT
  ↓
IMPROVE DETECTIONS
  ↓
REPEAT
```

---

## References

- [VMware Fusion Documentation](https://techdocs.broadcom.com/us/en/vmware-cis/desktop-hypervisors/fusion-pro.html)
- [Microsoft Sysmon Documentation](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Kali Linux Documentation](https://www.kali.org/docs/)
- [MITRE ATT&CK](https://attack.mitre.org/)
