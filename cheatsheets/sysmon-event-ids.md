# Sysmon Event ID Cheat Sheet

This cheat sheet focuses on the Sysmon events most useful for Blue Team, SOC, threat hunting, and incident investigation.

The goal is not to memorise every Sysmon Event ID.

The goal is to understand:

```text
What happened?
Which Sysmon event records it?
What fields matter?
What should I pivot to next?
```

---

# 1. What Is Sysmon?

Sysmon, or **System Monitor**, is a Microsoft Sysinternals tool that provides detailed Windows telemetry.

It can record activity such as:

- Process creation
- Network connections
- File creation
- DNS queries
- Registry changes
- Driver loading
- Process access
- Named pipes
- WMI activity
- Process termination

Sysmon does not automatically decide whether activity is malicious.

It provides telemetry.

The analyst or detection engine provides the interpretation.

---

# 2. Sysmon Log Location

Sysmon events are stored in:

```text
Applications and Services Logs
    ↓
Microsoft
    ↓
Windows
    ↓
Sysmon
    ↓
Operational
```

Full Event Log channel:

```text
Microsoft-Windows-Sysmon/Operational
```

---

# 3. PowerShell Query

View recent Sysmon events:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 20
```

Filter by Event ID:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 1
} -MaxEvents 20
```

---

# 4. Most Important Sysmon Events

For junior SOC / BTL1-style work, prioritise:

| Event ID | Event | Priority |
|---|---|---|
| `1` | Process Creation | Critical |
| `3` | Network Connection | Critical |
| `7` | Image Loaded | High |
| `10` | Process Access | High |
| `11` | File Create | Critical |
| `22` | DNS Query | Critical |

Also useful:

| Event ID | Event |
|---|---|
| `2` | File creation time changed |
| `4` | Sysmon service state changed |
| `5` | Process terminated |
| `6` | Driver loaded |
| `8` | CreateRemoteThread |
| `9` | RawAccessRead |
| `12` | Registry object create/delete |
| `13` | Registry value set |
| `14` | Registry object renamed |
| `15` | FileCreateStreamHash |
| `17` | Pipe created |
| `18` | Pipe connected |
| `19` | WMI filter |
| `20` | WMI consumer |
| `21` | WMI consumer to filter |
| `23` | File delete archived |
| `25` | Process tampering |
| `26` | File delete logged |
| `27` | File blocked executable |
| `28` | File blocked shredding |
| `29` | File executable detected |

---

# 5. Event ID 1 — Process Creation

## What It Records

```text
A new process started.
```

Example:

```text
explorer.exe
     ↓
powershell.exe
```

This is one of the most important Sysmon events.

---

## Important Fields

Look for:

```text
UtcTime
ProcessGuid
ProcessId
Image
FileVersion
Description
Product
Company
OriginalFileName
CommandLine
CurrentDirectory
User
LogonGuid
LogonId
TerminalSessionId
IntegrityLevel
Hashes
ParentProcessGuid
ParentProcessId
ParentImage
ParentCommandLine
ParentUser
```

---

## Key Analyst Questions

```text
What executed?

Where did it execute from?

Which user ran it?

What command line was used?

What process launched it?

Is the parent-child relationship expected?

Is the path suspicious?

Is the filename masquerading as something legitimate?
```

---

## Example

```text
Image:
C:\Windows\System32\whoami.exe

CommandLine:
whoami

ParentImage:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

---

## Suspicious Examples

```text
winword.exe
    ↓
powershell.exe
```

```text
outlook.exe
    ↓
cmd.exe
```

```text
explorer.exe
    ↓
C:\Users\User\AppData\Local\Temp\update.exe
```

These are not automatically malicious, but they deserve investigation.

---

## PowerShell Query

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 1
} -MaxEvents 20
```

---

## Best Pivots

From Event ID 1, pivot to:

```text
Event 3  → Network Connection
Event 10 → Process Access
Event 11 → File Create
Event 22 → DNS Query
```

Use:

```text
ProcessGuid
```

where possible.

---

# 6. Event ID 2 — File Creation Time Changed

## What It Records

```text
A process changed a file's creation timestamp.
```

This may be associated with:

```text
Timestamp manipulation
Anti-forensics
Timestomping
```

---

## Important Fields

```text
ProcessGuid
ProcessId
Image
TargetFilename
CreationUtcTime
PreviousCreationUtcTime
User
```

---

## Analyst Questions

```text
Why did the timestamp change?

Which process changed it?

Was the file recently created?

Is this software known to modify timestamps legitimately?
```

---

# 7. Event ID 3 — Network Connection

## What It Records

```text
A process established a network connection.
```

This is extremely useful for correlating process activity with network behaviour.

---

## Important Fields

```text
UtcTime
ProcessGuid
ProcessId
Image
User
Protocol
Initiated
SourceIsIpv6
SourceIp
SourceHostname
SourcePort
SourcePortName
DestinationIsIpv6
DestinationIp
DestinationHostname
DestinationPort
DestinationPortName
```

---

## Example

```text
Image:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

DestinationIp:
93.184.216.34

DestinationPort:
443

Protocol:
tcp
```

---

## Questions

```text
Which process made the connection?

Where did it connect?

Was the connection outbound?

Which protocol was used?

Which destination port?

Did DNS resolution happen immediately before this?
```

---

## PowerShell Query

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 3
} -MaxEvents 20
```

---

## Strong Pivot

If you have a suspicious process:

```text
Event 1
   ↓
ProcessGuid
   ↓
Event 3
```

This can show:

```text
Process created
      ↓
Network connection
```

---

# 8. Event ID 4 — Sysmon Service State Changed

## What It Records

```text
The Sysmon service started or stopped.
```

Useful for:

- Monitoring Sysmon status
- Detecting unexpected service changes
- Troubleshooting

Unexpected stopping of Sysmon may be interesting during an investigation.

---

# 9. Event ID 5 — Process Terminated

## What It Records

```text
A process exited.
```

Important fields include:

```text
ProcessGuid
ProcessId
Image
User
UtcTime
```

Useful for process lifetime analysis.

---

## Example Timeline

```text
10:00:00 Event 1 — powershell.exe created
10:00:15 Event 3 — outbound connection
10:00:45 Event 5 — powershell.exe terminated
```

---

# 10. Event ID 6 — Driver Loaded

## What It Records

```text
A kernel driver loaded.
```

Drivers operate with high privileges and can be important in security investigations.

---

## Important Fields

```text
ImageLoaded
Hashes
Signed
Signature
SignatureStatus
```

---

## Questions

```text
Is the driver signed?

Who signed it?

Where is it located?

Is the driver expected?

Is the hash known?
```

---

# 11. Event ID 7 — Image Loaded

## What It Records

```text
A DLL or executable image was loaded into a process.
```

This can help identify:

- Suspicious DLL loading
- DLL side-loading
- Unexpected modules
- Malicious libraries

---

## Important Fields

```text
ProcessGuid
ProcessId
Image
ImageLoaded
FileVersion
Description
Product
Company
OriginalFileName
Hashes
Signed
Signature
SignatureStatus
User
```

---

## Example

```text
Process:
C:\Windows\System32\notepad.exe

ImageLoaded:
C:\Users\User\AppData\Local\Temp\example.dll
```

That would deserve investigation.

---

## Questions

```text
Which process loaded the DLL?

Where is the DLL located?

Is it signed?

Does the location make sense?

Is the company / product metadata consistent?
```

---

# 12. Event ID 8 — CreateRemoteThread

## What It Records

```text
A process created a thread inside another process.
```

This can be associated with process injection.

---

## Important Fields

```text
SourceProcessGuid
SourceProcessId
SourceImage
TargetProcessGuid
TargetProcessId
TargetImage
NewThreadId
StartAddress
StartModule
StartFunction
SourceUser
TargetUser
```

---

## Why It Matters

Conceptually:

```text
Process A
   ↓
Creates Thread
   ↓
Inside Process B
```

This can be legitimate but is also a common injection technique.

---

# 13. Event ID 9 — RawAccessRead

## What It Records

```text
A process read directly from a disk or volume.
```

This may be relevant to:

- Forensic tools
- Backup software
- Credential theft
- Raw disk access

Context is essential.

---

# 14. Event ID 10 — Process Access

## What It Records

```text
One process opened another process.
```

This was central to the lab's first investigation.

Example:

```text
OneDrive.exe
     ↓
Explorer.EXE
```

---

## Important Fields

```text
UtcTime
SourceProcessGuid
SourceProcessId
SourceThreadId
SourceImage
TargetProcessGuid
TargetProcessId
TargetImage
GrantedAccess
CallTrace
SourceUser
TargetUser
```

---

## Questions

```text
Which process accessed which process?

What access rights were requested?

Are both processes expected?

Are the source and target users expected?

What modules appear in CallTrace?

Is there other evidence of injection?
```

---

## Lab Example

```text
SourceImage:
C:\Users\labadmin\AppData\Local\Microsoft\OneDrive\OneDrive.exe

TargetImage:
C:\WINDOWS\Explorer.EXE

GrantedAccess:
0x101411
```

The alert looked severe but was ultimately assessed as likely benign.

---

## Important Lesson

```text
Process Access
      ≠
Process Injection
```

Event ID 10 is telemetry, not proof of malicious activity.

---

# 15. Event ID 11 — File Create

## What It Records

```text
A file was created or overwritten.
```

This is extremely useful.

---

## Important Fields

```text
UtcTime
ProcessGuid
ProcessId
Image
TargetFilename
CreationUtcTime
User
```

---

## Examples

```text
powershell.exe
    ↓
C:\Users\User\AppData\Local\Temp\payload.exe
```

or:

```text
browser.exe
    ↓
C:\Users\User\Downloads\invoice.zip
```

---

## Questions

```text
Which process created the file?

Where was the file written?

Is that directory user-writable?

What extension does it have?

Was the file later executed?

What is the file hash?
```

---

## PowerShell Query

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 11
} -MaxEvents 20
```

---

## Strong Pivot

```text
Event 11 — File Created
       ↓
Event 1 — File Executed?
```

or:

```text
Event 1 — Process
       ↓
Event 11 — What did it create?
```

---

# 16. Event ID 12 — Registry Object Create/Delete

## What It Records

```text
A registry key or value was created or deleted.
```

Useful for detecting:

- Persistence
- Configuration changes
- Security setting changes

---

## Important Fields

```text
EventType
UtcTime
ProcessGuid
ProcessId
Image
TargetObject
User
```

---

# 17. Event ID 13 — Registry Value Set

## What It Records

```text
A registry value was written or modified.
```

This is particularly useful for persistence investigations.

---

## Example Areas

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

```text
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
```

---

## Important Fields

```text
EventType
ProcessGuid
ProcessId
Image
TargetObject
Details
User
```

---

# 18. Event ID 14 — Registry Object Renamed

## What It Records

```text
A registry key or value was renamed.
```

Less common, but potentially useful during registry-based persistence or evasion investigations.

---

# 19. Event ID 15 — FileCreateStreamHash

## What It Records

```text
A named file stream was created and hashed.
```

This can help detect:

```text
NTFS Alternate Data Streams
```

ADS can be used legitimately but may also be abused for hiding data.

---

## Example

```text
file.txt:hidden.exe
```

---

# 20. Event ID 17 — Pipe Created

## What It Records

```text
A named pipe was created.
```

Named pipes are used for inter-process communication.

They may appear in:

- Normal Windows operations
- Remote administration
- Malware
- Lateral movement

---

# 21. Event ID 18 — Pipe Connected

## What It Records

```text
A process connected to a named pipe.
```

Correlate Event 17 and 18 when investigating named-pipe activity.

---

# 22. Event ID 19 — WMI Filter

## What It Records

```text
A permanent WMI event filter was registered.
```

WMI can be used for:

- Administration
- Automation
- Persistence

---

# 23. Event ID 20 — WMI Consumer

## What It Records

```text
A permanent WMI event consumer was registered.
```

---

# 24. Event ID 21 — WMI Consumer to Filter

## What It Records

```text
A WMI consumer was bound to a filter.
```

Together:

```text
Event 19
+
Event 20
+
Event 21
```

may indicate WMI-based persistence.

---

# 25. Event ID 22 — DNS Query

## What It Records

```text
A process made a DNS query.
```

This is one of the most useful Sysmon events for SOC work.

---

## Important Fields

```text
UtcTime
ProcessGuid
ProcessId
QueryName
QueryStatus
QueryResults
Image
User
```

---

## Example

```text
Image:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

QueryName:
example.com
```

---

## Questions

```text
Which process queried the domain?

Was the domain expected?

What happened immediately afterward?

Did the process connect to the resolved IP?

Is the domain suspicious?
```

---

## PowerShell Query

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 22
} -MaxEvents 20
```

---

## Strong Pivot

```text
Event 22 — DNS Query
       ↓
Event 3 — Network Connection
```

Look for:

```text
Same ProcessGuid
Same ProcessId
Close timestamp
Matching destination
```

---

# 26. Event ID 23 — File Delete Archived

## What It Records

```text
A file was deleted and archived by Sysmon.
```

This depends on configuration.

It can preserve deleted files for investigation.

---

# 27. Event ID 25 — Process Tampering

## What It Records

```text
Process image manipulation or tampering detected by Sysmon.
```

Potential examples include suspicious manipulation techniques.

This event deserves significant attention.

---

# 28. Event ID 26 — File Delete Logged

## What It Records

```text
A file deletion was logged without archiving the file.
```

Useful for:

- Cleanup activity
- Evidence destruction
- Temporary payloads
- Normal application behaviour

Again:

```text
Deletion ≠ Malicious
```

---

# 29. Event ID 27 — File Block Executable

## What It Records

```text
Sysmon blocked creation of an executable file based on configured rules.
```

Only relevant when Sysmon's file blocking features are configured.

---

# 30. Event ID 28 — File Block Shredding

## What It Records

```text
Sysmon detected and blocked file shredding behaviour under configured rules.
```

Configuration dependent.

---

# 31. Event ID 29 — File Executable Detected

## What It Records

```text
Creation of an executable file was detected.
```

This can provide useful context around dropped executables.

---

# 32. ProcessGuid — One of the Most Important Fields

Sysmon assigns a:

```text
ProcessGuid
```

to processes.

This is useful because PIDs can be reused.

Example:

```text
ProcessGuid:
{480d0770-c381-6aa7-3e03-000000000c00}
```

Use it to correlate activity.

---

# 33. Why ProcessGuid Matters

A single process may generate:

```text
Event 1  — Process Creation
Event 3  — Network Connection
Event 10 — Process Access
Event 11 — File Create
Event 22 — DNS Query
```

Conceptually:

```text
                ProcessGuid
                    │
       ┌────────────┼────────────┐
       │            │            │
       ▼            ▼            ▼
   DNS Query    File Create   Connection
   Event 22      Event 11      Event 3
```

This allows analysts to reconstruct behaviour.

---

# 34. PID vs ProcessGuid

## PID

```text
ProcessId
```

Useful but may be reused over time.

## ProcessGuid

```text
ProcessGuid
```

More reliable for correlating Sysmon activity belonging to the same process instance.

---

# 35. High-Value Investigation Chain

One of the best patterns to learn:

```text
Event 1
Process Created
      ↓
Event 22
DNS Query
      ↓
Event 3
Network Connection
      ↓
Event 11
File Created
```

This might reveal:

```text
Process executes
      ↓
Resolves domain
      ↓
Connects outbound
      ↓
Drops file
```

---

# 36. Authentication + Sysmon Correlation

Sysmon does not replace Windows Security logs.

Combine:

```text
Windows Event 4624
Successful Login
      ↓
Sysmon Event 1
Process Creation
      ↓
Sysmon Event 22
DNS Query
      ↓
Sysmon Event 3
Network Connection
```

This creates a richer timeline.

---

# 37. Sysmon and Windows Security Logs

Think of them as complementary.

Windows Security:

```text
Who authenticated?
Which account?
What logon type?
```

Sysmon:

```text
What process ran?
What file was created?
What network connection happened?
What DNS query occurred?
```

Together:

```text
Identity + Endpoint Behaviour
```

---

# 38. Sysmon and Wazuh

Current lab architecture:

```text
Windows Activity
      ↓
Sysmon
      ↓
Windows Event Channel
      ↓
Wazuh Agent
      ↓
Wazuh Manager
      ↓
Detection Rules
      ↓
Alerts
```

Important lesson:

```text
Sysmon Event
      ≠
Wazuh Alert
```

Not every Sysmon event will trigger an alert.

---

# 39. Raw Telemetry vs Alert

Example:

```text
whoami.exe
```

may generate:

```text
Sysmon Event ID 1
```

but Wazuh may not generate an alert because the behaviour itself is not suspicious enough to match a detection rule.

This is normal.

---

# 40. Common Fields to Always Check

For process-related events:

```text
UtcTime
ProcessGuid
ProcessId
Image
CommandLine
User
ParentImage
ParentCommandLine
```

For network events:

```text
Image
ProcessGuid
SourceIp
SourcePort
DestinationIp
DestinationPort
Protocol
```

For DNS:

```text
Image
ProcessGuid
QueryName
QueryResults
```

For files:

```text
Image
ProcessGuid
TargetFilename
CreationUtcTime
```

---

# 41. Suspicious Paths

Pay special attention to executables running from user-writable locations such as:

```text
C:\Users\<user>\AppData\
C:\Users\<user>\Downloads\
C:\Users\<user>\Desktop\
C:\Users\<user>\AppData\Local\Temp\
C:\Windows\Temp\
C:\Users\Public\
```

But:

> A user-writable location does not automatically mean malicious.

Many legitimate applications use these paths.

---

# 42. Suspicious Parent-Child Relationships

Examples worth investigating:

```text
winword.exe
    ↓
powershell.exe
```

```text
excel.exe
    ↓
cmd.exe
```

```text
outlook.exe
    ↓
wscript.exe
```

```text
browser.exe
    ↓
powershell.exe
```

```text
services.exe
    ↓
unexpected-user-binary.exe
```

Again, investigate context.

---

# 43. Suspicious Command-Line Indicators

Look for patterns such as:

```text
-EncodedCommand
-ExecutionPolicy Bypass
-WindowStyle Hidden
Invoke-Expression
IEX
DownloadString
FromBase64String
certutil
bitsadmin
rundll32
regsvr32
mshta
```

These are pivots, not automatic proof of maliciousness.

---

# 44. Event ID Priority for Beginners

Learn in this order:

```text
1
↓
22
↓
3
↓
11
↓
10
↓
7
```

Meaning:

```text
Process
↓
DNS
↓
Network
↓
File
↓
Process Access
↓
DLL Loading
```

This covers a large amount of practical SOC investigation.

---

# 45. Quick Reference Table

| ID | Meaning | Key Question |
|---|---|---|
| `1` | Process Creation | What executed? |
| `2` | File Creation Time Changed | Was a timestamp altered? |
| `3` | Network Connection | Where did the process connect? |
| `4` | Sysmon State Changed | Was Sysmon started/stopped? |
| `5` | Process Terminated | When did the process end? |
| `6` | Driver Loaded | What driver loaded? |
| `7` | Image Loaded | What DLL/module loaded? |
| `8` | CreateRemoteThread | Did one process create a thread in another? |
| `9` | RawAccessRead | Did a process access raw disk? |
| `10` | Process Access | Which process accessed another? |
| `11` | File Create | What file was created? |
| `12` | Registry Create/Delete | What registry object changed? |
| `13` | Registry Value Set | What registry value was modified? |
| `14` | Registry Rename | What registry object was renamed? |
| `15` | File Stream Hash | Was an alternate stream created? |
| `17` | Pipe Created | What named pipe was created? |
| `18` | Pipe Connected | Who connected to the pipe? |
| `19` | WMI Filter | Was WMI persistence configured? |
| `20` | WMI Consumer | Was a WMI consumer created? |
| `21` | WMI Binding | Was consumer bound to filter? |
| `22` | DNS Query | What domain was queried? |
| `23` | File Delete Archived | What file was deleted? |
| `25` | Process Tampering | Was process tampering detected? |
| `26` | File Delete | What file was deleted? |
| `27` | File Block Executable | Was executable creation blocked? |
| `28` | File Block Shredding | Was shredding blocked? |
| `29` | Executable Detected | Was an executable file created? |

---

# 46. Quick PowerShell Queries

## Process Creation

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 1
} -MaxEvents 20
```

## Network Connections

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 3
} -MaxEvents 20
```

## Process Access

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 10
} -MaxEvents 20
```

## File Creation

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 11
} -MaxEvents 20
```

## DNS Queries

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 22
} -MaxEvents 20
```

---

# 47. Search by Process Name

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 1
} |
Where-Object {
    $_.Message -match "powershell.exe"
}
```

---

# 48. Search by ProcessGuid

```powershell
$guid = '{PROCESS-GUID}'

Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" |
Where-Object {
    $_.Message -match [regex]::Escape($guid)
}
```

Use this to correlate multiple event types.

---

# 49. Investigation Workflow

When a Sysmon event looks suspicious:

```text
Identify Event ID
      ↓
Understand What It Represents
      ↓
Check Timestamp
      ↓
Identify Process
      ↓
Identify User
      ↓
Check Parent Process
      ↓
Review Command Line
      ↓
Collect ProcessGuid
      ↓
Search Related Sysmon Events
      ↓
Check Windows Security Logs
      ↓
Build Timeline
      ↓
Reach Verdict
```

---

# 50. Example Investigation Chain

Suppose you see:

```text
Event 1
powershell.exe
```

Do not stop there.

Investigate:

```text
Who launched it?
      ↓
What command line?
      ↓
Which user?
      ↓
What ProcessGuid?
      ↓
Did it query DNS?
      ↓
Did it connect out?
      ↓
Did it create files?
      ↓
Did it access another process?
```

Search:

```text
Event 22
Event 3
Event 11
Event 10
```

---

# 51. Analyst Mistakes to Avoid

Do not assume:

```text
PowerShell = malicious
```

Do not assume:

```text
High alert severity = compromise
```

Do not assume:

```text
Process access = process injection
```

Do not assume:

```text
Unknown domain = malicious
```

Do not investigate a single event in isolation if correlated evidence is available.

---

# 52. Sysmon Does Not Replace Investigation

Sysmon tells you:

```text
What happened on the endpoint.
```

It does not automatically tell you:

```text
Why it happened.
```

That requires:

```text
Context
+
Correlation
+
Analyst reasoning
```

---

# 53. Core Sysmon Mental Model

Think in behaviour chains:

```text
PROCESS
  ↓
DNS
  ↓
NETWORK
  ↓
FILE
  ↓
PROCESS ACCESS
```

Mapped to Sysmon:

```text
1
↓
22
↓
3
↓
11
↓
10
```

This is more useful than memorising isolated Event IDs.

---

# 54. Key Takeaway

The most useful Sysmon question is not:

```text
What does Event ID 3 mean?
```

The better question is:

```text
Which process made this connection,
why did it make it,
and what did it do before and after?
```

For Blue Team investigations, think:

```text
Event
  ↓
Process
  ↓
User
  ↓
Parent
  ↓
Command Line
  ↓
Files
  ↓
DNS
  ↓
Network
  ↓
Timeline
  ↓
Verdict
```

That is how Sysmon becomes an investigation tool instead of just a list of event numbers.
