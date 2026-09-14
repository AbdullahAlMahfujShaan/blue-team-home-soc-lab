# Sysmon to Wazuh Integration (`BTL-WIN11`)

This document outlines the configuration and validation process used to forward **Microsoft Sysmon telemetry** from the Windows endpoint `BTL-WIN11` to the Wazuh SIEM server.

---

## Overview

Sysmon provides detailed endpoint telemetry including process creation, network connections, DNS queries, file creation, process access, and other security-relevant activity.

The Wazuh Agent installed on `BTL-WIN11` was configured to collect events from the Sysmon Operational event channel and forward them to the Wazuh Manager.

The resulting monitoring pipeline is:

```text
Windows Activity
      │
      ▼
Microsoft Sysmon
      │
      ▼
Sysmon Operational Log
      │
      ▼
Wazuh Windows Agent
      │
      ▼
Wazuh Manager
      │
      ▼
Wazuh Detection Rules
      │
      ▼
Wazuh Alerts
      │
      ▼
SOC Investigation
```

---

# Prerequisites

Before configuring the integration, the following components were already operational:

- [x] Windows 11 endpoint configured as `BTL-WIN11`
- [x] Microsoft Sysmon installed
- [x] Sysmon configuration loaded
- [x] Sysmon Operational event log working
- [x] Wazuh Windows Agent installed
- [x] Wazuh Agent enrolled with the Wazuh Manager
- [x] Wazuh Agent status displayed as `Active`
- [x] Wazuh Dashboard accessible

---

# Sysmon Event Channel

Sysmon writes its telemetry to the following Windows Event Log channel:

```text
Microsoft-Windows-Sysmon/Operational
```

This can be viewed through Event Viewer:

```text
Applications and Services Logs
└── Microsoft
    └── Windows
        └── Sysmon
            └── Operational
```

Before configuring Wazuh, Sysmon telemetry was verified locally using Windows Event Viewer.

---

# Wazuh Agent Configuration

The Wazuh Windows Agent configuration file is located at:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Because this file is located inside `Program Files`, editing it may require Administrator privileges.

---

## 1. Open the Configuration File

Open **Notepad as Administrator**.

Then open:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Alternatively, from an elevated PowerShell session:

```powershell
notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"
```

---

## 2. Existing Windows Security Log Collection

The Wazuh Agent configuration already contained a `<localfile>` block for collecting the Windows Security event channel:

```xml
<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
</localfile>
```

This allows Wazuh to process security events such as Windows authentication activity.

Examples include:

```text
4624 - Successful Logon
4625 - Failed Logon
```

---

## 3. Add Sysmon Event Collection

To collect Sysmon events, the following `<localfile>` block was added to `ossec.conf`:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

The relevant configuration therefore included both Windows Security and Sysmon event channels:

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

Save the configuration file after making the change.

---

# Restart the Wazuh Agent

Configuration changes require the Wazuh Agent service to be restarted.

Open an elevated Command Prompt or PowerShell session.

Stop the service:

```cmd
NET STOP Wazuh
```

Start it again:

```cmd
NET START Wazuh
```

Alternatively, PowerShell can be used:

```powershell
Restart-Service -Name Wazuh
```

---

# Verify Agent Status

Verify that the Wazuh Agent returned to a running state:

```powershell
Get-Service -Name Wazuh
```

Expected status:

```text
Status   Name    DisplayName
------   ----    -----------
Running  Wazuh   Wazuh
```

The Wazuh Dashboard should also continue to show:

```text
Agent: BTL-WIN11
Status: Active
```

---

# Generate Test Telemetry

After configuring Sysmon collection, controlled activity was generated on `BTL-WIN11`.

## Process Activity

```powershell
ping 127.0.0.1
```

Additional process creation tests:

```powershell
whoami
hostname
ipconfig
```

These commands generated **Sysmon Event ID 1 — Process Creation** events locally.

---

## DNS Activity

```powershell
Resolve-DnsName example.com
```

This generated DNS telemetry associated with:

```text
Sysmon Event ID 22 - DNS Query
```

---

## Network Activity

```powershell
Test-NetConnection example.com -Port 443
```

This generated network telemetry associated with:

```text
Sysmon Event ID 3 - Network Connection
```

---

# Verify Sysmon Locally

Before investigating the Wazuh side, the generated events were confirmed locally.

Open:

```text
Event Viewer
→ Applications and Services Logs
→ Microsoft
→ Windows
→ Sysmon
→ Operational
```

Useful Sysmon Event IDs observed during the lab include:

| Event ID | Description |
|---:|---|
| `1` | Process Creation |
| `3` | Network Connection |
| `7` | Image Loaded |
| `10` | Process Access |
| `11` | File Create |
| `22` | DNS Query |

---

# Verify Telemetry in Wazuh

Open the Wazuh Dashboard and navigate to the event/threat-hunting view.

Filter for the Windows endpoint:

```text
agent.name: "BTL-WIN11"
```

Sysmon-derived Wazuh alerts were successfully observed for the endpoint.

Examples encountered during testing included detections related to:

- PowerShell activity
- Discovery activity
- Process access
- DLL/module loading
- File creation
- Windows process behaviour

This confirmed that Sysmon telemetry was successfully reaching the Wazuh detection pipeline.

---

# Important Observation: Raw Telemetry vs Alerts

During validation, an important SIEM concept was identified.

The command:

```powershell
whoami
```

was executed on `BTL-WIN11`.

Windows created:

```text
whoami.exe
```

Sysmon successfully recorded:

```text
Event ID 1 - Process Creation
```

The event was visible locally in:

```text
Microsoft-Windows-Sysmon/Operational
```

A `ProcessGuid` from the event was then searched in the Wazuh alerts view.

The search returned:

```text
0 hits
```

This did **not** mean that Sysmon or the Wazuh Agent was broken.

Instead, it demonstrated an important distinction between **telemetry** and **alerts**.

---

## Telemetry Pipeline

A simplified detection pipeline looks like:

```text
Activity occurs
      │
      ▼
Sysmon records event
      │
      ▼
Wazuh Agent collects event
      │
      ▼
Wazuh analyses event
      │
      ▼
Does activity match a detection rule?
      │
      ├───────────────┐
      │               │
     YES              NO
      │               │
      ▼               ▼
   Alert          No alert generated
```

Therefore:

> A raw Sysmon event does not necessarily appear as a Wazuh alert.

This distinction is important when performing SIEM investigations.

---

# Telemetry vs Detection vs Alert

The lab demonstrated the following concepts:

### Telemetry

Raw information generated by systems and security tools.

Example:

```text
Sysmon Event ID 1
Image: C:\Windows\System32\whoami.exe
CommandLine: whoami
```

### Detection

Logic that analyses telemetry for behaviour considered interesting or suspicious.

Example:

```text
PowerShell creates an unusual file
```

### Alert

A detection rule matches activity and generates something for an analyst to investigate.

Example:

```text
Wazuh Rule 92910
Possible process injection
Severity Level 12
```

### Incident

An alert becomes an incident only after investigation determines that malicious or unauthorised activity occurred.

Therefore:

```text
Telemetry
    ↓
Detection
    ↓
Alert
    ↓
Investigation
    ↓
Benign or Malicious?
    ↓
Incident if required
```

---

# Example Wazuh Detection

One of the alerts observed during testing involved:

```text
OneDrive.exe
      │
      │ Process Access
      ▼
Explorer.EXE
```

Wazuh generated:

```text
Rule ID: 92910
Level: 12
Description:
Explorer process was accessed by OneDrive.exe,
possible process injection
```

The underlying telemetry was:

```text
Sysmon Event ID 10 - Process Access
```

This alert was subsequently investigated and determined to be consistent with legitimate OneDrive activity.

The complete investigation is documented separately:

```text
investigations/001-onedrive-explorer-process-access.md
```

---

# Troubleshooting

## Sysmon Events Appear Locally but Not as Wazuh Alerts

First verify that the event exists locally:

```text
Event Viewer
→ Applications and Services Logs
→ Microsoft
→ Windows
→ Sysmon
→ Operational
```

If the event exists locally but does not appear in the Wazuh alerts view, this does not automatically indicate a forwarding failure.

The event may simply not have triggered a Wazuh detection rule.

---

## Verify Wazuh Agent Service

```powershell
Get-Service -Name Wazuh
```

Expected:

```text
Running
```

If required:

```cmd
NET STOP Wazuh
NET START Wazuh
```

---

## Inspect Wazuh Agent Logs

The Windows Wazuh Agent log is located at:

```text
C:\Program Files (x86)\ossec-agent\ossec.log
```

View recent entries:

```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 100
```

Monitor the log continuously:

```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Wait
```

---

## Verify the Sysmon Configuration Block

Confirm that `ossec.conf` contains:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

After changing `ossec.conf`, restart the Wazuh Agent.

---

# Integration Result

The Sysmon-to-Wazuh integration was successfully completed.

- [x] Sysmon installed on `BTL-WIN11`
- [x] Sysmon Operational log verified
- [x] Wazuh Agent installed
- [x] Wazuh Agent enrolled
- [x] Wazuh Agent status `Active`
- [x] Windows Security log collection configured
- [x] Sysmon Operational log collection configured
- [x] Wazuh Agent restarted successfully
- [x] Test endpoint activity generated
- [x] Sysmon telemetry verified locally
- [x] Sysmon-derived Wazuh alerts observed
- [x] Wazuh alert investigation performed
- [x] Difference between raw telemetry and alerts validated

---

# Skills Practised

This stage of the project provided hands-on experience with:

- Windows Event Logs
- Microsoft Sysmon
- Endpoint telemetry
- Wazuh Agent configuration
- XML configuration
- SIEM log collection
- Detection rules
- Alert generation
- Threat hunting
- Event correlation
- Process investigation
- SOC alert triage
- Troubleshooting security telemetry pipelines

---

# Key Takeaway

One of the most important lessons from this integration was:

> **An event is not an alert, and an alert is not an incident.**

Sysmon provides telemetry.

Wazuh analyses that telemetry using detection rules.

Only activity matching relevant detection logic becomes an alert.

The analyst must then investigate the alert and determine whether the underlying activity is benign or malicious.

---

# Next Step

With endpoint telemetry successfully flowing into the SIEM, the next phase of the project focuses on practical SOC investigations.

Planned investigations include:

1. Windows failed logon analysis
2. Multiple failed authentication attempts
3. PowerShell activity investigation
4. Suspicious process investigation
5. DNS activity investigation
6. Network connection investigation

Each investigation will be documented separately under:

```text
investigations/
```

---

## References

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Wazuh Windows Event Channel Monitoring](https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/configuration.html)
- [Microsoft Sysmon Documentation](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Olaf Hartong — sysmon-modular](https://github.com/olafhartong/sysmon-modular)
