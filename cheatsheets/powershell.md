# PowerShell Cheat Sheet for Blue Team / SOC Analysis

This cheat sheet focuses on PowerShell commands useful for:

- Windows investigation
- SOC analysis
- Incident response
- Event Log review
- Process analysis
- Network investigation
- File analysis
- Service troubleshooting
- User and authentication analysis

The goal is not to memorise PowerShell syntax.

The goal is to use PowerShell to answer investigation questions quickly and clearly.

---

# 1. PowerShell Basics

PowerShell commands are called:

```text
Cmdlets
```

They commonly follow:

```text
Verb-Noun
```

Examples:

```powershell
Get-Process
Get-Service
Get-ChildItem
Get-WinEvent
Get-FileHash
```

A useful mental model:

```text
Get = retrieve information
Set = change something
New = create something
Remove = delete something
Start = start something
Stop = stop something
Restart = restart something
```

---

# 2. Get Help

Show help for a command:

```powershell
Get-Help Get-Process
```

More detail:

```powershell
Get-Help Get-Process -Full
```

Examples:

```powershell
Get-Help Get-WinEvent -Examples
```

Search commands:

```powershell
Get-Command *event*
```

---

# 3. Current User

```powershell
whoami
```

PowerShell alternative:

```powershell
[System.Security.Principal.WindowsIdentity]::GetCurrent().Name
```

Useful during investigations to confirm execution context.

---

# 4. Hostname

```powershell
hostname
```

PowerShell:

```powershell
$env:COMPUTERNAME
```

---

# 5. Operating System Information

```powershell
Get-ComputerInfo
```

A shorter view:

```powershell
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsArchitecture
```

---

# 6. PowerShell Version

```powershell
$PSVersionTable
```

Architecture can also matter in the current ARM64 lab.

---

# 7. Environment Variables

List:

```powershell
Get-ChildItem Env:
```

Specific variable:

```powershell
$env:USERNAME
```

```powershell
$env:COMPUTERNAME
```

```powershell
$env:TEMP
```

---

# 8. Navigation

Current directory:

```powershell
Get-Location
```

Alias:

```powershell
pwd
```

List files:

```powershell
Get-ChildItem
```

Alias:

```powershell
dir
```

Include hidden files:

```powershell
Get-ChildItem -Force
```

Change directory:

```powershell
Set-Location C:\Windows
```

Alias:

```powershell
cd C:\Windows
```

---

# 9. Recursive File Listing

```powershell
Get-ChildItem C:\Temp -Recurse
```

Files only:

```powershell
Get-ChildItem C:\Temp -Recurse -File
```

---

# 10. Search for Files by Name

```powershell
Get-ChildItem C:\ -Recurse -Filter suspicious.exe -ErrorAction SilentlyContinue
```

Search extensions:

```powershell
Get-ChildItem C:\Users -Recurse -Filter *.ps1 -ErrorAction SilentlyContinue
```

---

# 11. Recently Modified Files

Modified in the last hour:

```powershell
Get-ChildItem C:\Temp -File |
Where-Object {
    $_.LastWriteTime -gt (Get-Date).AddHours(-1)
}
```

Modified in the last day:

```powershell
Get-ChildItem C:\Temp -File |
Where-Object {
    $_.LastWriteTime -gt (Get-Date).AddDays(-1)
}
```

---

# 12. File Metadata

```powershell
Get-Item C:\Temp\example.exe
```

More detail:

```powershell
Get-Item C:\Temp\example.exe |
Format-List *
```

Useful fields:

```text
Name
FullName
Length
CreationTime
LastWriteTime
LastAccessTime
Attributes
```

---

# 13. Hash a File

SHA256:

```powershell
Get-FileHash C:\Temp\example.exe -Algorithm SHA256
```

Other available algorithms may include:

```text
SHA1
SHA256
SHA384
SHA512
MD5
```

For investigation work, SHA256 is generally preferred.

---

# 14. Check Digital Signature

```powershell
Get-AuthenticodeSignature C:\Temp\example.exe
```

Useful fields:

```text
Status
SignerCertificate
Path
```

A valid signature can support trust, but:

```text
Signed ≠ Automatically Safe
Unsigned ≠ Automatically Malicious
```

Context still matters.

---

# 15. View File Contents

```powershell
Get-Content C:\Temp\example.txt
```

Last 20 lines:

```powershell
Get-Content C:\Temp\example.log -Tail 20
```

Follow live:

```powershell
Get-Content C:\Temp\example.log -Wait
```

---

# 16. Search Text in Files

Use:

```powershell
Select-String
```

Example:

```powershell
Select-String -Path C:\Logs\*.log -Pattern "failed"
```

Case-insensitive by default.

Multiple terms:

```powershell
Select-String -Path C:\Logs\*.log -Pattern "failed","error","denied"
```

Recursive:

```powershell
Get-ChildItem C:\Logs -Recurse -File |
Select-String -Pattern "failed"
```

---

# 17. Processes

List processes:

```powershell
Get-Process
```

Specific process:

```powershell
Get-Process powershell
```

By PID:

```powershell
Get-Process -Id 1234
```

Useful fields:

```powershell
Get-Process |
Select-Object Id, ProcessName, CPU, Path
```

Note:

```text
Path may require elevated privileges for some processes.
```

---

# 18. Process Details with CIM

PowerShell process objects do not always expose the command line.

Use CIM:

```powershell
Get-CimInstance Win32_Process
```

Useful view:

```powershell
Get-CimInstance Win32_Process |
Select-Object ProcessId, ParentProcessId, Name, ExecutablePath, CommandLine
```

This is very useful for SOC investigation.

---

# 19. Find a Process by Name

```powershell
Get-CimInstance Win32_Process |
Where-Object {
    $_.Name -eq "powershell.exe"
}
```

---

# 20. Inspect a Process by PID

```powershell
Get-CimInstance Win32_Process |
Where-Object {
    $_.ProcessId -eq 1234
} |
Select-Object ProcessId, ParentProcessId, Name, ExecutablePath, CommandLine
```

---

# 21. Find Parent Process

Suppose PID:

```text
1234
```

First:

```powershell
$p = Get-CimInstance Win32_Process -Filter "ProcessId = 1234"
```

Then:

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = $($p.ParentProcessId)"
```

This helps build process relationships.

---

# 22. Process Tree Investigation

PowerShell does not provide the cleanest native process-tree output, but parent-child relationships can still be investigated using:

```powershell
Get-CimInstance Win32_Process |
Select-Object ProcessId, ParentProcessId, Name, CommandLine
```

Questions:

```text
What launched the process?

Was the parent expected?

Did it launch suspicious children?

Was the command line normal?
```

---

# 23. Stop a Process

```powershell
Stop-Process -Id 1234
```

Force:

```powershell
Stop-Process -Id 1234 -Force
```

During incident response:

> Do not terminate suspicious processes before collecting necessary evidence unless containment is required immediately.

---

# 24. Services

List services:

```powershell
Get-Service
```

Specific:

```powershell
Get-Service Wazuh
```

Running:

```powershell
Get-Service |
Where-Object {
    $_.Status -eq "Running"
}
```

---

# 25. Start / Stop / Restart Service

```powershell
Start-Service Wazuh
```

```powershell
Stop-Service Wazuh
```

```powershell
Restart-Service Wazuh
```

For the Wazuh Windows Agent, the lab has also used:

```cmd
NET START Wazuh
```

and:

```cmd
NET STOP Wazuh
```

---

# 26. Service Configuration

Use CIM:

```powershell
Get-CimInstance Win32_Service
```

Useful view:

```powershell
Get-CimInstance Win32_Service |
Select-Object Name, State, StartMode, PathName
```

Look for:

```text
Unexpected executable path
Suspicious startup mode
Unusual service name
```

---

# 27. Network Configuration

```powershell
Get-NetIPConfiguration
```

IPv4 addresses:

```powershell
Get-NetIPAddress -AddressFamily IPv4
```

Adapters:

```powershell
Get-NetAdapter
```

---

# 28. Routing Table

```powershell
Get-NetRoute
```

IPv4:

```powershell
Get-NetRoute -AddressFamily IPv4
```

---

# 29. DNS Configuration

```powershell
Get-DnsClientServerAddress
```

---

# 30. DNS Query

```powershell
Resolve-DnsName example.com
```

This command was used in the lab to generate Sysmon Event ID 22.

Useful for testing:

```text
DNS resolution
Sysmon telemetry
Wazuh ingestion
```

---

# 31. Test Network Connection

```powershell
Test-NetConnection example.com
```

Specific port:

```powershell
Test-NetConnection example.com -Port 443
```

This command was used in the lab to generate network telemetry.

Useful fields include:

```text
ComputerName
RemoteAddress
RemotePort
TcpTestSucceeded
```

---

# 32. Test Wazuh Port

Example:

```powershell
Test-NetConnection 172.16.106.134 -Port 1514
```

Use your current Wazuh server IP rather than assuming this DHCP address remains unchanged.

---

# 33. Active TCP Connections

```powershell
Get-NetTCPConnection
```

Established only:

```powershell
Get-NetTCPConnection -State Established
```

Listening:

```powershell
Get-NetTCPConnection -State Listen
```

---

# 34. Network Connection with Process ID

```powershell
Get-NetTCPConnection |
Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess
```

Then investigate the PID:

```powershell
Get-Process -Id <PID>
```

or:

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = <PID>"
```

---

# 35. Example Network Pivot

Suppose:

```powershell
Get-NetTCPConnection -State Established
```

shows:

```text
OwningProcess = 4620
RemoteAddress = 203.0.113.50
RemotePort = 443
```

Investigate:

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 4620" |
Select-Object Name, ExecutablePath, CommandLine, ParentProcessId
```

This answers:

```text
Which process owns the connection?
```

---

# 36. Event Logs

PowerShell's main modern Event Log cmdlet is:

```powershell
Get-WinEvent
```

List logs:

```powershell
Get-WinEvent -ListLog *
```

---

# 37. Security Log

Recent Security events:

```powershell
Get-WinEvent -LogName Security -MaxEvents 20
```

---

# 38. Filter by Event ID

Failed logons:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4625
}
```

Successful logons:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4624
}
```

---

# 39. Limit Event Results

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4625
} -MaxEvents 20
```

---

# 40. Event Time Range

Events from last hour:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName   = 'Security'
    StartTime = (Get-Date).AddHours(-1)
}
```

Failed logons in last hour:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName   = 'Security'
    Id        = 4625
    StartTime = (Get-Date).AddHours(-1)
}
```

---

# 41. Specific Time Range

```powershell
$start = Get-Date "2026-09-14 09:00"
$end   = Get-Date "2026-09-14 10:00"

Get-WinEvent -FilterHashtable @{
    LogName   = 'Security'
    StartTime = $start
    EndTime   = $end
}
```

Time ranges are critical during investigations.

---

# 42. View Event Message

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4625
} -MaxEvents 1 |
Format-List TimeCreated, Id, Message
```

---

# 43. Windows Event ID 4624

```text
Successful logon
```

Query:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4624
} -MaxEvents 20
```

Important fields in the message may include:

```text
Account Name
Logon Type
Workstation Name
Source Network Address
Authentication Package
```

---

# 44. Windows Event ID 4625

```text
Failed logon
```

Query:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4625
} -MaxEvents 20
```

Look for:

```text
Account Name
Failure Reason
Status
Sub Status
Logon Type
Workstation
Source Network Address
```

---

# 45. Count Failed Logons

```powershell
(Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4625
}).Count
```

For large logs, narrow the time range first.

---

# 46. Authentication Timeline

Query both:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4624,4625
    StartTime = (Get-Date).AddHours(-2)
} |
Sort-Object TimeCreated |
Select-Object TimeCreated, Id, Message
```

Useful for identifying:

```text
Repeated failures
        ↓
Successful authentication
```

---

# 47. Sysmon Log

Sysmon events are located at:

```text
Microsoft-Windows-Sysmon/Operational
```

Query:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 20
```

---

# 48. Sysmon Event ID 1 — Process Creation

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 1
} -MaxEvents 20
```

Useful fields:

```text
Image
CommandLine
ProcessId
ProcessGuid
ParentImage
ParentCommandLine
User
Hashes
```

---

# 49. Sysmon Event ID 3 — Network Connection

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 3
} -MaxEvents 20
```

Look for:

```text
Image
SourceIp
SourcePort
DestinationIp
DestinationPort
Protocol
User
```

---

# 50. Sysmon Event ID 10 — Process Access

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 10
} -MaxEvents 20
```

This was used in the OneDrive / Explorer investigation.

Important fields:

```text
SourceImage
TargetImage
GrantedAccess
CallTrace
SourceUser
TargetUser
```

---

# 51. Sysmon Event ID 11 — File Create

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 11
} -MaxEvents 20
```

Useful for:

```text
Dropped files
Temporary scripts
Downloaded payloads
New executables
```

---

# 52. Sysmon Event ID 22 — DNS Query

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 22
} -MaxEvents 20
```

Important fields:

```text
Image
QueryName
QueryStatus
ProcessId
ProcessGuid
```

---

# 53. Search Sysmon for Process Name

Example:

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

# 54. Search Sysmon for `whoami.exe`

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 1
} |
Where-Object {
    $_.Message -match "whoami.exe"
} |
Select-Object TimeCreated, Id, Message
```

---

# 55. Search by Process GUID

```powershell
$guid = '{480d0770-c381-6aa7-3e03-000000000c00}'

Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" |
Where-Object {
    $_.Message -match [regex]::Escape($guid)
}
```

Process GUID is extremely useful for correlating Sysmon activity.

---

# 56. Process GUID Correlation

A Process GUID can help correlate:

```text
Event ID 1 — Process Creation
Event ID 3 — Network Connection
Event ID 10 — Process Access
Event ID 11 — File Create
Event ID 22 — DNS Query
```

Conceptually:

```text
ProcessGuid
    │
    ├── Process Created
    ├── DNS Query
    ├── Network Connection
    └── File Creation
```

---

# 57. Event Log Investigation Strategy

Do not search events randomly.

Start with:

```text
Host
Time
User
Event ID
Process
```

Then pivot into:

```text
Parent Process
Command Line
Files
DNS
Network
Related Authentication
```

---

# 58. PowerShell Operational Logs

List available PowerShell logs:

```powershell
Get-WinEvent -ListLog *PowerShell*
```

Common log:

```text
Microsoft-Windows-PowerShell/Operational
```

Query:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 50
```

---

# 59. PowerShell Event ID 4104

If Script Block Logging is enabled:

```text
4104
```

can contain PowerShell script block content.

Query:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id      = 4104
} -MaxEvents 20
```

Very useful for investigating suspicious PowerShell.

---

# 60. PowerShell Event ID 4103

```text
4103
```

can provide module logging information when enabled.

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id      = 4103
}
```

---

# 61. Search PowerShell Logs for Suspicious Terms

Example:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" |
Where-Object {
    $_.Message -match "DownloadString|Invoke-WebRequest|EncodedCommand"
}
```

These strings are investigation pivots, not proof of maliciousness.

---

# 62. Encoded PowerShell Indicators

Common suspicious parameter:

```text
-EncodedCommand
```

Also abbreviated forms may exist.

Search Sysmon process events:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 1
} |
Where-Object {
    $_.Message -match "EncodedCommand"
}
```

---

# 63. Base64 Decode

PowerShell can decode Base64:

```powershell
$encoded = "SGVsbG8="
[System.Text.Encoding]::UTF8.GetString(
    [System.Convert]::FromBase64String($encoded)
)
```

For PowerShell `-EncodedCommand`, UTF-16LE is commonly relevant:

```powershell
[System.Text.Encoding]::Unicode.GetString(
    [System.Convert]::FromBase64String($encoded)
)
```

Always inspect safely.

---

# 64. Local Users

```powershell
Get-LocalUser
```

Specific:

```powershell
Get-LocalUser -Name labadmin
```

---

# 65. Local Groups

```powershell
Get-LocalGroup
```

Administrators:

```powershell
Get-LocalGroupMember -Group Administrators
```

Useful for identifying privileged users.

---

# 66. Logged-In Sessions

Simple:

```powershell
quser
```

or:

```powershell
query user
```

Useful for identifying active sessions.

---

# 67. Scheduled Tasks

List:

```powershell
Get-ScheduledTask
```

Useful view:

```powershell
Get-ScheduledTask |
Select-Object TaskName, TaskPath, State
```

Suspicious areas may include:

```text
Unusual task names
Unexpected executable paths
Scripts in user-writable directories
```

---

# 68. Scheduled Task Details

```powershell
Get-ScheduledTask -TaskName "<TASK>" |
Format-List *
```

Actions:

```powershell
(Get-ScheduledTask -TaskName "<TASK>").Actions
```

---

# 69. Registry

List a registry key:

```powershell
Get-ChildItem HKLM:\Software
```

Read value:

```powershell
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
```

Also:

```powershell
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
```

These are common autorun locations.

---

# 70. Persistence Investigation

Useful areas:

```text
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
Scheduled Tasks
Services
Startup folders
```

Startup folders include:

```text
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
```

and:

```text
%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup
```

---

# 71. Defender Status

```powershell
Get-MpComputerStatus
```

Useful fields:

```text
AntivirusEnabled
RealTimeProtectionEnabled
AntivirusSignatureLastUpdated
```

---

# 72. Defender Detections

```powershell
Get-MpThreatDetection
```

Useful during endpoint investigation.

---

# 73. Firewall Status

```powershell
Get-NetFirewallProfile
```

Useful fields:

```text
Name
Enabled
DefaultInboundAction
DefaultOutboundAction
```

---

# 74. Firewall Rules

```powershell
Get-NetFirewallRule
```

Enabled:

```powershell
Get-NetFirewallRule |
Where-Object {
    $_.Enabled -eq "True"
}
```

---

# 75. Installed Software

One method:

```powershell
Get-ItemProperty `
"HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*" `
-ErrorAction SilentlyContinue |
Select-Object DisplayName, DisplayVersion, Publisher
```

Also check 32-bit software:

```powershell
Get-ItemProperty `
"HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" `
-ErrorAction SilentlyContinue |
Select-Object DisplayName, DisplayVersion, Publisher
```

---

# 76. Running Drivers

```powershell
Get-CimInstance Win32_SystemDriver
```

Useful view:

```powershell
Get-CimInstance Win32_SystemDriver |
Select-Object Name, State, StartMode, PathName
```

---

# 77. SMB Shares

```powershell
Get-SmbShare
```

Sessions:

```powershell
Get-SmbSession
```

Connections:

```powershell
Get-SmbConnection
```

Useful for lateral-movement investigations.

---

# 78. ARP Cache

```powershell
arp -a
```

PowerShell:

```powershell
Get-NetNeighbor
```

---

# 79. DNS Cache

```powershell
Get-DnsClientCache
```

Useful for recent domain resolution context.

---

# 80. Clear DNS Cache

```powershell
Clear-DnsClientCache
```

During an investigation:

> Avoid clearing useful evidence unless necessary.

---

# 81. Command History

Current PowerShell session:

```powershell
Get-History
```

PSReadLine history may exist at:

```powershell
(Get-PSReadLineOption).HistorySavePath
```

Display:

```powershell
Get-Content (Get-PSReadLineOption).HistorySavePath
```

Important:

> History is useful evidence but may be incomplete, cleared, disabled, or manipulated.

---

# 82. Variables

Create:

```powershell
$name = "example"
```

Use:

```powershell
$name
```

Useful for investigations:

```powershell
$start = (Get-Date).AddHours(-1)
```

---

# 83. Pipeline

The PowerShell pipeline:

```text
|
```

passes objects between commands.

Example:

```powershell
Get-Process |
Where-Object {
    $_.CPU -gt 10
}
```

Unlike traditional shells, PowerShell usually passes structured objects rather than plain text.

---

# 84. `Where-Object`

Filter:

```powershell
Get-Service |
Where-Object {
    $_.Status -eq "Running"
}
```

Short form:

```powershell
Get-Service | ? Status -eq "Running"
```

For learning and documentation, full syntax is clearer.

---

# 85. `Select-Object`

Select useful fields:

```powershell
Get-Process |
Select-Object Id, ProcessName, CPU
```

First 10:

```powershell
Get-Process |
Select-Object -First 10
```

Unique:

```powershell
... | Select-Object -Unique
```

---

# 86. `Sort-Object`

Sort:

```powershell
Get-Process |
Sort-Object CPU
```

Descending:

```powershell
Get-Process |
Sort-Object CPU -Descending
```

---

# 87. `Group-Object`

Very useful for SOC analysis.

Example:

```powershell
Get-Process |
Group-Object ProcessName |
Sort-Object Count -Descending
```

For logs:

```text
Group repeated values
Count frequency
Identify outliers
```

---

# 88. `Measure-Object`

Count:

```powershell
Get-Process | Measure-Object
```

Count property:

```powershell
(Get-Process | Measure-Object).Count
```

---

# 89. Export Results

CSV:

```powershell
Get-Process |
Export-Csv C:\Temp\processes.csv -NoTypeInformation
```

Text:

```powershell
Get-Process |
Out-File C:\Temp\processes.txt
```

Useful for evidence collection.

---

# 90. Format Output

Table:

```powershell
Get-Process |
Format-Table
```

List:

```powershell
Get-Process -Id 1234 |
Format-List *
```

Remember:

> Use `Format-*` primarily for display, not in the middle of a pipeline you still need to process.

---

# 91. Error Handling

Hide non-critical errors:

```powershell
-ErrorAction SilentlyContinue
```

Example:

```powershell
Get-ChildItem C:\ -Recurse -Filter *.exe -ErrorAction SilentlyContinue
```

Useful during broad searches.

---

# 92. Execution Policy

Check:

```powershell
Get-ExecutionPolicy
```

All scopes:

```powershell
Get-ExecutionPolicy -List
```

Important:

> PowerShell execution policy is not a strong security boundary.

Do not treat it as one.

---

# 93. PowerShell Script Policy Test Files

The lab previously observed a Wazuh alert involving a file similar to:

```text
__PSScriptPolicyTest_*.ps1
```

PowerShell can create temporary script-policy test files as part of normal behaviour.

This is a good example of:

```text
Suspicious-looking artifact
      ≠
Automatically malicious
```

Always investigate context.

---

# 94. Common Suspicious PowerShell Patterns

Useful investigation pivots include:

```text
-EncodedCommand
Invoke-Expression
IEX
DownloadString
DownloadFile
Invoke-WebRequest
Start-BitsTransfer
FromBase64String
Reflection
Hidden
WindowStyle Hidden
ExecutionPolicy Bypass
```

These are indicators for further analysis, not automatic proof of compromise.

---

# 95. Search Running Process Commands for PowerShell

```powershell
Get-CimInstance Win32_Process |
Where-Object {
    $_.CommandLine -match "powershell"
} |
Select-Object ProcessId, ParentProcessId, Name, CommandLine
```

---

# 96. Search for Encoded Commands

```powershell
Get-CimInstance Win32_Process |
Where-Object {
    $_.CommandLine -match "EncodedCommand"
} |
Select-Object ProcessId, Name, CommandLine
```

---

# 97. Basic Endpoint Triage

Useful starting commands:

```powershell
hostname
whoami
Get-Date
Get-ComputerInfo
Get-LocalUser
Get-Process
Get-NetTCPConnection
Get-NetIPConfiguration
Get-Service
Get-ScheduledTask
```

---

# 98. Fast SOC Triage Block

```powershell
Write-Host "===== HOST ====="
hostname

Write-Host "===== USER ====="
whoami

Write-Host "===== TIME ====="
Get-Date

Write-Host "===== NETWORK ====="
Get-NetIPConfiguration

Write-Host "===== CONNECTIONS ====="
Get-NetTCPConnection

Write-Host "===== PROCESSES ====="
Get-CimInstance Win32_Process |
Select-Object ProcessId, ParentProcessId, Name, ExecutablePath, CommandLine

Write-Host "===== SERVICES ====="
Get-Service

Write-Host "===== LOGGED-IN USERS ====="
quser
```

Use only within your own or authorised environments.

---

# 99. Windows Investigation Workflow

A practical endpoint workflow:

```text
Identify Host
     ↓
Check Time
     ↓
Identify User
     ↓
Review Authentication
     ↓
Review Processes
     ↓
Build Process Relationships
     ↓
Check Files
     ↓
Check DNS
     ↓
Check Network Connections
     ↓
Check Persistence
     ↓
Correlate Sysmon
     ↓
Build Timeline
     ↓
Reach Verdict
```

---

# 100. Example — Failed Login Investigation

Start:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4625
    StartTime = (Get-Date).AddHours(-1)
} |
Select-Object TimeCreated, Id, Message
```

Questions:

```text
Which account failed?

Which source IP?

Which logon type?

How many attempts?

What was the failure reason?
```

Then look for successful logons:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4624
    StartTime = (Get-Date).AddHours(-1)
}
```

Question:

```text
Did repeated failures eventually lead to success?
```

---

# 101. Example — Suspicious Process Investigation

Suppose:

```text
powershell.exe
```

looks suspicious.

Search process creation:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 1
} |
Where-Object {
    $_.Message -match "powershell.exe"
}
```

Inspect:

```text
CommandLine
ParentImage
ParentCommandLine
User
ProcessGuid
```

Then pivot using ProcessGuid into:

```text
DNS
Network
File Creation
Process Access
```

---

# 102. Example — DNS to Network Pivot

Search DNS:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 22
} |
Where-Object {
    $_.Message -match "example.com"
}
```

Identify:

```text
ProcessGuid
ProcessId
Image
Timestamp
```

Then search network events around the same time.

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 3
}
```

Correlate using:

```text
ProcessGuid
ProcessId
Timestamp
Destination
```

---

# 103. Example — File Investigation

For:

```text
C:\Temp\suspicious.exe
```

Run:

```powershell
Get-Item C:\Temp\suspicious.exe |
Format-List *
```

Then:

```powershell
Get-FileHash C:\Temp\suspicious.exe -Algorithm SHA256
```

Then:

```powershell
Get-AuthenticodeSignature C:\Temp\suspicious.exe
```

Then determine whether it executed using Sysmon Event ID 1.

---

# 104. Evidence Collection

Example:

```powershell
New-Item -ItemType Directory -Path C:\Evidence -ErrorAction SilentlyContinue
```

Processes:

```powershell
Get-CimInstance Win32_Process |
Export-Csv C:\Evidence\processes.csv -NoTypeInformation
```

Connections:

```powershell
Get-NetTCPConnection |
Export-Csv C:\Evidence\network-connections.csv -NoTypeInformation
```

Services:

```powershell
Get-CimInstance Win32_Service |
Export-Csv C:\Evidence\services.csv -NoTypeInformation
```

In real forensic investigations, follow your organisation's evidence-handling procedures.

---

# 105. Important Event IDs

Useful Windows Security Event IDs:

| Event ID | Meaning |
|---|---|
| `4624` | Successful logon |
| `4625` | Failed logon |
| `4634` | Logoff |
| `4648` | Explicit credentials used |
| `4672` | Special privileges assigned |
| `4688` | Process creation |
| `4697` | Service installed |
| `4720` | User created |
| `4728` | User added to global group |
| `4732` | User added to local group |
| `4740` | Account locked |
| `4768` | Kerberos TGT |
| `4769` | Kerberos service ticket |
| `4771` | Kerberos pre-auth failed |
| `4776` | Credential validation |

---

# 106. Important Sysmon Event IDs

| Event ID | Meaning |
|---|---|
| `1` | Process Creation |
| `3` | Network Connection |
| `7` | Image Loaded |
| `10` | Process Access |
| `11` | File Create |
| `22` | DNS Query |

---

# 107. Commands to Know First

Prioritise:

```text
Get-Help
Get-Command
Get-Process
Get-CimInstance
Get-Service
Get-WinEvent
Get-ChildItem
Get-Content
Select-String
Get-FileHash
Get-AuthenticodeSignature
Get-NetIPConfiguration
Get-NetTCPConnection
Resolve-DnsName
Test-NetConnection
Get-LocalUser
Get-LocalGroupMember
Get-ScheduledTask
Get-ItemProperty
```

---

# 108. Commands to Learn Next

```text
Where-Object
Select-Object
Sort-Object
Group-Object
Measure-Object
Export-Csv
Get-MpComputerStatus
Get-MpThreatDetection
Get-NetFirewallRule
Get-SmbSession
Get-NetNeighbor
Get-DnsClientCache
```

---

# 109. Analyst Mental Model

PowerShell is most useful when connected to a question.

Example:

```text
Question:
Which process owns this network connection?
```

Command:

```powershell
Get-NetTCPConnection
```

Then:

```text
Question:
What is the process command line?
```

Command:

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = <PID>"
```

Then:

```text
Question:
What launched it?
```

Use:

```text
ParentProcessId
```

Then:

```text
Question:
Did it resolve domains or create files?
```

Search Sysmon.

This is an investigation.

---

# 110. Key Takeaway

Do not think of PowerShell as a list of commands.

Think of it as a way to answer questions.

```text
Who am I?
whoami

Which computer is this?
hostname

What is running?
Get-Process

How was it launched?
Get-CimInstance Win32_Process

What connected to the network?
Get-NetTCPConnection

What domain was resolved?
Resolve-DnsName / Sysmon Event 22

What happened in Windows logs?
Get-WinEvent

What was created?
Sysmon Event 11

What is the file hash?
Get-FileHash

Is the executable signed?
Get-AuthenticodeSignature

Who logged in?
4624

Who failed to log in?
4625
```

The most valuable skill is not remembering every cmdlet.

It is knowing:

```text
Which evidence do I need next?
```
