# Wazuh Windows Agent Deployment (`BTL-WIN11`)

This document outlines the installation, enrollment, service management, and verification of the **Wazuh Windows Agent** deployed on the `BTL-WIN11` endpoint.

---

## Overview

The Wazuh Agent was installed on `BTL-WIN11` to forward Windows security telemetry to the central `BTL-WAZUH` server.

The endpoint configuration consists of:

- **Endpoint:** `BTL-WIN11`
- **Operating System:** Windows 11 Pro ARM64
- **Wazuh Agent:** 4.14.7
- **Wazuh Manager:** `BTL-WAZUH`
- **Agent Group:** `default`
- **Communication:** VMware NAT network

> **Note:** IP addresses are represented using placeholders throughout this document. Replace `<WAZUH-SERVER-IP>` with the IP address of the Wazuh Manager when reproducing the deployment.

---

# Agent Installation & Enrollment

## 1. Open an Elevated PowerShell Session

Open **PowerShell as Administrator** using the `labadmin` account.

---

## 2. Download the Wazuh Windows Agent

Download the official Wazuh Windows Agent MSI package:

```powershell
Invoke-WebRequest `
  -Uri "https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.7-1.msi" `
  -OutFile "$env:TEMP\wazuh-agent.msi"
```

This downloads the installer to:

```text
%TEMP%\wazuh-agent.msi
```

---

## 3. Install and Enroll the Agent

Run the installer silently and configure the Wazuh Manager address and endpoint name:

```powershell
msiexec.exe /i "$env:TEMP\wazuh-agent.msi" /q `
  WAZUH_MANAGER='<WAZUH-SERVER-IP>' `
  WAZUH_AGENT_NAME='BTL-WIN11'
```

Replace:

```text
<WAZUH-SERVER-IP>
```

with the IP address assigned to the `BTL-WAZUH` server.

For example:

```powershell
msiexec.exe /i "$env:TEMP\wazuh-agent.msi" /q `
  WAZUH_MANAGER='192.168.100.10' `
  WAZUH_AGENT_NAME='BTL-WIN11'
```

> **Security Note:** The IP address above is an example only. Credentials, passwords, API keys, and other sensitive information should never be committed to a public GitHub repository.

---

# Service Management

The Windows service installed by the Wazuh Agent is named:

```text
Wazuh
```

The following commands can be executed from an elevated **Command Prompt** or **PowerShell** session.

## Start the Wazuh Agent

```cmd
NET START Wazuh
```

---

## Stop the Wazuh Agent

```cmd
NET STOP Wazuh
```

---

## Restart the Wazuh Agent

```cmd
NET STOP Wazuh
NET START Wazuh
```

---

## Check Service Status

The service can also be inspected using PowerShell:

```powershell
Get-Service -Name Wazuh
```

A successfully running agent should show:

```text
Status   Name    DisplayName
------   ----    -----------
Running  Wazuh   Wazuh
```

---

# Agent Log Location

The Wazuh Agent maintains a local log file at:

```text
C:\Program Files (x86)\ossec-agent\ossec.log
```

This log is useful when troubleshooting:

- Agent enrollment
- Manager connectivity
- Configuration errors
- Event collection
- Connection failures
- Agent startup issues

---

# Verify Manager Connectivity

Open the agent log:

```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 50
```

To continuously monitor new log entries:

```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Wait
```

Look for messages indicating that the agent successfully connected to the Wazuh Manager.

A successful connection may contain information similar to:

```text
Connected to server (<WAZUH-SERVER-IP>:1514)
```

---

# Verify the Agent in Wazuh Dashboard

After starting the Wazuh Agent, open the Wazuh Dashboard and verify that the endpoint appears in the agent list.

The endpoint should eventually display:

```text
Agent Name: BTL-WIN11
Status: Active
```

During the initial enrollment process, the agent may temporarily appear as:

```text
Pending
```

After successful communication with the Wazuh Manager, the status should change to:

```text
Active
```

---

# Communication Flow

The basic communication path is:

```text
BTL-WIN11
    │
    │ Wazuh Agent
    │
    ▼
TCP 1514
    │
    ▼
BTL-WAZUH
    │
    ├── Wazuh Manager
    ├── Wazuh Indexer
    └── Wazuh Dashboard
```

The Windows endpoint collects local telemetry and forwards relevant information to the Wazuh infrastructure for analysis and alert generation.

---

# Troubleshooting

## Agent Service Is Stopped

During the initial deployment, the Wazuh Agent service was found in a stopped state.

The service was started manually:

```cmd
NET START Wazuh
```

After starting the service and refreshing the Wazuh Dashboard, the endpoint successfully transitioned to:

```text
Active
```

---

## Check the Agent Log

If the endpoint does not become active, inspect:

```text
C:\Program Files (x86)\ossec-agent\ossec.log
```

Using:

```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 100
```

Look for errors relating to:

- Connection failures
- Enrollment failures
- Invalid manager address
- Authentication problems
- Configuration errors

---

## Check Network Connectivity

From `BTL-WIN11`, test connectivity to the Wazuh Manager:

```powershell
Test-NetConnection <WAZUH-SERVER-IP> -Port 1514
```

For example:

```powershell
Test-NetConnection 192.168.100.10 -Port 1514
```

A successful test should show:

```text
TcpTestSucceeded : True
```

---

# Deployment Result

The deployment was successfully completed and verified.

- [x] Wazuh Windows Agent downloaded
- [x] Agent installed on `BTL-WIN11`
- [x] Wazuh Manager address configured
- [x] Agent name configured as `BTL-WIN11`
- [x] Wazuh Agent service started
- [x] Agent successfully enrolled
- [x] Agent appeared in Wazuh Dashboard
- [x] Agent status changed to `Active`
- [x] Manager connectivity verified

---

# Next Step

After successfully enrolling the Windows endpoint, the next stage of the lab is to configure the Wazuh Agent to collect **Sysmon telemetry** from:

```text
Microsoft-Windows-Sysmon/Operational
```

This allows detailed endpoint activity such as process creation, network connections, DNS queries, file creation, and process access to be analysed by Wazuh.

The configuration is documented in:

```text
docs/08-sysmon-to-wazuh.md
```

---

## References

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Wazuh Windows Agent Installation](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html)
- [Wazuh Agent Enrollment](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/index.html)
