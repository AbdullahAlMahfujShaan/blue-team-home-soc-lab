Blue Team Home SOC Lab
======================

A hands-on Security Operations / Blue Team homelab built to develop practical skills in Windows telemetry, Sysmon, SIEM monitoring, alert triage, threat hunting, incident investigation, and defensive security.

The environment runs primarily on Apple Silicon and uses Windows 11, Sysmon, Wazuh, Kali Linux, and VMware Fusion.

Objectives
----------

The goal of this project is to develop practical SOC analyst skills by building and operating a small security monitoring environment rather than relying only on theoretical study.

The lab is used to practise:

-   Windows Event Log analysis

-   Sysmon telemetry

-   SIEM monitoring

-   Alert triage

-   Process investigation

-   Threat hunting

-   Incident-response workflow

-   MITRE ATT&CK mapping

-   Networking fundamentals

-   PowerShell and Linux

-   Detection engineering

-   Blue-team certification preparation

Lab Architecture
----------------

Host:

-   Apple Mac with Apple Silicon

-   VMware Fusion

Virtual machines:

### BTL-WIN11

Windows 11 endpoint used to generate and investigate security telemetry.

Configuration:

-   Windows 11 Pro ARM64

-   4 vCPU

-   8 GB RAM

-   64 GB disk

-   VMware NAT networking

-   Sysmon installed

-   Wazuh agent installed

### BTL-WAZUH

Central SIEM and monitoring server.

Configuration:

-   Ubuntu Server ARM64

-   4 vCPU

-   8 GB RAM

-   80 GB disk

-   Wazuh Manager

-   Wazuh Indexer

-   Wazuh Dashboard

### BTL-KALI

Linux security workstation used for networking exercises and controlled lab testing.

Configuration:

-   Kali Linux ARM64

-   2 vCPU

-   4 GB RAM

-   VMware NAT networking

Lab Network
-----------

The lab currently uses VMware NAT.

Example lab subnet:

```
172.16.106.0/24
```

Example hosts during initial build:

```
BTL-WIN11  - 172.16.106.131
BTL-KALI   - 172.16.106.132
BTL-WAZUH  - 172.16.106.134
```

Addresses may change because DHCP is currently being used.

Monitoring Pipeline
-------------------

```
Windows Activity
      |
      v
Windows Event Logs
      |
      v
Sysmon
      |
      v
Wazuh Agent
      |
      v
Wazuh Manager
      |
      v
Detection Rules
      |
      v
Wazuh Alerts
      |
      v
SOC Investigation
```

Completed Work
--------------

### Virtualisation

-   Installed VMware Fusion on Apple Silicon

-   Created Windows 11 ARM64 endpoint

-   Created Kali Linux ARM64 workstation

-   Created Ubuntu ARM64 Wazuh server

-   Configured VMware NAT networking

-   Created clean VM snapshots

### Windows Endpoint

-   Installed Windows 11 Pro ARM64

-   Created administrator and standard-user lab accounts

-   Configured Windows Update

-   Practised Windows Event Viewer

-   Generated successful and failed authentication events

### Sysmon

Installed Microsoft Sysmon using the ARM64 binary.

Configuration was based on Olaf Hartong's sysmon-modular project.

Telemetry tested includes:

-   Event ID 1 --- Process Creation

-   Event ID 3 --- Network Connection

-   Event ID 7 --- Image Loaded

-   Event ID 10 --- Process Access

-   Event ID 11 --- File Create

-   Event ID 22 --- DNS Query

### Wazuh

-   Installed Wazuh all-in-one on Ubuntu Server

-   Configured Wazuh Manager

-   Configured Wazuh Indexer

-   Configured Wazuh Dashboard

-   Installed Wazuh Windows agent

-   Successfully enrolled BTL-WIN11

-   Forwarded Windows Security Event Logs

-   Forwarded Sysmon Operational logs

-   Verified endpoint telemetry

-   Investigated Wazuh alerts

First SOC Investigation
-----------------------

The first documented investigation involved:

```
OneDrive.exe
      |
      | Process Access
      v
Explorer.EXE
```

Wazuh generated:

```
Rule ID: 92910
Severity: Level 12
Description:
Explorer process was accessed by OneDrive.exe,
possible process injection
```

Investigation identified:

```
Sysmon Event ID: 10
SourceImage: OneDrive.exe
TargetImage: Explorer.EXE
GrantedAccess: 0x101411
SourceUser: BTL-WIN11\labadmin
TargetUser: BTL-WIN11\labadmin
```

The call trace contained legitimate-looking Microsoft OneDrive modules including:

```
FileSyncClient.dll
FileSyncEvents.dll
FileSyncHost.DLL
```

### Analyst Verdict

Likely benign activity.

The exercise demonstrated an important SOC principle:

> An alert is not automatically an incident.

Severity determines what deserves investigation. Evidence determines whether activity is malicious.

Full investigation:

`investigations/001-onedrive-explorer-process-access.md`

Repository Documentation
------------------------

Detailed build documentation is available under:

```
docs/
```

Investigation reports:

```
investigations/
```

Command and event references:

```
cheatsheets/
```

Study notes:

```
learning/
```

Current Learning Focus
----------------------

Current areas of development include:

-   SOC fundamentals

-   SIEM investigation

-   Windows Event Logs

-   Sysmon

-   Wazuh

-   Splunk and SPL

-   Network traffic analysis

-   Wireshark

-   Phishing analysis

-   Threat intelligence

-   Digital forensics

-   Incident response

-   MITRE ATT&CK

Future Lab Development
----------------------

Planned additions include:

-   Splunk practice

-   Wireshark/PCAP investigations

-   Authentication attack simulation

-   Controlled Kali-to-Windows activity

-   Sigma detection rules

-   Additional incident reports

-   Threat-intelligence enrichment

-   Phishing investigations

-   Detection engineering

-   Expanded network segmentation

Security Notice
---------------

This repository does not contain:

-   passwords

-   API keys

-   private tokens

-   licence keys

-   sensitive personal data

-   VM disk images

-   malicious binaries

All exercises are performed in an isolated lab environment.

Disclaimer
----------

This project is for defensive cybersecurity education and authorised laboratory testing only.
