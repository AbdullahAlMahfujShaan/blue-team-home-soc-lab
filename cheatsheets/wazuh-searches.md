# Wazuh Search Cheat Sheet for Blue Team / SOC Analysis

This cheat sheet focuses on practical Wazuh searches and investigation patterns useful for:

- SOC alert triage
- Windows investigations
- Sysmon analysis
- Authentication investigations
- Process analysis
- DNS analysis
- Network activity
- File activity
- Threat hunting
- Incident investigation

The goal is not to memorise search syntax.

The goal is to understand:

```text
What am I investigating?
        ↓
Which field identifies it?
        ↓
What should I search next?
```

---

# 1. Current Lab Context

The current home SOC lab uses:

```text
BTL-WIN11
   ↓
Windows Event Logs
   +
Sysmon
   ↓
Wazuh Agent
   ↓
BTL-WAZUH
   ↓
Wazuh Manager
   ↓
Wazuh Indexer
   ↓
Wazuh Dashboard
```

The Windows endpoint is registered as:

```text
BTL-WIN11
```

The Wazuh agent ID is:

```text
001
```

---

# 2. Important Wazuh Concept

Wazuh alerts are not the same thing as all endpoint telemetry.

The normal detection flow is:

```text
Raw Telemetry
     ↓
Wazuh Agent
     ↓
Wazuh Manager
     ↓
Detection Rules
     ↓
Alert
```

Therefore:

```text
Raw Event ≠ Alert
```

and:

```text
No Alert ≠ No Activity
```

An event may exist locally in Windows or Sysmon without creating a Wazuh alert.

---

# 3. Main Alert Index

Wazuh alert data is typically stored under:

```text
wazuh-alerts-*
```

A date-specific index may look similar to:

```text
wazuh-alerts-4.x-2026.09.14
```

The exact index naming depends on the Wazuh deployment and date.

---

# 4. Start Broad

When beginning an investigation, avoid overly specific searches immediately.

Start broad enough to understand the available data.

Example:

```text
agent.name:"BTL-WIN11"
```

This narrows results to the Windows lab endpoint.

---

# 5. Search by Agent Name

```text
agent.name:"BTL-WIN11"
```

Use when:

```text
Investigating all alerts associated with one endpoint.
```

Questions:

```text
What alerts occurred?

Which rules triggered?

Which event types are present?

What happened around the relevant time?
```

---

# 6. Search by Agent ID

```text
agent.id:"001"
```

Useful when the agent name is unknown or when searching by Wazuh agent identifier.

---

# 7. Search by Rule ID

Example:

```text
rule.id:"92910"
```

Use when investigating a known Wazuh detection.

Current lab example:

```text
Rule 92910
```

was associated with a process access alert involving:

```text
OneDrive.exe
    ↓
Explorer.EXE
```

---

# 8. Search by Rule Description

Example:

```text
rule.description:"possible process injection"
```

Depending on search behaviour, partial text may be used through the dashboard interface.

Another useful approach is searching a distinctive phrase such as:

```text
process injection
```

Use when:

```text
You know what the alert said but do not know the rule ID.
```

---

# 9. Search by Severity Level

Wazuh alerts include rule levels.

Example:

```text
rule.level:12
```

This can help find high-severity alerts.

However:

```text
High Severity ≠ Confirmed Malicious
```

Severity tells you how the rule is prioritised.

It does not replace investigation.

---

# 10. Search High-Severity Alerts

Example:

```text
rule.level:>=10
```

If the dashboard query interface does not accept this exact syntax, use its numeric filtering controls to select high rule levels.

Use for:

```text
Reviewing high-priority detections.
```

Then investigate each alert individually.

---

# 11. Search by Windows Event ID

Windows event IDs may appear under:

```text
data.win.system.eventID
```

Example:

```text
data.win.system.eventID:"4625"
```

This may identify failed logon events when they have triggered Wazuh alerts.

---

# 12. Successful Logon Search

```text
data.win.system.eventID:"4624"
```

Use to investigate:

```text
Successful authentication
```

Important fields may include:

```text
Account
Logon Type
Source IP
Workstation
Authentication Package
```

Remember:

Not every 4624 necessarily generates an alert.

---

# 13. Failed Logon Search

```text
data.win.system.eventID:"4625"
```

Use when investigating:

- Failed authentication
- Brute-force patterns
- Password spraying
- Mistyped passwords
- Suspicious RDP activity

Questions:

```text
Which account failed?

From which source?

How many attempts?

What logon type?

Was there later a successful 4624?
```

---

# 14. Sysmon Event ID Search

Sysmon event IDs also commonly appear under:

```text
data.win.system.eventID
```

Example:

```text
data.win.system.eventID:"1"
```

This represents:

```text
Sysmon Process Creation
```

if the underlying event was a Sysmon event.

Always validate the event provider or related fields so you do not confuse Sysmon Event ID 1 with an unrelated Windows event source.

---

# 15. Sysmon Event ID 1 — Process Creation

```text
data.win.system.eventID:"1"
```

Useful associated fields may include:

```text
data.win.eventdata.image
data.win.eventdata.commandLine
data.win.eventdata.parentImage
data.win.eventdata.parentCommandLine
data.win.eventdata.processId
data.win.eventdata.processGuid
data.win.eventdata.user
```

---

# 16. Search by Process Name

Example:

```text
data.win.eventdata.image:*powershell.exe
```

Use to identify alerting activity involving PowerShell.

Other examples:

```text
data.win.eventdata.image:*cmd.exe
```

```text
data.win.eventdata.image:*whoami.exe
```

```text
data.win.eventdata.image:*rundll32.exe
```

---

# 17. Search Exact Process Path

Example:

```text
data.win.eventdata.image:"C:\WINDOWS\System32\whoami.exe"
```

Exact path matching can help distinguish:

```text
Legitimate system binary
```

from:

```text
Same filename in an unusual directory
```

---

# 18. Search Process Command Line

Example:

```text
data.win.eventdata.commandLine:*EncodedCommand*
```

Useful suspicious command-line pivots include:

```text
EncodedCommand
ExecutionPolicy Bypass
WindowStyle Hidden
Invoke-Expression
IEX
DownloadString
FromBase64String
Invoke-WebRequest
certutil
bitsadmin
mshta
rundll32
regsvr32
```

These are investigation leads.

They are not proof of compromise.

---

# 19. Search PowerShell Alerts

Example:

```text
data.win.eventdata.image:*powershell.exe
```

Then inspect:

```text
commandLine
parentImage
parentCommandLine
user
processGuid
targetFilename
destinationIp
queryName
```

depending on the event type.

---

# 20. Search by Parent Process

Example:

```text
data.win.eventdata.parentImage:*winword.exe
```

This may help identify suspicious parent-child relationships such as:

```text
winword.exe
    ↓
powershell.exe
```

or:

```text
outlook.exe
    ↓
cmd.exe
```

---

# 21. Parent + Child Process Search

Conceptually search for:

```text
Parent:
winword.exe

Child:
powershell.exe
```

Possible query:

```text
data.win.eventdata.parentImage:*winword.exe AND data.win.eventdata.image:*powershell.exe
```

Use when investigating suspicious process relationships.

---

# 22. Search by Process ID

Example:

```text
data.win.eventdata.processId:"1960"
```

PIDs are useful within a narrow time window.

However:

```text
PID values can be reused.
```

Therefore ProcessGuid is often a stronger Sysmon correlation field.

---

# 23. Search by ProcessGuid

Example from the lab:

```text
data.win.eventdata.processGuid:"{480d0770-bd6a-6aa7-f902-000000000c00}"
```

Use ProcessGuid to correlate multiple activities belonging to the same Sysmon process instance.

Possible related events:

```text
Event 1  — Process Creation
Event 3  — Network Connection
Event 10 — Process Access
Event 11 — File Create
Event 22 — DNS Query
```

---

# 24. Important ProcessGuid Lesson

The lab previously searched for a ProcessGuid related to:

```text
whoami.exe
```

and received no matching Wazuh alerts even though Sysmon recorded the event locally.

This demonstrated:

```text
Local Sysmon Event Exists
        ↓
But No Detection Rule Triggered
        ↓
Therefore No Matching Wazuh Alert
```

This is expected behaviour in an alert index.

---

# 25. Search Sysmon Event ID 3 — Network Connection

```text
data.win.system.eventID:"3"
```

Relevant fields may include:

```text
data.win.eventdata.image
data.win.eventdata.processGuid
data.win.eventdata.sourceIp
data.win.eventdata.sourcePort
data.win.eventdata.destinationIp
data.win.eventdata.destinationPort
data.win.eventdata.protocol
data.win.eventdata.user
```

---

# 26. Search by Destination IP

Example:

```text
data.win.eventdata.destinationIp:"203.0.113.50"
```

Use when:

```text
Investigating communication with a particular IP.
```

Then ask:

```text
Which process connected?

Which user?

Which port?

Was there DNS activity first?
```

---

# 27. Search by Destination Port

Example:

```text
data.win.eventdata.destinationPort:"443"
```

This identifies events involving HTTPS-style TCP port 443 where that field exists.

Other useful examples:

```text
22
53
80
445
3389
```

Do not treat a port number alone as malicious.

---

# 28. Search Network Activity from PowerShell

Example:

```text
data.win.eventdata.image:*powershell.exe AND data.win.eventdata.destinationIp:*
```

Use to investigate alerts where PowerShell generated network-related activity.

---

# 29. Search by Protocol

Example:

```text
data.win.eventdata.protocol:"tcp"
```

or:

```text
data.win.eventdata.protocol:"udp"
```

Useful when narrowing network telemetry.

---

# 30. Search Sysmon Event ID 22 — DNS Query

```text
data.win.system.eventID:"22"
```

Relevant fields may include:

```text
data.win.eventdata.image
data.win.eventdata.processGuid
data.win.eventdata.processId
data.win.eventdata.queryName
data.win.eventdata.queryStatus
data.win.eventdata.queryResults
data.win.eventdata.user
```

---

# 31. Search by DNS Domain

Example:

```text
data.win.eventdata.queryName:"example.com"
```

Wildcard:

```text
data.win.eventdata.queryName:*example.com
```

Use to determine:

```text
Which process queried the domain?
```

---

# 32. Search Suspicious TLD or Domain Pattern

Example:

```text
data.win.eventdata.queryName:*.xyz
```

or:

```text
data.win.eventdata.queryName:*suspicious*
```

Use only as a hunting starting point.

A domain suffix does not prove maliciousness.

---

# 33. DNS to Network Pivot

Investigation flow:

```text
DNS Alert / Event
      ↓
QueryName
      ↓
ProcessGuid
      ↓
Search Event 3
      ↓
Destination IP
      ↓
Network Context
```

Questions:

```text
Did the same process connect after the DNS query?

Was the destination one of the resolved addresses?

How quickly did the connection occur?
```

---

# 34. Search Sysmon Event ID 11 — File Creation

```text
data.win.system.eventID:"11"
```

Relevant fields may include:

```text
data.win.eventdata.image
data.win.eventdata.processGuid
data.win.eventdata.processId
data.win.eventdata.targetFilename
data.win.eventdata.creationUtcTime
data.win.eventdata.user
```

---

# 35. Search by Created File

Example:

```text
data.win.eventdata.targetFilename:*\.exe
```

Other examples:

```text
data.win.eventdata.targetFilename:*\.ps1
```

```text
data.win.eventdata.targetFilename:*\.dll
```

```text
data.win.eventdata.targetFilename:*\.zip
```

Treat extensions as leads, not verdicts.

---

# 36. Search Temporary Directory Activity

Example:

```text
data.win.eventdata.targetFilename:*\\Temp\\*
```

Other useful paths:

```text
\AppData\
\Downloads\
\Users\Public\
\Windows\Temp\
```

User-writable paths deserve context.

They are not inherently malicious.

---

# 37. PowerShell File Creation Search

Example:

```text
data.win.eventdata.image:*powershell.exe AND data.win.eventdata.targetFilename:*
```

This may help identify files created by PowerShell where an alert was generated.

---

# 38. Previous PowerShell Alert

The lab previously observed a Wazuh alert involving:

```text
C:\WINDOWS\SysWOW64\WindowsPowerShell\v1.0\powershell.exe
```

and a target file similar to:

```text
C:\Windows\SystemTemp\__PSScriptPolicyTest_*.ps1
```

Relevant values included:

```text
Sysmon Event ID 11
```

and:

```text
Rule ID 92205
```

This is a useful future case for investigation.

---

# 39. Search Rule 92205

```text
rule.id:"92205"
```

Then review:

```text
data.win.eventdata.image
data.win.eventdata.targetFilename
data.win.eventdata.processGuid
data.win.eventdata.processId
data.win.eventdata.user
```

Do not conclude that the alert is malicious based only on severity.

---

# 40. Search Sysmon Event ID 10 — Process Access

```text
data.win.system.eventID:"10"
```

Important fields may include:

```text
data.win.eventdata.sourceImage
data.win.eventdata.targetImage
data.win.eventdata.sourceProcessGuid
data.win.eventdata.targetProcessGuid
data.win.eventdata.sourceProcessId
data.win.eventdata.targetProcessId
data.win.eventdata.grantedAccess
data.win.eventdata.callTrace
data.win.eventdata.sourceUser
data.win.eventdata.targetUser
```

---

# 41. Search by Source Process

Lab example:

```text
data.win.eventdata.sourceImage:*OneDrive.exe
```

This searches for alerts where OneDrive was the source process.

---

# 42. Search by Target Process

Example:

```text
data.win.eventdata.targetImage:*Explorer.EXE
```

Useful for process access investigations.

---

# 43. Source + Target Search

Lab example:

```text
data.win.eventdata.sourceImage:*OneDrive.exe AND data.win.eventdata.targetImage:*Explorer.EXE
```

This can narrow results to the relationship:

```text
OneDrive.exe
      ↓
Explorer.EXE
```

---

# 44. Search GrantedAccess

Example:

```text
data.win.eventdata.grantedAccess:"0x101411"
```

The current investigation included:

```text
0x101411
```

Remember:

```text
GrantedAccess alone does not prove process injection.
```

Interpret it together with:

```text
Source process
Target process
Call trace
User
Timeline
Related activity
```

---

# 45. Search CallTrace

Example:

```text
data.win.eventdata.callTrace:*FileSyncClient.dll*
```

This can help identify which modules participated in a process access operation.

The OneDrive investigation contained Microsoft OneDrive components such as:

```text
FileSyncClient.dll
FileSyncEvents.dll
FileSyncHost.DLL
```

which supported the benign interpretation.

---

# 46. Search by User

Example:

```text
data.win.eventdata.user:"BTL-WIN11\labadmin"
```

Depending on event type, usernames may appear in different fields.

Other possible fields include:

```text
data.win.eventdata.targetUserName
data.win.eventdata.subjectUserName
data.win.eventdata.sourceUser
data.win.eventdata.targetUser
```

Always inspect the event document to confirm field names.

---

# 47. Search for `labadmin`

Broad search:

```text
labadmin
```

More targeted searches:

```text
data.win.eventdata.user:*labadmin
```

```text
data.win.eventdata.targetUserName:*labadmin
```

Use when building user activity timelines.

---

# 48. Authentication User Search

Example:

```text
data.win.system.eventID:"4625" AND data.win.eventdata.targetUserName:*testuser
```

This could be useful during failed-login exercises involving the standard account.

---

# 49. Source IP Search

Authentication-related events may contain a field such as:

```text
data.win.eventdata.ipAddress
```

Example:

```text
data.win.eventdata.ipAddress:"172.16.106.132"
```

Field names can vary depending on the decoder and event.

Inspect the event document rather than assuming.

---

# 50. Search Failed Logons from One IP

Conceptually:

```text
data.win.system.eventID:"4625" AND data.win.eventdata.ipAddress:"<SOURCE-IP>"
```

Use to answer:

```text
How many authentication failures came from this source?
```

---

# 51. Search Failed and Successful Authentication

You may search both event IDs:

```text
data.win.system.eventID:("4624" OR "4625")
```

If the interface does not support this syntax directly, use separate searches or dashboard filters.

Goal:

```text
Failures
   ↓
Potential Success
```

---

# 52. Brute-Force Investigation Pattern

Search:

```text
4625
```

Then group mentally or using dashboard aggregations by:

```text
TargetUserName
Source IP
Time
Logon Type
```

Look for:

```text
4625
4625
4625
4625
4624
```

This may indicate:

```text
Repeated failure followed by success
```

but still requires context.

---

# 53. Search by Logon Type

Authentication events may expose:

```text
data.win.eventdata.logonType
```

Example:

```text
data.win.eventdata.logonType:"10"
```

Meaning:

```text
Remote Interactive / RDP
```

Other useful values:

```text
2  = Interactive
3  = Network
5  = Service
7  = Unlock
10 = Remote Interactive
```

---

# 54. Failed RDP Search

Conceptually:

```text
data.win.system.eventID:"4625" AND data.win.eventdata.logonType:"10"
```

This can help identify failed Remote Desktop authentication.

---

# 55. Successful RDP Search

```text
data.win.system.eventID:"4624" AND data.win.eventdata.logonType:"10"
```

Then inspect:

```text
User
Source IP
Time
Follow-on process activity
```

---

# 56. Search by Host + Event ID

Example:

```text
agent.name:"BTL-WIN11" AND data.win.system.eventID:"4625"
```

This is generally better than searching Event ID alone in a multi-host environment.

---

# 57. Search by Host + Process

Example:

```text
agent.name:"BTL-WIN11" AND data.win.eventdata.image:*powershell.exe
```

Use to narrow activity to one endpoint.

---

# 58. Search by Rule + Host

```text
agent.name:"BTL-WIN11" AND rule.id:"92910"
```

Useful when the same rule may trigger on multiple systems.

---

# 59. Search by Filename

Example:

```text
data.win.eventdata.targetFilename:*__PSScriptPolicyTest_*
```

Use to identify alerts related to a particular file or naming pattern.

---

# 60. Search for Executables in User-Writable Paths

Example:

```text
data.win.eventdata.image:*\\AppData\\*
```

or:

```text
data.win.eventdata.image:*\\Temp\\*
```

Questions:

```text
What executable?

Who launched it?

Was it signed?

What was its parent?

Did it make network connections?
```

---

# 61. Search by MITRE Technique

Wazuh rules may include MITRE ATT&CK mappings.

Depending on indexed field mappings, fields may include rule MITRE information.

Examples may include:

```text
rule.mitre.id
rule.mitre.technique
rule.mitre.tactic
```

Possible search:

```text
rule.mitre.id:"T1059.001"
```

This corresponds to:

```text
PowerShell
```

Always check the actual indexed field names in the event document.

---

# 62. Search PowerShell ATT&CK Mapping

Example:

```text
rule.mitre.id:"T1059.001"
```

Then inspect the underlying telemetry.

MITRE mapping means:

```text
Behaviour resembles a known technique
```

It does not mean:

```text
Technique confirmed as malicious
```

---

# 63. Search Process Injection Mapping

Example:

```text
rule.mitre.id:"T1055"
```

Potentially useful for process-injection-related detections.

Again:

```text
MITRE Mapping ≠ Confirmation
```

---

# 64. Search by Rule Group

Wazuh rules can belong to groups.

Depending on field mapping, examples may include categories such as:

```text
windows
sysmon
authentication
powershell
```

Inspect:

```text
rule.groups
```

Possible search:

```text
rule.groups:"sysmon"
```

Use the actual field values visible in your environment.

---

# 65. Search Alert Timestamp

Use the Wazuh Dashboard time picker first.

Examples:

```text
Last 15 minutes
Last 1 hour
Last 24 hours
Custom range
```

Time range is one of the most important investigation controls.

---

# 66. Why Time Range Matters

If an event occurred at:

```text
09:24
```

but your dashboard is showing:

```text
Last 15 minutes
```

at 11:00, the event will appear missing.

Before troubleshooting ingestion, verify:

```text
Time range
Timezone
Event timestamp
```

---

# 67. Investigation Time Window

For an alert at:

```text
10:00
```

start with roughly:

```text
09:55–10:05
```

Then expand if necessary.

Think:

```text
Before
During
After
```

---

# 68. Search Around a Known Process

Suppose you identify:

```text
powershell.exe
```

at:

```text
10:00:00
```

Search nearby alerts for:

```text
10 minutes before
10 minutes after
```

and inspect:

```text
Authentication
Parent process
DNS
Network
File creation
Process access
```

---

# 69. Broad Search vs Narrow Search

Bad investigation pattern:

```text
Start with an extremely specific ProcessGuid
      ↓
No results
      ↓
Assume data is missing
```

Better pattern:

```text
Host
  ↓
Time
  ↓
Rule / Event Type
  ↓
Process
  ↓
User
  ↓
ProcessGuid
```

Start broad, then narrow.

---

# 70. Search Workflow — Process Investigation

Example:

```text
agent.name:"BTL-WIN11"
```

Then:

```text
data.win.eventdata.image:*powershell.exe
```

Then inspect:

```text
ProcessGuid
CommandLine
ParentImage
User
Timestamp
```

Then pivot:

```text
ProcessGuid
```

into related events.

---

# 71. Search Workflow — DNS Investigation

Start:

```text
data.win.system.eventID:"22"
```

Then identify:

```text
QueryName
Image
ProcessGuid
```

Then search:

```text
data.win.eventdata.processGuid:"<GUID>"
```

Then inspect any related:

```text
Event 3
Event 11
Event 1
```

---

# 72. Search Workflow — Authentication

Start:

```text
data.win.system.eventID:"4625"
```

Then identify:

```text
Target user
Source
Logon type
Failure reason
Timestamp
```

Then search:

```text
4624
```

for the same:

```text
User
Source
Time period
```

---

# 73. Search Workflow — Process Access

Start:

```text
data.win.system.eventID:"10"
```

Then inspect:

```text
SourceImage
TargetImage
GrantedAccess
CallTrace
SourceUser
TargetUser
```

Then search nearby alerts for:

```text
Process Creation
DNS
Network
Files
Other process access
```

---

# 74. Search Workflow — Suspicious File

Start with:

```text
data.win.eventdata.targetFilename:*suspicious.exe
```

Then determine:

```text
Which process created it?
```

Find:

```text
ProcessGuid
Image
User
Timestamp
```

Then search whether it executed:

```text
data.win.eventdata.image:*suspicious.exe
```

---

# 75. Search Workflow — PowerShell

Start:

```text
data.win.eventdata.image:*powershell.exe
```

Review:

```text
CommandLine
ParentImage
ParentCommandLine
User
ProcessGuid
```

Search for:

```text
EncodedCommand
ExecutionPolicy
IEX
DownloadString
Invoke-WebRequest
```

Then pivot into:

```text
DNS
Network
Files
Child processes
```

---

# 76. Detection vs Telemetry

Wazuh may show:

```text
Rule 92205
```

but the underlying Sysmon event might be:

```text
Event ID 11
```

Think in two layers:

```text
Layer 1:
Telemetry

Layer 2:
Detection
```

Example:

```text
Sysmon Event 11
      ↓
File Created
      ↓
Wazuh Rule
      ↓
Alert
```

Always investigate both.

---

# 77. Rule Information to Review

For every alert, review:

```text
rule.id
rule.level
rule.description
rule.groups
rule.mitre
```

These explain:

```text
Why did Wazuh alert?
```

---

# 78. Telemetry Information to Review

Then review:

```text
data.win.system.eventID
data.win.system.providerName
data.win.system.computer
data.win.system.systemTime
```

and relevant:

```text
data.win.eventdata.*
```

This explains:

```text
What actually happened?
```

---

# 79. Alert Triage Checklist

For every Wazuh alert, answer:

```text
What rule fired?

What severity?

Which endpoint?

Which user?

Which event ID?

Which process?

What command line?

What parent process?

Any files involved?

Any DNS?

Any network connections?

What happened around the same time?

Does the alert make sense in context?
```

---

# 80. High-Severity Alert Workflow

If Wazuh reports:

```text
Level 12
```

do not immediately classify:

```text
Malicious
```

Instead:

```text
High Severity Alert
       ↓
Understand Rule
       ↓
Inspect Raw Fields
       ↓
Correlate Activity
       ↓
Determine Context
       ↓
Reach Verdict
```

---

# 81. OneDrive Investigation Example

Alert:

```text
Rule ID:
92910
```

Description involved possible process injection.

Search:

```text
rule.id:"92910"
```

Relevant fields:

```text
data.win.system.eventID:"10"

data.win.eventdata.sourceImage:
C:\Users\labadmin\AppData\Local\Microsoft\OneDrive\OneDrive.exe

data.win.eventdata.targetImage:
C:\WINDOWS\Explorer.EXE

data.win.eventdata.grantedAccess:
0x101411
```

---

# 82. OneDrive Investigation Pivot

Search:

```text
data.win.eventdata.sourceImage:*OneDrive.exe
```

Then search:

```text
data.win.eventdata.targetImage:*Explorer.EXE
```

Then:

```text
data.win.eventdata.callTrace:*FileSyncClient.dll*
```

Review surrounding alerts.

Final lab verdict:

```text
Likely benign / benign positive
```

because the available evidence did not indicate malicious injection.

---

# 83. PowerShell Investigation Example

Search rule:

```text
rule.id:"92205"
```

Then inspect:

```text
data.win.eventdata.image
data.win.eventdata.targetFilename
data.win.eventdata.processGuid
data.win.eventdata.processId
data.win.eventdata.user
```

Known example target:

```text
__PSScriptPolicyTest_*.ps1
```

Next pivots:

```text
What launched PowerShell?

What command line?

Did it contact the network?

Did it create anything else?

Was the action expected?
```

---

# 84. Generating Safe Lab Telemetry

The Windows VM can generate safe test activity.

Process activity:

```powershell
whoami
hostname
ipconfig
```

DNS:

```powershell
Resolve-DnsName example.com
```

Network:

```powershell
Test-NetConnection example.com -Port 443
```

These help verify:

```text
Sysmon
      ↓
Wazuh Agent
      ↓
Wazuh
```

---

# 85. Search for `whoami.exe`

Possible search:

```text
data.win.eventdata.image:*whoami.exe
```

If no Wazuh alert appears:

```text
Do not immediately assume Sysmon failed.
```

Check the event locally in:

```text
Microsoft-Windows-Sysmon/Operational
```

This determines whether:

```text
Telemetry exists but did not trigger an alert.
```

---

# 86. Search for `hostname.exe`

```text
data.win.eventdata.image:*hostname.exe
```

Again, lack of alerts does not prove lack of execution.

---

# 87. Search for `ipconfig.exe`

```text
data.win.eventdata.image:*ipconfig.exe
```

Use as a simple discovery-process test.

---

# 88. Search Windows Commands

Useful examples:

```text
data.win.eventdata.image:*whoami.exe
```

```text
data.win.eventdata.image:*hostname.exe
```

```text
data.win.eventdata.image:*ipconfig.exe
```

```text
data.win.eventdata.image:*net.exe
```

```text
data.win.eventdata.image:*netstat.exe
```

```text
data.win.eventdata.image:*tasklist.exe
```

These commands can be legitimate or associated with system discovery.

Context matters.

---

# 89. Search Discovery Behaviour

A sequence such as:

```text
whoami
hostname
ipconfig
net user
tasklist
```

may represent:

```text
Normal administration
```

or:

```text
System discovery after compromise
```

The commands alone do not decide the verdict.

Look at:

```text
User
Parent process
Time
Execution context
Related network activity
```

---

# 90. Search for CMD

```text
data.win.eventdata.image:*cmd.exe
```

Then inspect:

```text
commandLine
parentImage
user
processGuid
```

---

# 91. Search for `rundll32.exe`

```text
data.win.eventdata.image:*rundll32.exe
```

Then inspect its command line.

`rundll32.exe` is legitimate Windows software but can be abused.

---

# 92. Search for `mshta.exe`

```text
data.win.eventdata.image:*mshta.exe
```

Potentially useful during LOLBin investigations.

Again:

```text
Presence ≠ Maliciousness
```

---

# 93. Search for `certutil.exe`

```text
data.win.eventdata.image:*certutil.exe
```

Potential analyst questions:

```text
Why was it executed?

Was it hashing a file?

Downloading something?

Encoding or decoding data?
```

---

# 94. Search for `regsvr32.exe`

```text
data.win.eventdata.image:*regsvr32.exe
```

Inspect command line and parent process.

---

# 95. Search for `wscript.exe` or `cscript.exe`

```text
data.win.eventdata.image:*wscript.exe
```

```text
data.win.eventdata.image:*cscript.exe
```

Useful when investigating script execution.

---

# 96. Search for Suspicious Parent Office Application

Examples:

```text
data.win.eventdata.parentImage:*WINWORD.EXE
```

```text
data.win.eventdata.parentImage:*EXCEL.EXE
```

```text
data.win.eventdata.parentImage:*OUTLOOK.EXE
```

Then inspect child processes.

---

# 97. Search by Computer Name

Windows events may expose:

```text
data.win.system.computer
```

Example:

```text
data.win.system.computer:"BTL-WIN11"
```

This can complement:

```text
agent.name
```

---

# 98. Search by Provider Name

Useful field:

```text
data.win.system.providerName
```

Example Sysmon provider:

```text
Microsoft-Windows-Sysmon
```

Search:

```text
data.win.system.providerName:"Microsoft-Windows-Sysmon"
```

This helps ensure a numeric event ID belongs to Sysmon.

---

# 99. Sysmon Event ID 1 with Provider

Safer query:

```text
data.win.system.providerName:"Microsoft-Windows-Sysmon" AND data.win.system.eventID:"1"
```

This is clearer than searching:

```text
eventID 1
```

alone.

---

# 100. Sysmon Event ID 3 with Provider

```text
data.win.system.providerName:"Microsoft-Windows-Sysmon" AND data.win.system.eventID:"3"
```

---

# 101. Sysmon Event ID 10 with Provider

```text
data.win.system.providerName:"Microsoft-Windows-Sysmon" AND data.win.system.eventID:"10"
```

---

# 102. Sysmon Event ID 11 with Provider

```text
data.win.system.providerName:"Microsoft-Windows-Sysmon" AND data.win.system.eventID:"11"
```

---

# 103. Sysmon Event ID 22 with Provider

```text
data.win.system.providerName:"Microsoft-Windows-Sysmon" AND data.win.system.eventID:"22"
```

---

# 104. Search Windows Security Provider

Possible provider:

```text
Microsoft-Windows-Security-Auditing
```

Example:

```text
data.win.system.providerName:"Microsoft-Windows-Security-Auditing"
```

Combine with:

```text
4625
```

for failed logons.

---

# 105. Search Security Failed Logons

```text
data.win.system.providerName:"Microsoft-Windows-Security-Auditing" AND data.win.system.eventID:"4625"
```

---

# 106. Search Security Successful Logons

```text
data.win.system.providerName:"Microsoft-Windows-Security-Auditing" AND data.win.system.eventID:"4624"
```

---

# 107. Investigation Field Discovery

Wazuh field names can vary between event types.

When unsure:

```text
Open one matching event
      ↓
Expand the document
      ↓
Inspect data.win.eventdata
      ↓
Copy the exact field name
      ↓
Use it in the next search
```

Do not guess field names when the dashboard shows the real structure.

---

# 108. Field Naming Differences

For example, one event may contain:

```text
data.win.eventdata.image
```

while another may use:

```text
data.win.eventdata.sourceImage
```

or:

```text
data.win.eventdata.targetImage
```

The field depends on the Windows event.

---

# 109. Search Strategy — Known Alert

If you know:

```text
Rule ID
```

start with:

```text
rule.id:"<RULE-ID>"
```

Then investigate:

```text
Host
Time
Event ID
User
Process
Related activity
```

---

# 110. Search Strategy — Known Process

If you know:

```text
powershell.exe
```

start with:

```text
data.win.eventdata.image:*powershell.exe
```

Then narrow using:

```text
Host
Time
ProcessGuid
Rule ID
```

---

# 111. Search Strategy — Known User

If you know:

```text
labadmin
```

start with the relevant user field or broad text search.

Then correlate:

```text
Authentication
Processes
Files
Network
```

---

# 112. Search Strategy — Known IP

If you know:

```text
Destination IP
```

search:

```text
data.win.eventdata.destinationIp:"<IP>"
```

If investigating authentication, use the event's source-IP field instead.

---

# 113. Search Strategy — Known Domain

If you know:

```text
suspicious.example
```

search:

```text
data.win.eventdata.queryName:*suspicious.example
```

Then identify:

```text
ProcessGuid
Image
User
Timestamp
```

---

# 114. Search Strategy — Known File

Search:

```text
data.win.eventdata.targetFilename:*filename*
```

Then identify:

```text
Creator process
User
ProcessGuid
Timestamp
```

Then determine whether the file executed.

---

# 115. Search Strategy — Unknown Alert

If all you know is:

```text
Something happened on BTL-WIN11
```

start:

```text
agent.name:"BTL-WIN11"
```

Then set the relevant time window and sort by newest/oldest as needed.

Look for:

```text
High rule levels
Unusual rules
Authentication
PowerShell
Process access
File creation
Network activity
```

---

# 116. Timeline Building

Build a timeline using:

```text
timestamp
rule.id
eventID
user
process
destination
file
```

Example:

```text
09:24:45 — Process started
09:24:50 — DNS query
09:24:52 — File created
09:24:55 — Network connection
09:25:02 — Alert generated
```

---

# 117. Timeline Questions

For each event:

```text
What happened immediately before?

What happened immediately after?

Does another event involve the same process?

Does another event involve the same user?

Does another event involve the same destination?
```

---

# 118. Event Correlation Keys

Useful correlation fields include:

```text
Host
Timestamp
User
ProcessGuid
ProcessId
ParentProcessId
Image
CommandLine
Source IP
Destination IP
Destination Port
QueryName
TargetFilename
```

---

# 119. Strongest Sysmon Correlation Key

For process-centric investigations:

```text
ProcessGuid
```

is often one of the most valuable keys.

Reason:

```text
Process ID can be reused.
ProcessGuid identifies a specific process instance.
```

---

# 120. Searching Too Narrowly

Avoid immediately searching:

```text
One exact GUID
+
One event ID
+
One process
+
One specific second
```

If any field differs, you may miss the activity.

Instead:

```text
Host
  ↓
Time range
  ↓
Process
  ↓
GUID
```

---

# 121. Searching Too Broadly

Conversely, searching everything over:

```text
30 days
```

may return too much noise.

Start with:

```text
Host + Relevant Time Window
```

then expand only if needed.

---

# 122. Timezone Awareness

Always verify whether event timestamps are displayed as:

```text
UTC
```

or:

```text
Local time
```

Sysmon often records fields such as:

```text
UtcTime
```

while dashboards may display timestamps using another timezone.

A timezone mismatch can make events look unrelated.

---

# 123. Dashboard Time Picker Checklist

Before saying:

```text
The alert is missing
```

check:

```text
Correct index?
Correct time range?
Correct timezone?
Correct host?
Correct field?
Alert vs raw telemetry?
```

---

# 124. Wazuh Alert Investigation Model

Use:

```text
RULE
  ↓
What detected it?

EVENT
  ↓
What happened?

ENTITY
  ↓
Which user/process/host?

CONTEXT
  ↓
What happened nearby?

VERDICT
  ↓
Benign, suspicious, malicious?
```

---

# 125. Wazuh Alert Fields to Learn

High-value top-level fields:

```text
timestamp
agent.id
agent.name
rule.id
rule.level
rule.description
rule.groups
rule.mitre
```

Windows-related:

```text
data.win.system.*
data.win.eventdata.*
```

---

# 126. Investigation Question — What Rule Fired?

Search or inspect:

```text
rule.id
rule.description
rule.level
```

---

# 127. Investigation Question — Which Machine?

Inspect:

```text
agent.name
```

and possibly:

```text
data.win.system.computer
```

---

# 128. Investigation Question — Which User?

Potential fields:

```text
data.win.eventdata.user
data.win.eventdata.sourceUser
data.win.eventdata.targetUser
data.win.eventdata.targetUserName
data.win.eventdata.subjectUserName
```

---

# 129. Investigation Question — Which Process?

Potential fields:

```text
data.win.eventdata.image
data.win.eventdata.sourceImage
data.win.eventdata.targetImage
```

---

# 130. Investigation Question — What Command?

Look for:

```text
data.win.eventdata.commandLine
```

and sometimes:

```text
data.win.eventdata.parentCommandLine
```

---

# 131. Investigation Question — What Parent Process?

Look for:

```text
data.win.eventdata.parentImage
data.win.eventdata.parentProcessId
data.win.eventdata.parentCommandLine
```

---

# 132. Investigation Question — What File?

Look for:

```text
data.win.eventdata.targetFilename
```

---

# 133. Investigation Question — Which Domain?

Look for:

```text
data.win.eventdata.queryName
```

---

# 134. Investigation Question — Which Destination?

Look for:

```text
data.win.eventdata.destinationIp
data.win.eventdata.destinationPort
data.win.eventdata.destinationHostname
```

---

# 135. Investigation Question — Which Source?

Possible fields:

```text
data.win.eventdata.sourceIp
data.win.eventdata.sourcePort
data.win.eventdata.ipAddress
```

depending on event type.

---

# 136. Triage Example — High-Level Alert

Suppose Wazuh displays:

```text
Rule Level: 12
Rule ID: 92910
```

Do:

```text
1. Search rule.id:"92910"

2. Identify host

3. Check eventID

4. Identify source process

5. Identify target process

6. Review GrantedAccess

7. Review CallTrace

8. Review user context

9. Search surrounding alerts

10. Reach verdict
```

---

# 137. Triage Example — Failed Login

Search:

```text
data.win.system.eventID:"4625"
```

Then:

```text
TargetUserName
Source IP
Logon Type
FailureReason
Timestamp
```

Next search:

```text
data.win.system.eventID:"4624"
```

for the same account/source around the same time.

---

# 138. Triage Example — Suspicious PowerShell

Search:

```text
data.win.eventdata.image:*powershell.exe
```

Check:

```text
CommandLine
ParentImage
User
ProcessGuid
```

Then search related:

```text
DNS
Network
File Creation
```

---

# 139. Triage Example — Suspicious DNS

Search:

```text
data.win.eventdata.queryName:*domain*
```

Then:

```text
ProcessGuid
Image
User
Timestamp
```

Search related network activity using the same process.

---

# 140. Triage Example — Suspicious File

Search:

```text
data.win.eventdata.targetFilename:*filename*
```

Then identify:

```text
Creator process
ProcessGuid
User
Time
```

Then search process creation for the same file path.

---

# 141. Triage Example — Process Access

Search:

```text
data.win.system.eventID:"10"
```

Review:

```text
SourceImage
TargetImage
GrantedAccess
CallTrace
Users
```

Then ask:

```text
Is there evidence of actual injection?

Were remote threads created?

Was memory manipulation observed?

Are there suspicious follow-on events?
```

---

# 142. Alert ≠ Incident

Keep this distinction:

```text
Telemetry
   ↓
Detection
   ↓
Alert
   ↓
Triage
   ↓
Investigation
   ↓
Verdict
```

Wazuh gives you evidence and detections.

The analyst decides whether an incident exists.

---

# 143. Classification Options

Useful investigation verdicts:

```text
Benign
Benign Positive
False Positive
Suspicious
Malicious
Confirmed Incident
```

---

# 144. Benign Positive Example

The OneDrive ProcessAccess case is a useful example.

The detection correctly observed:

```text
OneDrive.exe accessed Explorer.EXE
```

but the available context supported legitimate software behaviour.

Therefore:

```text
Detection worked
+
Activity legitimate
=
Benign Positive
```

---

# 145. False Positive vs Benign Positive

## False Positive

```text
Detection logic incorrectly identified activity.
```

## Benign Positive

```text
Detection accurately identified the behaviour,
but the behaviour itself was legitimate.
```

This distinction is useful in SOC work.

---

# 146. Useful Search Combinations

Host + PowerShell:

```text
agent.name:"BTL-WIN11" AND data.win.eventdata.image:*powershell.exe
```

Host + failed login:

```text
agent.name:"BTL-WIN11" AND data.win.system.eventID:"4625"
```

Host + Sysmon Process Creation:

```text
agent.name:"BTL-WIN11" AND data.win.system.providerName:"Microsoft-Windows-Sysmon" AND data.win.system.eventID:"1"
```

Rule + host:

```text
rule.id:"92910" AND agent.name:"BTL-WIN11"
```

Process access:

```text
data.win.eventdata.sourceImage:*OneDrive.exe AND data.win.eventdata.targetImage:*Explorer.EXE
```

---

# 147. Useful Search Patterns to Practise

```text
agent.name:"BTL-WIN11"
```

```text
rule.id:"92910"
```

```text
rule.id:"92205"
```

```text
data.win.system.eventID:"4625"
```

```text
data.win.system.eventID:"4624"
```

```text
data.win.system.providerName:"Microsoft-Windows-Sysmon" AND data.win.system.eventID:"1"
```

```text
data.win.eventdata.image:*powershell.exe
```

```text
data.win.eventdata.queryName:*example.com
```

```text
data.win.eventdata.targetFilename:*\.ps1
```

```text
data.win.eventdata.sourceImage:*OneDrive.exe
```

---

# 148. Searches to Know First

Prioritise:

```text
agent.name
rule.id
rule.level
rule.description
data.win.system.eventID
data.win.system.providerName
data.win.eventdata.image
data.win.eventdata.commandLine
data.win.eventdata.parentImage
data.win.eventdata.processGuid
data.win.eventdata.user
data.win.eventdata.targetFilename
data.win.eventdata.queryName
data.win.eventdata.destinationIp
data.win.eventdata.destinationPort
```

---

# 149. Searches to Learn Next

Then learn:

```text
sourceImage
targetImage
callTrace
grantedAccess
sourceUser
targetUser
targetUserName
logonType
ipAddress
sourceIp
sourcePort
parentCommandLine
rule.mitre.id
rule.groups
```

---

# 150. SOC Investigation Workflow

When starting from a Wazuh alert:

```text
1. Read Alert Description
2. Record Rule ID
3. Record Severity
4. Identify Endpoint
5. Identify Event ID
6. Identify User
7. Identify Process
8. Review Command Line
9. Review Parent Process
10. Record ProcessGuid
11. Search Related Events
12. Check DNS
13. Check Network
14. Check Files
15. Check Authentication
16. Build Timeline
17. Compare Hypotheses
18. Reach Verdict
19. Document Findings
```

---

# 151. Search Logic Mental Model

Think:

```text
WHO?
User fields

WHAT?
Event ID / process / file

WHERE?
Host / IP / domain

WHEN?
Timestamp / dashboard time range

WHY?
Rule description / detection logic

WHAT NEXT?
Pivot using correlated fields
```

---

# 152. Evidence Correlation Mental Model

Example:

```text
Rule Alert
   ↓
Process
   ↓
ProcessGuid
   ├── DNS
   ├── Network
   ├── File
   └── Process Access
```

Then combine with:

```text
Authentication
```

to determine who initiated the activity.

---

# 153. Most Important Wazuh Lesson

Do not ask only:

```text
What alert fired?
```

Ask:

```text
What underlying behaviour caused the alert?
```

The rule helps you find the activity.

The telemetry helps you understand it.

---

# 154. When Search Returns Zero Results

Check:

```text
1. Correct time range?

2. Correct host?

3. Correct index?

4. Correct field name?

5. Exact value vs wildcard?

6. Correct case?

7. Event alerting at all?

8. Raw event only?

9. Sysmon locally recording it?

10. Wazuh agent active?
```

---

# 155. Local Verification

If Wazuh does not show an expected Sysmon event, verify Windows locally:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 20
```

For Event ID 1:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id      = 1
} -MaxEvents 20
```

---

# 156. Check Wazuh Agent Service

On Windows:

```cmd
NET START Wazuh
```

or PowerShell:

```powershell
Get-Service Wazuh
```

Restart:

```powershell
Restart-Service Wazuh
```

---

# 157. Wazuh Agent Log

Windows agent log:

```text
C:\Program Files (x86)\ossec-agent\ossec.log
```

View recent content:

```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 50
```

Follow live:

```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Wait
```

---

# 158. Verify Sysmon Collection Configuration

Wazuh agent configuration:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Expected Sysmon entry:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

---

# 159. Restart After Config Change

```cmd
NET STOP Wazuh
NET START Wazuh
```

or:

```powershell
Restart-Service Wazuh
```

---

# 160. Agent Communication Ports

Common Wazuh ports used in the lab:

```text
1514/TCP — Agent communication
1515/TCP — Agent enrollment
55000/TCP — API-related communication
```

Test communication:

```powershell
Test-NetConnection <WAZUH-SERVER-IP> -Port 1514
```

---

# 161. Wazuh Search Pitfalls

Avoid:

- Searching the wrong time range
- Assuming every event creates an alert
- Using a field that does not exist in that event type
- Ignoring the event provider
- Searching only by severity
- Treating the rule description as the final verdict
- Ignoring ProcessGuid
- Ignoring parent process
- Ignoring user context
- Ignoring surrounding telemetry
- Assuming zero search hits means no activity occurred

---

# 162. Detection Rule vs Analyst Verdict

Example:

```text
Detection Rule:
Possible Process Injection
```

Analyst finding:

```text
No additional evidence of malicious process injection identified.
```

Verdict:

```text
Likely benign.
```

This is normal SOC work.

---

# 163. Query Documentation Practice

For every useful search, record:

```text
Purpose
Query
What fields to inspect
What result means
Next pivot
```

Example:

```text
Purpose:
Find failed Windows authentication alerts.

Query:
data.win.system.eventID:"4625"

Inspect:
TargetUserName
Source IP
LogonType
FailureReason

Next:
Search 4624 for the same user/source.
```

---

# 164. Quick Reference — Authentication

Failed logon:

```text
data.win.system.eventID:"4625"
```

Successful logon:

```text
data.win.system.eventID:"4624"
```

RDP failure:

```text
data.win.system.eventID:"4625" AND data.win.eventdata.logonType:"10"
```

RDP success:

```text
data.win.system.eventID:"4624" AND data.win.eventdata.logonType:"10"
```

---

# 165. Quick Reference — Processes

PowerShell:

```text
data.win.eventdata.image:*powershell.exe
```

CMD:

```text
data.win.eventdata.image:*cmd.exe
```

Whoami:

```text
data.win.eventdata.image:*whoami.exe
```

Parent process:

```text
data.win.eventdata.parentImage:*WINWORD.EXE
```

ProcessGuid:

```text
data.win.eventdata.processGuid:"<GUID>"
```

---

# 166. Quick Reference — Network

Destination IP:

```text
data.win.eventdata.destinationIp:"<IP>"
```

Port:

```text
data.win.eventdata.destinationPort:"443"
```

Protocol:

```text
data.win.eventdata.protocol:"tcp"
```

---

# 167. Quick Reference — DNS

Domain:

```text
data.win.eventdata.queryName:"example.com"
```

Wildcard:

```text
data.win.eventdata.queryName:*example.com
```

Sysmon DNS event:

```text
data.win.system.providerName:"Microsoft-Windows-Sysmon" AND data.win.system.eventID:"22"
```

---

# 168. Quick Reference — Files

Target filename:

```text
data.win.eventdata.targetFilename:*filename*
```

PowerShell files:

```text
data.win.eventdata.targetFilename:*\.ps1
```

Executable:

```text
data.win.eventdata.targetFilename:*\.exe
```

Temporary paths:

```text
data.win.eventdata.targetFilename:*\\Temp\\*
```

---

# 169. Quick Reference — Process Access

Sysmon Event 10:

```text
data.win.system.providerName:"Microsoft-Windows-Sysmon" AND data.win.system.eventID:"10"
```

Source process:

```text
data.win.eventdata.sourceImage:*OneDrive.exe
```

Target process:

```text
data.win.eventdata.targetImage:*Explorer.EXE
```

Access mask:

```text
data.win.eventdata.grantedAccess:"0x101411"
```

---

# 170. Quick Reference — Rules

Rule ID:

```text
rule.id:"92910"
```

Severity:

```text
rule.level:12
```

MITRE:

```text
rule.mitre.id:"T1059.001"
```

Host:

```text
agent.name:"BTL-WIN11"
```

---

# 171. Recommended Search Order

For an unknown alert:

```text
Host
 ↓
Time
 ↓
Rule
 ↓
Event ID
 ↓
User
 ↓
Process
 ↓
ProcessGuid
 ↓
Files / DNS / Network
```

For a known process:

```text
Process
 ↓
Parent
 ↓
Command Line
 ↓
ProcessGuid
 ↓
DNS
 ↓
Network
 ↓
Files
```

---

# 172. Wazuh Investigation Notes Template

```text
Alert:
<rule description>

Rule ID:
<rule id>

Severity:
<level>

Host:
<agent>

Timestamp:
<time>

Event ID:
<event>

User:
<user>

Process:
<image>

Parent:
<parent>

Command:
<command line>

ProcessGuid:
<guid>

Files:
<files>

DNS:
<domains>

Network:
<connections>

Initial Hypothesis:
<hypothesis>

Evidence:
<evidence>

Verdict:
<verdict>
```

---

# 173. Current Lab Searches Worth Practising

Practice these regularly:

```text
agent.name:"BTL-WIN11"
```

```text
rule.id:"92910"
```

```text
rule.id:"92205"
```

```text
data.win.eventdata.image:*powershell.exe
```

```text
data.win.eventdata.sourceImage:*OneDrive.exe
```

```text
data.win.eventdata.targetImage:*Explorer.EXE
```

```text
data.win.system.eventID:"4625"
```

```text
data.win.system.eventID:"4624"
```

```text
data.win.system.providerName:"Microsoft-Windows-Sysmon" AND data.win.system.eventID:"1"
```

```text
data.win.system.providerName:"Microsoft-Windows-Sysmon" AND data.win.system.eventID:"22"
```

---

# 174. What to Learn First

Prioritise:

```text
1. Search by host

2. Search by Rule ID

3. Search by Event ID

4. Search by process

5. Search by user

6. Search by ProcessGuid

7. Search by filename

8. Search by DNS domain

9. Search by destination IP

10. Correlate by time
```

---

# 175. What to Learn Next

Then practise:

```text
MITRE filters
Rule groups
Authentication fields
Parent-child process hunting
Network correlation
ProcessAccess analysis
File creation patterns
Custom Wazuh rules
Detection tuning
```

---

# 176. Core Investigation Principle

Wazuh searching is not about finding one suspicious string.

It is about moving from:

```text
Alert
```

to:

```text
Context
```

A good analyst search sequence looks like:

```text
Alert
  ↓
Host
  ↓
User
  ↓
Process
  ↓
Parent
  ↓
Command Line
  ↓
ProcessGuid
  ↓
DNS
  ↓
Network
  ↓
Files
  ↓
Timeline
  ↓
Verdict
```

---

# 177. Key Takeaway

Do not think:

```text
What query should I memorise?
```

Think:

```text
What question am I trying to answer?
```

Examples:

```text
Which host generated the alert?
→ agent.name
```

```text
Why did Wazuh alert?
→ rule.id / rule.description
```

```text
What happened on Windows?
→ data.win.system.eventID
```

```text
Which process was involved?
→ image / sourceImage / targetImage
```

```text
What did the process execute?
→ commandLine
```

```text
What else did the same process do?
→ processGuid
```

```text
Did it query a domain?
→ queryName
```

```text
Did it connect somewhere?
→ destinationIp / destinationPort
```

```text
Did it create a file?
→ targetFilename
```

The most important Wazuh skill is not searching.

It is knowing how to pivot from one piece of evidence to the next until the activity makes sense.
