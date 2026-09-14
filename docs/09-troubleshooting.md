# Troubleshooting Guide

This document records the main issues encountered while building the **Blue Team Home SOC Lab**, along with the symptoms, investigation steps, fixes, and lessons learned.

The purpose of this file is to preserve troubleshooting knowledge so the lab can be rebuilt, repaired, or explained later.

---

# 1. Troubleshooting Philosophy

The lab follows a simple troubleshooting model:

```text
Identify Symptom
      │
      ▼
Confirm Scope
      │
      ▼
Check Configuration
      │
      ▼
Check Service State
      │
      ▼
Check Network
      │
      ▼
Check Logs
      │
      ▼
Test Again
      │
      ▼
Document Fix
```

The goal is to avoid random changes.

Each troubleshooting step should answer a specific question.

---

# 2. Issue — Wazuh Installer Script Not Found

## Symptom

Attempting to run the Wazuh installation script resulted in an error similar to:

```text
cannot locate wazuh-install.sh
```

The installer could not run because the script was not present in the current directory.

---

## Diagnosis

Check whether the script exists:

```bash
ls -l
```

or:

```bash
ls -l wazuh-install.sh
```

If the file is missing, the installation cannot continue.

---

## Resolution

Download the installation script explicitly:

```bash
curl -L -o wazuh-install.sh https://packages.wazuh.com/4.14/wazuh-install.sh
```

Verify:

```bash
ls -l wazuh-install.sh
```

Then run:

```bash
sudo bash ./wazuh-install.sh -a
```

---

## Lesson Learned

Before executing an installation script, confirm that the expected file actually exists.

Useful sequence:

```bash
pwd
ls -la
```

This helps confirm both:

- Current working directory
- Available files

---

# 3. Issue — Wazuh System Requirements Warning

## Symptom

During installation, Wazuh displayed a warning indicating that the system did not fully match the recommended requirements.

The VM had:

```text
4 vCPU
8 GB RAM
80 GB Disk
```

---

## Result

The installation continued successfully.

The environment was sufficient for the small home lab workload.

---

## Verification

After installation:

```bash
sudo systemctl status wazuh-manager
```

```bash
sudo systemctl status wazuh-indexer
```

```bash
sudo systemctl status wazuh-dashboard
```

All core services should report a running state.

---

## Lesson Learned

A requirements warning is not automatically an installation failure.

Always verify:

```text
Did installation complete?
        │
        ▼
Are services running?
        │
        ▼
Is the dashboard accessible?
```

before assuming the deployment failed.

---

# 4. Issue — Wazuh Dashboard Certificate Warning

## Symptom

Opening:

```text
https://<WAZUH-SERVER-IP>
```

produced a browser security warning.

---

## Cause

The lab deployment uses a self-signed HTTPS certificate.

The browser does not automatically trust this certificate.

---

## Resolution

For the isolated home lab, the warning was expected.

The dashboard was accessed after confirming that the destination was the correct internal Wazuh server.

---

## Lesson Learned

A browser certificate warning does not necessarily indicate that the application is broken.

However, in a production environment, certificates should be properly trusted and managed.

---

# 5. Issue — Wazuh Agent Initially Stopped

## Symptom

After installing the Wazuh Windows Agent, the endpoint did not immediately appear as active in the Wazuh Dashboard.

The Wazuh service on Windows was stopped.

---

## Diagnosis

Check the service:

```powershell
Get-Service -Name Wazuh
```

The status may show:

```text
Stopped
```

---

## Resolution

Start the service:

```cmd
NET START Wazuh
```

Then verify:

```powershell
Get-Service -Name Wazuh
```

Expected:

```text
Running
```

Refresh the Wazuh Dashboard.

The agent should eventually transition to:

```text
Active
```

---

## Lesson Learned

Installing software does not always mean its service is running.

Always verify service state after deployment.

---

# 6. Issue — Restarting the Wazuh Windows Agent

When configuration changes are made to:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

the Wazuh Agent should be restarted.

Use:

```cmd
NET STOP Wazuh
NET START Wazuh
```

Alternatively:

```powershell
Restart-Service -Name Wazuh
```

Then verify:

```powershell
Get-Service -Name Wazuh
```

---

# 7. Issue — Windows Agent Does Not Become Active

## Symptom

The Wazuh Dashboard shows the Windows endpoint as:

```text
Disconnected
```

or the endpoint never becomes:

```text
Active
```

---

## Step 1 — Check the Windows Service

```powershell
Get-Service -Name Wazuh
```

If stopped:

```cmd
NET START Wazuh
```

---

## Step 2 — Check the Wazuh Agent Log

The log is located at:

```text
C:\Program Files (x86)\ossec-agent\ossec.log
```

View recent entries:

```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 100
```

Monitor live:

```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Wait
```

---

## Step 3 — Test Manager Connectivity

From Windows:

```powershell
Test-NetConnection <WAZUH-SERVER-IP> -Port 1514
```

Expected:

```text
TcpTestSucceeded : True
```

---

## Step 4 — Verify Manager Service

On `BTL-WAZUH`:

```bash
sudo systemctl status wazuh-manager
```

---

## Step 5 — Verify Listening Port

```bash
sudo ss -tulnp | grep 1514
```

---

## Step 6 — Confirm Manager IP

Because the Wazuh server currently uses DHCP, its IP address may change.

Check:

```bash
ip addr
```

If the address changed, the Windows Agent may still be configured to use the old manager IP.

---

# 8. Issue — Wazuh Server IP Address Changed

## Symptom

The dashboard works locally, but agents can no longer connect after a reboot.

---

## Cause

The VMware NAT environment currently assigns IP addresses using DHCP.

The Wazuh server address may therefore change.

---

## Verify Current Address

On `BTL-WAZUH`:

```bash
ip addr
```

Look for the address assigned to:

```text
enp0s20
```

---

## Resolution

Update dependent configurations if necessary.

For long-term stability, a future improvement may be:

- Static IP configuration
- DHCP reservation
- Internal DNS

---

## Lesson Learned

Infrastructure services benefit from predictable addressing.

A SIEM server is more reliable when endpoint agents do not depend on a changing IP address.

---

# 9. Issue — Sysmon Service Verification

## Symptom

Sysmon events are not appearing.

---

## Step 1 — Check Service Status

```powershell
Get-Service -Name Sysmon64
```

Expected:

```text
Running
```

---

## Step 2 — Verify Configuration

Navigate to:

```text
C:\Tools\Sysmon
```

Then:

```powershell
.\Sysmon64a.exe -c
```

This displays the active configuration.

---

## Step 3 — Verify Event Viewer Path

Check:

```text
Applications and Services Logs
→ Microsoft
→ Windows
→ Sysmon
→ Operational
```

---

## Step 4 — Generate Test Activity

```powershell
whoami
hostname
ipconfig
```

Then filter for:

```text
Event ID 1
```

---

# 10. Issue — Sysmon Event Exists Locally but Not in Wazuh Alerts

## Symptom

A command such as:

```powershell
whoami
```

was executed.

Sysmon successfully recorded:

```text
Event ID 1
```

in Event Viewer.

However, searching for the same activity in the Wazuh alerts view returned:

```text
0 hits
```

---

## Initial Concern

This appeared to suggest:

```text
Sysmon works locally
BUT
Wazuh is not receiving events
```

However, that conclusion was incorrect.

---

## Root Cause

The Wazuh view being searched contained:

```text
wazuh-alerts-*
```

This represents events that triggered Wazuh detection rules.

It does not necessarily contain every raw Sysmon event.

---

## Key Concept

```text
Raw Event
    │
    ▼
Wazuh Rule Evaluation
    │
    ├── Match ───▶ Alert
    │
    └── No Match ─▶ No Alert
```

Therefore:

> A Sysmon event can exist without generating a Wazuh alert.

---

## Lesson Learned

This became one of the most important lessons in the lab:

```text
Telemetry ≠ Detection ≠ Alert ≠ Incident
```

---

# 11. Issue — Searching by Sysmon ProcessGuid Returned No Results

## Symptom

A Process GUID was copied from a local Sysmon Event ID 1.

Example format:

```text
{XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX}
```

Searching for it in the Wazuh alerts view returned:

```text
0 hits
```

---

## Explanation

The local Sysmon event existed, but it did not necessarily trigger an alert.

If the search is being performed only against:

```text
wazuh-alerts-*
```

the raw event may not be present.

---

## Lesson Learned

Before concluding that telemetry was lost, determine:

```text
Which dataset am I searching?
```

Questions to ask:

- Raw events?
- Alerts?
- Security events?
- Sysmon events?
- A filtered index?
- A specific time range?

---

# 12. Issue — Wrong Time Range in Wazuh

## Symptom

An expected event cannot be found in the dashboard.

---

## Troubleshooting

Check the dashboard time selector.

The event may exist outside the currently selected range.

For example:

```text
Last 15 minutes
```

may exclude activity generated earlier.

---

## Resolution

Increase the search window.

Examples:

```text
Last 1 hour
Last 24 hours
Custom range
```

---

## Lesson Learned

Always verify the time range before investigating a missing event.

---

# 13. Issue — Event Viewer Shows Activity but Search Filter Hides It

## Symptom

Sysmon is generating events, but they appear to be missing.

---

## Cause

An Event Viewer filter may still be active.

For example, Event Viewer may be displaying only:

```text
Event ID 1
```

while the analyst expects:

```text
Event ID 3
```

---

## Resolution

Check:

```text
Filter Current Log...
```

Clear or modify the active filter.

---

# 14. Issue — Network Test Fails

## Symptom

A system cannot communicate with another VM.

Example:

```text
ping failed
```

---

## Important Note

A failed ping does not automatically mean the target is unreachable.

ICMP may be blocked by the destination firewall.

---

## Better Troubleshooting

Test the specific service.

Example:

```powershell
Test-NetConnection <WAZUH-SERVER-IP> -Port 1514
```

Linux:

```bash
nc -vz <WAZUH-SERVER-IP> 1514
```

This answers:

```text
Can I reach the actual service I need?
```

rather than only testing ICMP.

---

# 15. Issue — Cannot Reach the Internet from a VM

## Step 1 — Check IP Address

Windows:

```cmd
ipconfig
```

Linux:

```bash
ip addr
```

---

## Step 2 — Check Default Route

Linux:

```bash
ip route
```

Windows:

```cmd
route print
```

---

## Step 3 — Test IP Connectivity

```text
1.1.1.1
```

If IP connectivity works but domain names fail, the problem may be DNS.

---

## Step 4 — Test DNS

Windows:

```powershell
Resolve-DnsName example.com
```

Linux:

```bash
getent hosts example.com
```

---

# 16. Issue — VMware NAT Addressing Changed

## Symptom

Previously documented VM IP addresses no longer match current addresses.

---

## Explanation

VMware NAT currently uses DHCP.

Example addresses during the original build were:

```text
BTL-WIN11  → 172.16.106.131
BTL-KALI   → 172.16.106.132
BTL-WAZUH  → 172.16.106.134
```

These are not guaranteed to remain constant.

---

## Resolution

Check the current address on each VM.

Windows:

```cmd
ipconfig
```

Linux:

```bash
ip addr
```

---

## Documentation Practice

Use placeholders where appropriate:

```text
<WAZUH-SERVER-IP>
<WINDOWS-ENDPOINT-IP>
<KALI-IP>
```

This makes the documentation more reusable.

---

# 17. Issue — Kali VMware Integration Problems

## Symptom

Kali may have poor guest integration, display behaviour, or clipboard functionality.

---

## Resolution

Install:

```bash
sudo apt update
sudo apt install open-vm-tools open-vm-tools-desktop -y
```

Then:

```bash
sudo reboot
```

---

## Verify

```bash
systemctl status open-vm-tools
```

---

# 18. Issue — Display Too Small on Retina Screen

## Symptom

Windows or Kali interface elements appear extremely small on the Mac display.

---

## Windows Fix

Navigate to:

```text
Settings
→ System
→ Display
→ Scale
```

Try:

```text
150%
175%
200%
```

---

## Kali Fix

Adjust Xfce display or DPI settings.

The goal is to improve readability without unnecessarily lowering the VM's display quality.

---

# 19. Issue — ARM64 Tool Compatibility

## Symptom

A security tool or virtual appliance cannot be installed or does not run properly.

---

## Cause

The lab runs on:

```text
Apple Silicon / ARM64
```

Some cybersecurity tools are compiled only for:

```text
x86-64 / AMD64
```

---

## Troubleshooting Questions

Before installing a tool, determine:

```text
Is there an ARM64 version?
```

If not:

```text
Can it be compiled from source?
```

If not:

```text
Is there a hosted lab alternative?
```

If not:

```text
Does this task require a separate x86-64 system?
```

---

## Lesson Learned

Do not force an x86-specific tool into an ARM64 lab without first understanding compatibility.

---

# 20. Issue — Wazuh Dashboard Not Loading

## Step 1 — Check Server Address

```bash
ip addr
```

---

## Step 2 — Check Dashboard Service

```bash
sudo systemctl status wazuh-dashboard
```

---

## Step 3 — Check Listening Ports

```bash
sudo ss -tulnp
```

Look for HTTPS.

---

## Step 4 — Test Locally

If available:

```bash
curl -k https://localhost
```

The:

```text
-k
```

option allows the self-signed certificate during testing.

---

## Step 5 — Test from Another System

Windows:

```powershell
Test-NetConnection <WAZUH-SERVER-IP> -Port 443
```

---

# 21. Issue — Wazuh Manager Problems

Check service state:

```bash
sudo systemctl status wazuh-manager
```

View logs:

```bash
sudo journalctl -u wazuh-manager -n 100
```

Follow live:

```bash
sudo journalctl -u wazuh-manager -f
```

Check Wazuh log:

```bash
sudo tail -n 100 /var/ossec/logs/ossec.log
```

---

# 22. Issue — Wazuh Indexer Problems

Check:

```bash
sudo systemctl status wazuh-indexer
```

View recent logs:

```bash
sudo journalctl -u wazuh-indexer -n 100
```

Check resource usage:

```bash
free -h
```

```bash
df -h
```

The indexer can be affected by resource constraints.

---

# 23. Issue — Disk Space

## Symptom

Wazuh becomes unstable or stops indexing correctly.

---

## Check Disk Usage

```bash
df -h
```

The indexer stores security data, so disk consumption increases as telemetry is generated.

---

## Lesson Learned

SIEM platforms require ongoing storage monitoring.

The more telemetry collected, the greater the storage requirement.

---

# 24. Issue — High Memory Usage

Check:

```bash
free -h
```

Then:

```bash
top
```

or:

```bash
htop
```

Wazuh consists of several components and can consume significant memory compared with a normal Ubuntu Server.

---

# 25. Issue — `ossec.conf` Changes Not Taking Effect

## Symptom

A new event channel was added, but expected telemetry is not appearing.

---

## Step 1 — Verify Configuration

Check:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Example Sysmon block:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

---

## Step 2 — Save the File

Make sure the edited file was actually saved.

Administrative privileges may be required.

---

## Step 3 — Restart the Agent

```cmd
NET STOP Wazuh
NET START Wazuh
```

---

## Step 4 — Inspect Agent Logs

```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 100
```

---

# 26. Issue — Cannot Save `ossec.conf`

## Symptom

Notepad refuses to save changes.

---

## Cause

The file is located under:

```text
C:\Program Files (x86)\
```

which requires elevated permissions.

---

## Resolution

Launch Notepad as Administrator:

```powershell
notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"
```

Then edit and save.

---

# 27. Issue — Confusing Alert Severity with Maliciousness

## Symptom

A high-severity Wazuh alert appears and is immediately assumed to be malicious.

---

## Example

The lab generated:

```text
Rule ID: 92910
Level: 12
```

Description:

```text
Explorer process was accessed by OneDrive.exe,
possible process injection
```

---

## Investigation

The alert showed:

```text
Source:
OneDrive.exe

Target:
Explorer.EXE
```

The call trace contained legitimate OneDrive modules such as:

```text
FileSyncClient.dll
FileSyncEvents.dll
FileSyncHost.DLL
```

The user context also matched normal endpoint activity.

---

## Verdict

```text
Likely Benign
```

---

## Lesson Learned

```text
High Severity
     ≠
Confirmed Malicious
```

Alert severity indicates detection significance, not final analyst verdict.

---

# 28. Issue — Treating an Alert as an Incident

An alert means:

```text
Detection logic matched activity.
```

It does not mean:

```text
A confirmed compromise occurred.
```

The correct workflow is:

```text
Alert
  │
  ▼
Triage
  │
  ▼
Investigation
  │
  ▼
Evidence
  │
  ▼
Verdict
```

Possible outcomes:

```text
Benign
False Positive
Benign Positive
Suspicious
Malicious
Confirmed Incident
```

---

# 29. Issue — One Event Is Not Enough Context

An individual event may look suspicious without surrounding context.

For example:

```text
powershell.exe
```

by itself does not prove malicious activity.

Investigate:

```text
Parent Process
Command Line
User
Integrity Level
Files Created
DNS Queries
Network Connections
Process Access
Timeline
```

---

# 30. Troubleshooting Investigation Workflow

When an alert appears, use:

```text
1. Identify the alert
2. Identify the host
3. Identify the user
4. Identify the source process
5. Identify the target
6. Check command line
7. Check parent process
8. Check network activity
9. Check DNS activity
10. Check file activity
11. Check surrounding timeline
12. Decide verdict
```

---

# 31. Useful Windows Troubleshooting Commands

## Hostname

```powershell
hostname
```

## Current User

```powershell
whoami
```

## Network Configuration

```powershell
ipconfig /all
```

## Services

```powershell
Get-Service
```

## Wazuh Service

```powershell
Get-Service -Name Wazuh
```

## Sysmon Service

```powershell
Get-Service -Name Sysmon64
```

## DNS

```powershell
Resolve-DnsName example.com
```

## TCP Connectivity

```powershell
Test-NetConnection <IP> -Port <PORT>
```

---

# 32. Useful Linux Troubleshooting Commands

## IP Address

```bash
ip addr
```

## Routing

```bash
ip route
```

## DNS

```bash
getent hosts example.com
```

## Listening Ports

```bash
sudo ss -tulnp
```

## Service Status

```bash
sudo systemctl status <service>
```

## Logs

```bash
sudo journalctl -u <service>
```

## Disk

```bash
df -h
```

## Memory

```bash
free -h
```

---

# 33. Troubleshooting Decision Tree

```text
Something Is Not Working
          │
          ▼
Is the VM powered on?
          │
          ▼
Does it have an IP?
          │
          ▼
Is the required service running?
          │
          ▼
Is the required port listening?
          │
          ▼
Can the other VM reach that port?
          │
          ▼
Are logs showing errors?
          │
          ▼
Is the configuration correct?
          │
          ▼
Restart only the required service
          │
          ▼
Test again
```

---

# 34. Avoid Random Troubleshooting

Avoid repeatedly changing settings without understanding the problem.

Bad workflow:

```text
Problem
  │
  ▼
Change random setting
  │
  ▼
Change another setting
  │
  ▼
Restart everything
  │
  ▼
Unknown result
```

Better workflow:

```text
Problem
  │
  ▼
Form Hypothesis
  │
  ▼
Run One Test
  │
  ▼
Observe Result
  │
  ▼
Confirm / Reject Hypothesis
  │
  ▼
Make Targeted Change
```

---

# 35. Preserve Working States

Before major configuration changes:

```text
Working State
     │
     ▼
Create Snapshot
     │
     ▼
Make Change
     │
     ▼
Test
```

If the change breaks the environment:

```text
Restore Snapshot
```

This is especially useful in a home lab.

---

# 36. Do Not Rebuild Too Quickly

If something stops working, rebuilding the entire VM should not be the first response.

Troubleshooting provides valuable learning.

Investigate:

- Logs
- Services
- Networking
- Configuration
- Permissions
- Dependencies

Rebuild only when it is actually the most efficient recovery method.

---

# 37. Key Troubleshooting Lessons

The main lessons from the lab so far are:

```text
1. Verify files before executing them.
2. Verify services after installation.
3. Verify network connectivity by port, not only ping.
4. Check logs before changing configuration.
5. DHCP addresses can change.
6. ARM64 compatibility matters.
7. A raw event may not become an alert.
8. An alert is not automatically malicious.
9. High severity is not a final verdict.
10. Time ranges and filters can hide evidence.
11. Context matters more than a single event.
12. Document every useful fix.
```

---

# 38. Lab Troubleshooting Checklist

When something fails:

- [ ] Is the VM powered on?
- [ ] Does the VM have the expected IP address?
- [ ] Is the network adapter connected?
- [ ] Is the required service running?
- [ ] Is the required port listening?
- [ ] Can the source reach the destination port?
- [ ] Are there errors in local logs?
- [ ] Is the configuration file valid?
- [ ] Was the service restarted after configuration changes?
- [ ] Is the dashboard time range correct?
- [ ] Are search filters hiding the event?
- [ ] Am I searching raw telemetry or alerts?
- [ ] Did the event actually trigger a detection rule?
- [ ] Did the VM IP change?
- [ ] Is the tool compatible with ARM64?
- [ ] Is there a known-good snapshot available?

---

# 39. Key Takeaway

Troubleshooting is part of security analysis.

The same mindset used to repair the lab also applies to SOC investigations:

```text
Observe
   │
   ▼
Form Hypothesis
   │
   ▼
Collect Evidence
   │
   ▼
Test
   │
   ▼
Correlate
   │
   ▼
Reach Conclusion
```

The goal is not simply to make the system work.

The goal is to understand **why it failed and why the fix worked**.

---

# 40. Related Documentation

Architecture:

```text
docs/01-architecture.md
```

VMware configuration:

```text
docs/02-vmware-fusion-setup.md
```

Windows endpoint:

```text
docs/03-windows-11-setup.md
```

Sysmon:

```text
docs/04-sysmon-setup.md
```

Kali:

```text
docs/05-kali-linux-setup.md
```

Wazuh server:

```text
docs/06-wazuh-server-setup.md
```

Wazuh Windows Agent:

```text
docs/07-wazuh-windows-agent.md
```

Sysmon to Wazuh:

```text
docs/08-sysmon-to-wazuh.md
```

Snapshot and recovery strategy:

```text
docs/10-snapshots-and-recovery.md
```

---

## References

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Microsoft Sysmon Documentation](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Microsoft PowerShell Documentation](https://learn.microsoft.com/powershell/)
- [Kali Linux Documentation](https://www.kali.org/docs/)
- [Ubuntu Server Documentation](https://ubuntu.com/server/docs)
