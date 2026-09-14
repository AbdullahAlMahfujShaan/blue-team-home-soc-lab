# Windows Event ID Cheat Sheet for Blue Team / SOC Analysis

This cheat sheet focuses on Windows Event IDs that are especially useful for:

- SOC analysis
- Incident response
- Authentication investigations
- Privilege escalation investigations
- Persistence detection
- Account activity
- Process execution
- Lateral movement
- Threat hunting

The goal is not to memorise every Windows Event ID.

The goal is to understand:

```text
What happened?
Which Event ID records it?
Which fields matter?
What should I investigate next?
```

---

# 1. Windows Event Log Locations

Common logs include:

```text
Security
System
Application
Microsoft-Windows-PowerShell/Operational
Microsoft-Windows-Sysmon/Operational
Microsoft-Windows-TaskScheduler/Operational
Microsoft-Windows-Windows Defender/Operational
```

For many SOC investigations, the most important starting point is:

```text
Security
```

---

# 2. View Events with PowerShell

Recent Security events:

```powershell
Get-WinEvent -LogName Security -MaxEvents 20
```

Filter by Event ID:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4625
} -MaxEvents 20
```

---

# 3. High-Priority Event IDs

Start with these:

| Event ID | Meaning |
|---|---|
| `4624` | Successful logon |
| `4625` | Failed logon |
| `4634` | Account logged off |
| `4648` | Logon using explicit credentials |
| `4672` | Special privileges assigned |
| `4688` | New process created |
| `4697` | Service installed |
| `4698` | Scheduled task created |
| `4702` | Scheduled task updated |
| `4720` | User account created |
| `4722` | User account enabled |
| `4724` | Password reset attempt |
| `4725` | User account disabled |
| `4726` | User account deleted |
| `4728` | User added to global security group |
| `4732` | User added to local security group |
| `4738` | User account changed |
| `4740` | Account locked out |
| `4768` | Kerberos TGT requested |
| `4769` | Kerberos service ticket requested |
| `4771` | Kerberos pre-authentication failed |
| `4776` | Credential validation |
| `1102` | Audit log cleared |

---

# 4. Event ID 4624 — Successful Logon

## What It Means

```text
An account successfully logged on.
```

This is one of the most important Windows Security events.

---

## Important Fields

Look for:

```text
SubjectUserName
TargetUserName
TargetDomainName
LogonType
WorkstationName
IpAddress
IpPort
AuthenticationPackageName
LogonProcessName
LogonId
```

---

## Key Questions

```text
Who logged in?

From where?

What type of logon?

Was the account expected?

Was the source IP expected?

Was the authentication method normal?
```

---

# 5. Common Logon Types

| Logon Type | Meaning |
|---|---|
| `2` | Interactive |
| `3` | Network |
| `4` | Batch |
| `5` | Service |
| `7` | Unlock |
| `8` | Network cleartext |
| `9` | New credentials |
| `10` | Remote Interactive / RDP |
| `11` | Cached Interactive |

---

# 6. Why Logon Type Matters

A 4624 by itself only tells you:

```text
A login succeeded.
```

The logon type tells you how.

Example:

```text
4624
+
Logon Type 2
```

usually means:

```text
Local interactive login
```

Example:

```text
4624
+
Logon Type 10
```

usually means:

```text
Remote Desktop login
```

---

# 7. Event ID 4625 — Failed Logon

## What It Means

```text
An account failed to log on.
```

This is critical for:

- Brute-force investigation
- Password spraying
- Mistyped password analysis
- RDP attack investigation
- Service authentication issues

---

## Important Fields

```text
TargetUserName
TargetDomainName
FailureReason
Status
SubStatus
LogonType
WorkstationName
IpAddress
IpPort
AuthenticationPackageName
```

---

## Questions

```text
Which account failed?

How many times?

From which IP?

Which logon type?

What was the failure reason?

Did a successful 4624 happen afterward?
```

---

# 8. Single Failure vs Brute Force

One event:

```text
4625
```

may simply be:

```text
User typed the wrong password.
```

A pattern such as:

```text
4625
4625
4625
4625
4625
4624
```

from the same source may deserve much more attention.

---

# 9. Failed Logon PowerShell Query

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4625
    StartTime = (Get-Date).AddHours(-1)
}
```

---

# 10. Event ID 4634 — Account Logged Off

## What It Means

```text
A logon session ended.
```

Useful for:

- Session timeline
- Determining logon duration
- Correlating activity to user sessions

---

# 11. Event ID 4647 — User Initiated Logoff

## What It Means

```text
A user explicitly initiated a logoff.
```

This differs from a generic session termination.

---

# 12. Event ID 4648 — Explicit Credentials Used

## What It Means

```text
A process attempted to log on using explicitly supplied credentials.
```

This may occur during:

- `runas`
- Administrative tools
- Remote management
- Credential use by applications

---

## Important Fields

```text
SubjectUserName
AccountWhoseCredentialsWereUsed
TargetServerName
TargetInfo
ProcessName
IpAddress
```

---

## Why It Matters

This event may help identify:

```text
Credential use
Lateral movement
Administrative activity
```

But it can also be completely legitimate.

---

# 13. Event ID 4672 — Special Privileges Assigned

## What It Means

```text
Sensitive privileges were assigned to a new logon.
```

Often generated for privileged accounts such as administrators.

---

## Common Privileges

Examples may include:

```text
SeDebugPrivilege
SeBackupPrivilege
SeRestorePrivilege
SeTakeOwnershipPrivilege
```

---

## Questions

```text
Which account received the privileges?

Was the account expected to be privileged?

What happened immediately afterward?
```

---

# 14. Event ID 4688 — Process Creation

## What It Means

```text
A new process was created.
```

This is highly valuable if process creation auditing is enabled.

Sysmon Event ID 1 usually provides richer process telemetry, but 4688 remains important.

---

## Important Fields

```text
NewProcessId
NewProcessName
CreatorProcessId
CommandLine
SubjectUserName
TokenElevationType
```

Command line visibility depends on audit configuration.

---

## Questions

```text
What executed?

Who executed it?

What launched it?

What command line was used?

Was the path expected?

Was the process elevated?
```

---

# 15. Process Creation Chain

Example:

```text
winword.exe
     ↓
powershell.exe
     ↓
cmd.exe
```

The relationship may matter more than any one process name.

---

# 16. Event ID 4689 — Process Terminated

## What It Means

```text
A process exited.
```

Useful for:

- Process lifetime analysis
- Timeline reconstruction

---

# 17. Event ID 4697 — Service Installed

## What It Means

```text
A new service was installed.
```

Services can be used for:

- Legitimate software
- Administrative management
- Persistence
- Privilege escalation
- Remote execution

---

## Important Fields

```text
ServiceName
ServiceFileName
ServiceType
ServiceStartType
ServiceAccount
SubjectUserName
```

---

## Questions

```text
Who installed the service?

What executable does it run?

Where is that executable located?

Is the service expected?

Did it appear during suspicious activity?
```

---

# 18. Event ID 4698 — Scheduled Task Created

## What It Means

```text
A scheduled task was created.
```

Scheduled tasks are a common administrative mechanism and a common persistence technique.

---

## Questions

```text
Who created the task?

What command does it execute?

When does it execute?

Is the executable path suspicious?
```

---

# 19. Event ID 4699 — Scheduled Task Deleted

```text
A scheduled task was deleted.
```

May be relevant when investigating cleanup or persistence changes.

---

# 20. Event ID 4700 — Scheduled Task Enabled

```text
A scheduled task was enabled.
```

---

# 21. Event ID 4701 — Scheduled Task Disabled

```text
A scheduled task was disabled.
```

---

# 22. Event ID 4702 — Scheduled Task Updated

## What It Means

```text
An existing scheduled task was modified.
```

A legitimate task modified unexpectedly may deserve investigation.

---

# 23. Event ID 4719 — Audit Policy Changed

## What It Means

```text
System audit policy was modified.
```

This can be security relevant because attackers may attempt to reduce logging.

---

## Questions

```text
Who changed the policy?

Which audit category changed?

Was the change authorised?
```

---

# 24. Event ID 4720 — User Account Created

## What It Means

```text
A new user account was created.
```

---

## Important Fields

```text
SubjectUserName
TargetUserName
TargetDomainName
SamAccountName
UserPrincipalName
```

---

## Questions

```text
Who created the account?

Was account creation expected?

Was the account later added to privileged groups?
```

---

# 25. Event ID 4722 — User Account Enabled

```text
A previously disabled user account was enabled.
```

Useful during account manipulation investigations.

---

# 26. Event ID 4723 — Password Change Attempt

```text
An attempt was made to change an account password.
```

Usually refers to a user changing their own password.

---

# 27. Event ID 4724 — Password Reset Attempt

## What It Means

```text
An attempt was made to reset another account's password.
```

Potentially sensitive because administrators commonly perform this action.

---

## Questions

```text
Who reset the password?

Which account was affected?

Was the reset authorised?
```

---

# 28. Event ID 4725 — User Account Disabled

```text
A user account was disabled.
```

May be normal administration or part of incident containment.

---

# 29. Event ID 4726 — User Account Deleted

```text
A user account was deleted.
```

Important when investigating account lifecycle activity.

---

# 30. Event ID 4728 — Member Added to Global Security Group

## What It Means

```text
A user was added to a global security group.
```

In Active Directory environments this may be highly important.

---

# 31. Event ID 4729 — Member Removed from Global Security Group

```text
A user was removed from a global security group.
```

---

# 32. Event ID 4732 — Member Added to Local Security Group

## What It Means

```text
A user was added to a local security group.
```

This is especially interesting for:

```text
Administrators
Remote Desktop Users
```

---

## Questions

```text
Who added the member?

Which account was added?

Which group?

Was the change expected?
```

---

# 33. Event ID 4733 — Member Removed from Local Group

```text
A user was removed from a local security group.
```

---

# 34. Event ID 4738 — User Account Changed

## What It Means

```text
Attributes of a user account were modified.
```

Possible changes include:

- Account name
- Password settings
- Profile information
- User account control properties

---

# 35. Event ID 4740 — Account Locked Out

## What It Means

```text
A user account was locked because of failed authentication attempts.
```

This is highly useful in brute-force and password-spray investigations.

---

## Important Fields

```text
TargetUserName
CallerComputerName
```

---

## Investigation Pattern

```text
Multiple 4625
     ↓
4740
```

This may indicate repeated failed authentication.

---

# 36. Event ID 4768 — Kerberos TGT Requested

## What It Means

```text
A Kerberos Ticket Granting Ticket was requested.
```

This event generally appears on domain controllers.

---

## Important Fields

```text
TargetUserName
IpAddress
TicketEncryptionType
Status
PreAuthType
```

---

# 37. Event ID 4769 — Kerberos Service Ticket Requested

## What It Means

```text
A Kerberos service ticket was requested.
```

Useful for:

- Kerberos investigation
- Service access
- Lateral movement analysis
- Some Kerberoasting investigations

---

## Important Fields

```text
TargetUserName
ServiceName
IpAddress
TicketEncryptionType
TicketOptions
```

---

# 38. Event ID 4770 — Kerberos Service Ticket Renewed

```text
A Kerberos service ticket was renewed.
```

---

# 39. Event ID 4771 — Kerberos Pre-Authentication Failed

## What It Means

```text
Kerberos authentication failed before a TGT was issued.
```

This can occur because of:

```text
Bad password
Expired password
Account restrictions
Clock issues
```

---

## Questions

```text
Which account failed?

From which IP?

How often?

Was there a later successful 4768?
```

---

# 40. Event ID 4776 — Credential Validation

## What It Means

```text
A computer attempted to validate account credentials.
```

Often associated with NTLM authentication.

---

## Important Fields

```text
LogonAccount
SourceWorkstation
ErrorCode
```

---

# 41. Event ID 1102 — Audit Log Cleared

## What It Means

```text
The Windows Security audit log was cleared.
```

This deserves investigation unless there is a known administrative reason.

---

## Why It Matters

Attackers may clear logs to reduce evidence.

But administrators and testing may also clear logs.

---

## Questions

```text
Who cleared the log?

When?

What occurred immediately before?

Was the action authorised?
```

---

# 42. Event ID 104 — System Log Cleared

Within the System log, Event ID:

```text
104
```

can indicate a log file was cleared.

Again, context matters.

---

# 43. RDP-Related Events

For Remote Desktop investigation, useful events may come from several logs.

Security log:

```text
4624 — Successful logon
4625 — Failed logon
```

Watch for:

```text
Logon Type 10
```

---

# 44. Remote Desktop Services Events

Useful log:

```text
Microsoft-Windows-TerminalServices-LocalSessionManager/Operational
```

Common events can include:

```text
21 — Session logon succeeded
23 — Session logoff
24 — Session disconnected
25 — Session reconnected
```

Exact usefulness depends on OS/version and logging configuration.

---

# 45. PowerShell Event ID 4103

Log:

```text
Microsoft-Windows-PowerShell/Operational
```

Event:

```text
4103
```

May contain:

```text
Module logging
Command execution details
```

when configured.

---

# 46. PowerShell Event ID 4104

## What It Means

```text
PowerShell Script Block Logging
```

when enabled.

This is one of the most useful PowerShell-related events.

---

## Useful For

```text
Decoded PowerShell code
Executed script blocks
Suspicious commands
Obfuscated PowerShell
```

---

## Query

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id      = 4104
} -MaxEvents 20
```

---

# 47. Defender Event Log

Log:

```text
Microsoft-Windows-Windows Defender/Operational
```

Useful for:

- Malware detections
- Defender actions
- Scan results
- Configuration changes

---

# 48. Defender Event ID 1116

Common Defender event:

```text
1116
```

typically indicates malware or potentially unwanted software was detected.

---

# 49. Defender Event ID 1117

Commonly associated with:

```text
Action taken against detected malware
```

Always review the complete event message and current Microsoft documentation if interpreting production data.

---

# 50. Account Investigation Chain

A useful pattern:

```text
4720
Account Created
      ↓
4732
Added to Administrators
      ↓
4624
Successful Logon
      ↓
4688
Process Execution
```

This sequence would deserve investigation if unexpected.

---

# 51. Authentication Investigation Chain

```text
4625
Failed Login
      ↓
4625
Failed Login
      ↓
4625
Failed Login
      ↓
4740
Account Locked
```

Potential interpretation:

```text
Repeated authentication failures
```

---

# 52. Brute-Force Success Pattern

```text
4625
4625
4625
4625
4624
```

Questions:

```text
Same source IP?

Same username?

Same logon type?

How quickly did attempts occur?

Was the successful login followed by suspicious behaviour?
```

---

# 53. Privileged Login Pattern

```text
4624
Successful Login
      ↓
4672
Special Privileges
```

This may simply indicate an administrator logged in.

Ask:

```text
Was this account expected to have these privileges?
```

---

# 54. Persistence Pattern — New Service

```text
4624
Login
   ↓
4688
Process
   ↓
4697
Service Installed
```

Then investigate:

```text
Service executable path
Service account
Process ancestry
Network activity
```

---

# 55. Persistence Pattern — Scheduled Task

```text
4688
Process Execution
      ↓
4698
Scheduled Task Created
```

Then check:

```text
Task name
Task action
Executable path
Trigger
User
```

---

# 56. New Account + Privilege Pattern

```text
4720
New Account
      ↓
4732
Added to Local Group
```

If the group is:

```text
Administrators
```

this is especially important.

---

# 57. Windows Security + Sysmon Correlation

Windows Security logs answer:

```text
Who authenticated?

Which account changed?

Was privilege assigned?

Was a process created?
```

Sysmon can provide:

```text
Detailed process information
DNS activity
Network connections
File creation
Process access
DLL loading
```

Together:

```text
Identity
+
Endpoint behaviour
```

---

# 58. Example Timeline

```text
09:00:01 — 4625 Failed login
09:00:05 — 4625 Failed login
09:00:09 — 4625 Failed login
09:00:15 — 4624 Successful login
09:00:15 — 4672 Privileged logon
09:01:02 — Sysmon 1 PowerShell started
09:01:05 — Sysmon 22 DNS query
09:01:06 — Sysmon 3 Network connection
```

This provides significantly more context than any one event.

---

# 59. Event IDs Are Not Verdicts

Example:

```text
4625
```

does not mean:

```text
Brute-force attack
```

It means:

```text
A login failed.
```

Similarly:

```text
4688 powershell.exe
```

does not mean:

```text
Malware executed.
```

It means:

```text
PowerShell started.
```

Context determines meaning.

---

# 60. Severity vs Evidence

Do not think:

```text
Interesting Event ID
      =
Incident
```

Think:

```text
Event
  ↓
Context
  ↓
Correlation
  ↓
Timeline
  ↓
Verdict
```

---

# 61. Time Is Critical

Always check:

```text
TimeCreated
```

Then look:

```text
Before
During
After
```

Example:

```text
What happened 5 minutes before the login?

What happened immediately after?

Were processes launched?

Were files created?

Did network activity begin?
```

---

# 62. Query Multiple Event IDs

Example:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4624,4625,4672
    StartTime = (Get-Date).AddHours(-2)
} |
Sort-Object TimeCreated |
Select-Object TimeCreated, Id, Message
```

Useful for authentication timelines.

---

# 63. Find Recent Account Changes

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4720,4722,4724,4725,4726,4732,4738
    StartTime = (Get-Date).AddDays(-1)
}
```

---

# 64. Find Recent Persistence Events

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4697,4698,4702
    StartTime = (Get-Date).AddDays(-1)
}
```

---

# 65. Find Failed Logons

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4625
} -MaxEvents 50 |
Select-Object TimeCreated, Id, Message
```

---

# 66. Find Successful Logons

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4624
} -MaxEvents 50 |
Select-Object TimeCreated, Id, Message
```

---

# 67. Most Important Fields for Authentication

Always look for:

```text
TargetUserName
TargetDomainName
LogonType
IpAddress
WorkstationName
AuthenticationPackageName
FailureReason
Status
SubStatus
```

---

# 68. Status and SubStatus

Failed login events can contain:

```text
Status
SubStatus
```

These can provide more specific failure reasons.

Do not memorise every hexadecimal value initially.

Use them when needed to distinguish issues such as:

```text
Bad password
Unknown account
Disabled account
Locked account
Expired password
```

---

# 69. Authentication Package

Common examples:

```text
Kerberos
NTLM
Negotiate
```

This can help explain how authentication occurred.

---

# 70. Workstation and Source IP

Compare:

```text
WorkstationName
IpAddress
```

These fields help answer:

```text
Where did the authentication originate?
```

Be aware that some events may contain:

```text
-
127.0.0.1
::1
```

depending on the authentication flow.

---

# 71. Important Active Directory Events

For future domain lab work, prioritise:

```text
4720
4728
4732
4740
4768
4769
4771
4776
```

These provide strong coverage of:

```text
Accounts
Groups
Lockouts
Kerberos
Credential validation
```

---

# 72. Domain Controller vs Endpoint

Not every Event ID appears on every machine.

For example:

```text
4768 / 4769
```

are typically most relevant on:

```text
Domain Controllers
```

while local endpoint authentication may be better understood through:

```text
4624
4625
```

Always know which system generated the event.

---

# 73. Event Source Matters

Ask:

```text
Was this event generated on:

Endpoint?
Domain Controller?
Server?
Jump host?
```

The same activity may appear differently across systems.

---

# 74. Quick Reference — Authentication

| Event ID | Meaning |
|---|---|
| `4624` | Successful logon |
| `4625` | Failed logon |
| `4634` | Logoff |
| `4647` | User initiated logoff |
| `4648` | Explicit credentials |
| `4672` | Special privileges |
| `4740` | Account lockout |
| `4768` | Kerberos TGT |
| `4769` | Kerberos service ticket |
| `4771` | Kerberos pre-auth failed |
| `4776` | Credential validation |

---

# 75. Quick Reference — Accounts

| Event ID | Meaning |
|---|---|
| `4720` | Account created |
| `4722` | Account enabled |
| `4723` | Password change attempt |
| `4724` | Password reset attempt |
| `4725` | Account disabled |
| `4726` | Account deleted |
| `4738` | Account changed |
| `4740` | Account locked |

---

# 76. Quick Reference — Groups

| Event ID | Meaning |
|---|---|
| `4728` | Added to global group |
| `4729` | Removed from global group |
| `4732` | Added to local group |
| `4733` | Removed from local group |

---

# 77. Quick Reference — Execution / Persistence

| Event ID | Meaning |
|---|---|
| `4688` | Process created |
| `4689` | Process terminated |
| `4697` | Service installed |
| `4698` | Scheduled task created |
| `4699` | Scheduled task deleted |
| `4700` | Scheduled task enabled |
| `4701` | Scheduled task disabled |
| `4702` | Scheduled task updated |

---

# 78. Quick Reference — Logging / Defense Evasion

| Event ID | Meaning |
|---|---|
| `4719` | Audit policy changed |
| `1102` | Security audit log cleared |
| `104` | System log cleared |

---

# 79. Quick Reference — PowerShell

| Event ID | Meaning |
|---|---|
| `4103` | PowerShell module logging |
| `4104` | Script Block Logging |

These require appropriate PowerShell logging configuration.

---

# 80. Investigation Workflow

When a Windows event looks suspicious:

```text
Identify Event ID
      ↓
Understand What It Means
      ↓
Check Host
      ↓
Check Timestamp
      ↓
Check User
      ↓
Check Source IP
      ↓
Check Logon Type / Context
      ↓
Review Nearby Events
      ↓
Pivot to Sysmon
      ↓
Check Process / DNS / Network / Files
      ↓
Build Timeline
      ↓
Reach Verdict
```

---

# 81. Failed Login Investigation Workflow

```text
4625 Found
    ↓
Identify Username
    ↓
Identify Source IP
    ↓
Identify Logon Type
    ↓
Count Failures
    ↓
Check Time Pattern
    ↓
Search for 4624
    ↓
Check 4740
    ↓
Review Post-Login Activity
```

---

# 82. Successful Login Investigation Workflow

```text
4624 Found
    ↓
User
    ↓
Logon Type
    ↓
Source IP
    ↓
Authentication Method
    ↓
4672?
    ↓
Processes Started
    ↓
Network Activity
```

---

# 83. Account Creation Investigation Workflow

```text
4720
    ↓
Who created account?
    ↓
What account?
    ↓
4732 / 4728?
    ↓
Privilege added?
    ↓
4624?
    ↓
Did the new account log in?
    ↓
What did it do?
```

---

# 84. Service Installation Investigation Workflow

```text
4697
    ↓
Service Name
    ↓
Executable Path
    ↓
Service Account
    ↓
Who installed it?
    ↓
Process telemetry
    ↓
Hash / Signature
    ↓
Network activity
```

---

# 85. Scheduled Task Investigation Workflow

```text
4698 / 4702
      ↓
Task Name
      ↓
Task Action
      ↓
Executable / Script
      ↓
User Context
      ↓
Trigger
      ↓
Process / File / Network Activity
```

---

# 86. Common Analyst Mistakes

Do not assume:

```text
4625 = attack
```

Do not assume:

```text
4624 = normal
```

Do not assume:

```text
4672 = malicious privilege escalation
```

Do not assume:

```text
PowerShell = malware
```

Do not analyse one event without checking nearby telemetry.

---

# 87. Contextual Example

Consider:

```text
4624
TargetUserName: labadmin
LogonType: 2
Source: local workstation
```

In a lab where `labadmin` intentionally signed in locally:

```text
Likely normal.
```

But:

```text
4624
TargetUserName: Administrator
LogonType: 10
Source IP: unknown external source
Time: 03:17 AM
```

would deserve far more investigation.

Same event ID.

Different context.

---

# 88. Event ID Learning Priority

For current study, learn in this order:

```text
4624
↓
4625
↓
4672
↓
4688
↓
4720
↓
4732
↓
4740
↓
4697
↓
4698
↓
4768
↓
4769
↓
4771
↓
1102
```

---

# 89. Core Mental Model

Think of Windows Event IDs by category rather than memorising a random list.

```text
AUTHENTICATION
4624 / 4625 / 4648 / 4740

PRIVILEGE
4672

PROCESS
4688

ACCOUNTS
4720–4740

PERSISTENCE
4697 / 4698 / 4702

KERBEROS
4768 / 4769 / 4771

LOG TAMPERING
1102
```

This makes them much easier to remember.

---

# 90. Windows + Sysmon Mental Model

Windows Security logs:

```text
WHO?
```

Sysmon:

```text
WHAT DID THEY DO?
```

Example:

```text
4624
User logged in
      ↓
Sysmon 1
PowerShell started
      ↓
Sysmon 22
Domain queried
      ↓
Sysmon 3
Outbound connection
      ↓
Sysmon 11
File created
```

This is how you build a real investigation timeline.

---

# 91. Key Takeaway

Do not memorise Windows Event IDs as isolated numbers.

Think:

```text
Who authenticated?
4624 / 4625

Was privilege assigned?
4672

What executed?
4688

Was an account created?
4720

Was privilege added?
4732

Was persistence created?
4697 / 4698

Was Kerberos used?
4768 / 4769

Were logs cleared?
1102
```

The Event ID tells you:

```text
What happened.
```

Your job as the analyst is to determine:

```text
Why it happened,
whether it was expected,
what happened before and after,
and whether the evidence supports an incident.
```
