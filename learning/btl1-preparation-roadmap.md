# BTL1 Preparation Roadmap

This roadmap documents my preparation plan for the **Security Blue Team Level 1 (BTL1)** certification and, more importantly, the practical skills required for a junior SOC / Blue Team role.

The focus is not on memorising answers.

The goal is to build a repeatable investigation mindset:

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
   ↓
Incident Response
```

---

# 1. Preparation Goal

The main objectives are to become comfortable with:

- SOC workflows
- Incident response
- Windows Event Logs
- Sysmon
- SIEM investigation
- Splunk and SPL
- Wazuh
- Phishing analysis
- Threat intelligence
- Digital forensics
- Network traffic analysis
- Wireshark
- MITRE ATT&CK
- Linux command-line investigation
- PowerShell
- Evidence correlation
- Investigation reporting

The target is not simply:

```text
Pass BTL1
```

The larger goal is:

```text
Develop practical junior SOC analyst skills
```

---

# 2. Current Lab Environment

The home SOC lab currently includes:

```text
Apple Silicon Mac
      │
      ▼
VMware Fusion
      │
      ├── BTL-WIN11
      │      ├── Windows 11 Pro ARM64
      │      ├── Sysmon
      │      └── Wazuh Agent
      │
      ├── BTL-WAZUH
      │      ├── Ubuntu Server ARM64
      │      ├── Wazuh Manager
      │      ├── Wazuh Indexer
      │      └── Wazuh Dashboard
      │
      └── BTL-KALI
             └── Kali Linux ARM64
```

This lab is used to practise:

- Endpoint telemetry
- Authentication events
- Process events
- DNS events
- Network connections
- SIEM alert triage
- Threat hunting
- Investigation reporting

---

# 3. Study Philosophy

The preparation process follows:

```text
Learn
  ↓
See
  ↓
Do
  ↓
Investigate
  ↓
Document
  ↓
Recall
```

Each topic should include both:

```text
Theory
+
Hands-On Practice
```

Reading alone is not enough.

---

# 4. Recommended Study Session

A normal study session can follow:

```text
20–30 min   Theory
20–30 min   Guided explanation
20–30 min   Practical lab
5–10 min    Notes / recall
```

Longer practical sessions can be used for full investigations.

For example:

```text
90–120 min Investigation Lab
```

---

# 5. Core Investigation Mindset

For every alert, ask:

```text
What happened?

How was it detected?

Which host is affected?

Which user is involved?

Which process is involved?

What happened before it?

What happened after it?

Is there related network activity?

Is there related DNS activity?

Are files involved?

Is this normal behaviour?

What evidence supports my conclusion?

What should happen next?
```

---

# 6. Phase 1 — SOC and Incident Response Fundamentals

## Objective

Understand how a Security Operations Center operates and how an alert progresses into an investigation.

Topics:

- SOC responsibilities
- Tier 1 / Tier 2 analyst roles
- Alert triage
- Escalation
- Incident response lifecycle
- Evidence preservation
- False positives
- Benign positives
- Severity vs confidence
- Incident prioritisation

---

## Core Model

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
Classification
    ↓
Response
```

---

## Learn the Difference Between

```text
Event
Alert
Detection
Incident
IOC
IOA
False Positive
Benign Positive
True Positive
```

---

## Practical Goal

Be able to explain:

> Why is an alert not automatically an incident?

---

# 7. Incident Response Lifecycle

A useful model is:

```text
Preparation
    ↓
Detection & Analysis
    ↓
Containment
    ↓
Eradication
    ↓
Recovery
    ↓
Lessons Learned
```

Understand what happens during each stage.

---

# 8. Phase 2 — Windows Security Fundamentals

Windows telemetry is one of the highest-priority areas.

Learn:

- Windows users
- Local administrators
- Services
- Processes
- Registry
- Scheduled Tasks
- Event Viewer
- Security logs
- PowerShell logs
- Windows Defender basics

---

# 9. Important Windows Event IDs

Start with:

| Event ID | Meaning |
|---|---|
| `4624` | Successful logon |
| `4625` | Failed logon |
| `4634` | Account logged off |
| `4648` | Logon using explicit credentials |
| `4672` | Special privileges assigned |
| `4688` | Process creation |
| `4697` | Service installed |
| `4720` | User account created |
| `4722` | User account enabled |
| `4724` | Password reset attempt |
| `4728` | Member added to global security group |
| `4732` | Member added to local security group |
| `4740` | Account locked out |
| `4768` | Kerberos TGT requested |
| `4769` | Kerberos service ticket requested |
| `4771` | Kerberos pre-authentication failed |
| `4776` | Credential validation |

Do not attempt to memorise every Windows Event ID immediately.

Focus first on understanding the common ones.

---

# 10. Windows Logon Types

Important logon types include:

| Type | Meaning |
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

Example:

```text
4625
+
Logon Type 10
```

may represent a failed RDP authentication attempt.

---

# 11. Windows Practical Exercises

Practise:

```text
Successful login
Failed login
Account lockout
Process execution
PowerShell execution
New user creation
Service creation
Network connection
DNS query
```

Document what appears in the logs.

---

# 12. Phase 3 — Sysmon

Sysmon provides richer endpoint telemetry than native Windows logging alone.

Important Event IDs:

| Sysmon Event ID | Meaning |
|---|---|
| `1` | Process Creation |
| `3` | Network Connection |
| `7` | Image Loaded |
| `10` | Process Access |
| `11` | File Create |
| `22` | DNS Query |

---

# 13. Sysmon Event ID 1 — Process Creation

Key fields:

```text
Image
CommandLine
ProcessId
ProcessGuid
ParentImage
ParentCommandLine
User
IntegrityLevel
Hashes
```

Questions:

```text
What executed?

Who executed it?

What command line was used?

What launched it?

Where did it execute from?
```

---

# 14. Process Tree Thinking

Instead of looking at a process alone, think:

```text
Parent Process
      │
      ▼
Child Process
      │
      ▼
Additional Child Process
```

Example:

```text
explorer.exe
     ↓
powershell.exe
     ↓
curl.exe
```

may deserve more scrutiny than:

```text
explorer.exe
     ↓
notepad.exe
```

---

# 15. Sysmon Event ID 3 — Network Connection

Important fields:

```text
Image
User
Protocol
SourceIp
SourcePort
DestinationIp
DestinationPort
DestinationHostname
```

Ask:

```text
Which process made the connection?

Where did it connect?

Which port was used?

Is that destination expected?
```

---

# 16. Sysmon Event ID 10 — Process Access

This event became important during the first lab investigation.

Example:

```text
OneDrive.exe
     ↓
Explorer.EXE
```

Process access can be legitimate or suspicious.

Always investigate context.

---

# 17. Sysmon Event ID 22 — DNS Query

DNS telemetry helps identify:

- Command-and-control domains
- Phishing infrastructure
- Malware downloads
- Suspicious subdomains
- Unexpected application behaviour

Ask:

```text
Which process generated the query?

What domain was requested?

Was the domain expected?

What happened immediately afterward?
```

---

# 18. Phase 4 — SIEM Fundamentals

Understand what a SIEM does:

```text
Collect
   ↓
Normalize
   ↓
Index
   ↓
Search
   ↓
Detect
   ↓
Alert
```

Know the difference between:

```text
Raw telemetry
Detection rule
Alert
Dashboard
Investigation
```

---

# 19. Wazuh Practice

The persistent home SIEM is Wazuh.

Use it to practise:

- Agent monitoring
- Sysmon integration
- Alert triage
- Event searches
- Rule interpretation
- Windows authentication investigations
- Process investigations
- Network events
- DNS events

---

# 20. Important Wazuh Lesson

The lab has already demonstrated:

```text
Raw Event ≠ Wazuh Alert
```

A Sysmon event may exist locally without appearing in:

```text
wazuh-alerts-*
```

because not every raw event triggers a detection rule.

This is a critical SIEM concept.

---

# 21. Phase 5 — Splunk and SPL

Splunk is an important part of the preparation plan.

Because the current host uses Apple Silicon ARM64, the persistent home SOC uses Wazuh while Splunk can be practised through supported hosted or training environments.

The goal is to become comfortable with SPL investigation workflows.

---

# 22. SPL Fundamentals

Start with:

```spl
index=*
```

Then:

```spl
index=* host="WIN11"
```

Failed logons:

```spl
index=* EventCode=4625
```

Count by user and source:

```spl
index=* EventCode=4625
| stats count by user, src_ip
| sort -count
```

---

# 23. Process Investigation in SPL

Example:

```spl
index=* EventCode=4688
| table _time host user New_Process_Name CommandLine
```

Questions:

```text
Which process executed?

When?

On which host?

Under which user?

What command line was used?
```

---

# 24. Network Hunting in SPL

Example:

```spl
index=*
| stats count by src_ip
| sort -count
| head 20
```

This introduces aggregation and frequency analysis.

---

# 25. SPL Commands to Learn

Become comfortable with:

```text
search
table
fields
stats
count
values
dc
sort
dedup
rename
where
eval
rex
regex
lookup
transaction
timechart
top
rare
```

Do not simply memorise syntax.

Understand what analytical question each command answers.

---

# 26. SPL Learning Pattern

For every SPL command, document:

```text
Purpose
When to use it
Syntax
Example
Expected output
Next investigation pivot
```

Example:

```spl
... | stats count by src_ip
```

Purpose:

```text
Count how many events came from each source IP.
```

Use when:

```text
Looking for repeated authentication attempts or unusually active hosts.
```

---

# 27. Phase 6 — Network Fundamentals

Strong networking knowledge makes security investigations significantly easier.

Continue CCNA fundamentals alongside BTL1 preparation.

Topics:

- OSI model
- TCP/IP model
- Ethernet
- ARP
- IPv4
- IPv6
- Subnetting
- TCP
- UDP
- DNS
- DHCP
- NAT
- VLANs
- Routing
- Switching
- Common ports

---

# 28. Important Ports

Start with:

| Port | Protocol / Service |
|---|---|
| `20/21` | FTP |
| `22` | SSH |
| `23` | Telnet |
| `25` | SMTP |
| `53` | DNS |
| `67/68` | DHCP |
| `80` | HTTP |
| `88` | Kerberos |
| `110` | POP3 |
| `123` | NTP |
| `135` | RPC |
| `139` | NetBIOS |
| `143` | IMAP |
| `389` | LDAP |
| `443` | HTTPS |
| `445` | SMB |
| `636` | LDAPS |
| `3389` | RDP |

Learn the service, not just the number.

---

# 29. TCP vs UDP

Understand:

```text
TCP
- Connection-oriented
- Reliable delivery
- Sequencing
- Retransmission
- Handshake
```

versus:

```text
UDP
- Connectionless
- Lower overhead
- No delivery guarantee
- No sequencing guarantee
```

Know why this matters during packet analysis.

---

# 30. Phase 7 — Wireshark and PCAP Analysis

Learn how to investigate packet captures.

Start with:

- Interfaces
- Capture filters
- Display filters
- Conversation view
- Endpoints
- TCP streams
- DNS
- HTTP
- TLS metadata
- ICMP

---

# 31. Essential Wireshark Filters

DNS:

```text
dns
```

HTTP:

```text
http
```

TCP:

```text
tcp
```

UDP:

```text
udp
```

IP address:

```text
ip.addr == 192.168.1.10
```

Source:

```text
ip.src == 192.168.1.10
```

Destination:

```text
ip.dst == 192.168.1.10
```

TCP port:

```text
tcp.port == 443
```

DNS queries:

```text
dns.flags.response == 0
```

---

# 32. PCAP Investigation Workflow

Use:

```text
Identify Endpoints
      ↓
Identify Conversations
      ↓
Review DNS
      ↓
Review Protocols
      ↓
Inspect Suspicious Streams
      ↓
Identify Files / Commands / Domains
      ↓
Build Timeline
      ↓
Reach Verdict
```

---

# 33. Phase 8 — Phishing Analysis

Phishing is a major SOC skill.

Learn:

- Email headers
- Sender spoofing
- Display-name impersonation
- Reply-To
- Return-Path
- Received headers
- SPF
- DKIM
- DMARC
- URLs
- Attachments
- Social engineering indicators

---

# 34. Phishing Investigation Workflow

```text
Sender
  ↓
Headers
  ↓
Authentication Results
  ↓
Links
  ↓
Attachments
  ↓
Threat Intelligence
  ↓
User Impact
  ↓
Verdict
```

---

# 35. Questions for Every Suspicious Email

Ask:

```text
Who sent it?

Does the domain match?

Is Reply-To different?

Did SPF pass?

Did DKIM pass?

Did DMARC pass?

Where do the links actually go?

Are attachments suspicious?

What is the sender trying to make the user do?

Was anyone affected?
```

---

# 36. Phase 9 — Threat Intelligence

Threat intelligence helps enrich evidence.

Learn:

```text
IP reputation
Domain reputation
File hashes
URLs
WHOIS / registration data
Passive DNS concepts
Malware families
Campaigns
TTPs
```

---

# 37. IOC Types

Common indicators:

```text
IP Address
Domain
URL
File Hash
Filename
Email Address
Registry Key
Mutex
User-Agent
Certificate
```

---

# 38. IOC Limitation

Remember:

> An IOC is not automatically proof of compromise.

Example:

```text
An IP address may be shared infrastructure.
```

Always add context.

---

# 39. Phase 10 — MITRE ATT&CK

Use ATT&CK to describe attacker behaviour.

Understand:

```text
Tactic
   ↓
Technique
   ↓
Sub-Technique
```

Example:

```text
Execution
   ↓
Command and Scripting Interpreter
   ↓
PowerShell
```

Technique:

```text
T1059.001 — PowerShell
```

---

# 40. Common ATT&CK Areas to Recognise

Examples:

```text
Initial Access
Execution
Persistence
Privilege Escalation
Defense Evasion
Credential Access
Discovery
Lateral Movement
Collection
Command and Control
Exfiltration
Impact
```

Do not try to memorise every technique.

Focus on recognising behaviour.

---

# 41. Phase 11 — Digital Forensics Fundamentals

Learn the basics of:

- File systems
- Metadata
- Timestamps
- Hashes
- Deleted files
- Browser artifacts
- Windows artifacts
- Logs
- Memory concepts
- Evidence integrity

---

# 42. Hashing

Understand:

```text
MD5
SHA1
SHA256
```

For modern integrity and investigation work, SHA256 is commonly preferred.

Windows example:

```powershell
Get-FileHash .\sample.exe -Algorithm SHA256
```

Linux:

```bash
sha256sum sample.exe
```

---

# 43. Timestamp Awareness

Understand that:

```text
Created
Modified
Accessed
```

can represent different actions.

Also understand:

```text
System time
UTC
Local time
Timezone
```

Timeline mistakes can completely change an investigation.

---

# 44. Phase 12 — Linux Investigation Commands

Become comfortable with:

```bash
pwd
ls -la
cd
cat
less
head
tail
grep
find
file
stat
sha256sum
ps
top
ss
ip
curl
wget
journalctl
systemctl
```

---

# 45. Useful Linux Examples

Search logs:

```bash
grep -i "failed" logfile.log
```

Recursive search:

```bash
grep -Rni "suspicious" /var/log/
```

Find files:

```bash
find /tmp -type f
```

Processes:

```bash
ps aux
```

Listening ports:

```bash
ss -tulnp
```

Follow log:

```bash
tail -f /var/log/example.log
```

---

# 46. Phase 13 — PowerShell for Analysts

PowerShell is useful both as:

```text
An investigation tool
```

and:

```text
An attacker technique
```

Learn both perspectives.

---

# 47. Useful PowerShell Commands

Current user:

```powershell
whoami
```

Processes:

```powershell
Get-Process
```

Services:

```powershell
Get-Service
```

Network:

```powershell
Get-NetTCPConnection
```

File hash:

```powershell
Get-FileHash <FILE> -Algorithm SHA256
```

Event logs:

```powershell
Get-WinEvent
```

Search specific Event ID:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4625
}
```

---

# 48. Phase 14 — CyberChef

Learn CyberChef for common analyst tasks:

- Base64 decoding
- Hex
- URL decoding
- Defanging / refanging
- Hashing
- XOR basics
- String extraction
- Character encoding

Examples of suspicious text that may need decoding:

```text
aHR0cHM6Ly9leGFtcGxlLmNvbQ==
```

Do not assume encoded data is malicious.

Decode and investigate it.

---

# 49. Phase 15 — Investigation Reporting

A good investigation should answer:

```text
What happened?

Why was it detected?

What evidence was reviewed?

What was the timeline?

What was the verdict?

How confident are we?

What should happen next?
```

Use the repository template:

```text
investigations/investigation-template.md
```

---

# 50. Current Investigation Portfolio

Completed:

```text
001-onedrive-explorer-process-access.md
```

This investigation practised:

- Wazuh alert triage
- Sysmon Event ID 10
- Process access
- Call trace analysis
- Contextual investigation
- Benign-positive classification

---

# 51. Planned Investigation Sequence

Recommended order:

```text
001 OneDrive Process Access
        ↓
002 Windows Failed Logon
        ↓
003 Brute-Force Detection
        ↓
004 PowerShell Investigation
        ↓
005 DNS Investigation
        ↓
006 Suspicious Network Connection
        ↓
007 Phishing Analysis
        ↓
008 PCAP Investigation
```

---

# 52. Investigation 002 — Windows Failed Logon

Planned topics:

```text
Event ID 4625
Account name
Logon type
Source workstation
Source IP
Failure reason
Timeline
```

Goal:

Understand a single failed authentication event.

---

# 53. Investigation 003 — Brute-Force Detection

Expand the failed-logon exercise into:

```text
Many 4625 Events
        ↓
Same User / Source
        ↓
Repeated Attempts
        ↓
Potential Successful 4624
```

Goal:

Identify patterns rather than individual events.

---

# 54. Investigation 004 — PowerShell

Investigate:

- Process creation
- Command line
- Parent process
- Script files
- User
- Network connections
- File creation

A previously observed Wazuh alert involving:

```text
__PSScriptPolicyTest_*.ps1
```

may be useful as a future case study.

---

# 55. Investigation 005 — DNS

Generate and investigate DNS activity.

Example:

```powershell
Resolve-DnsName example.com
```

Analyse:

```text
Process
Domain
Timestamp
Subsequent network connection
```

---

# 56. Investigation 006 — Network Connection

Generate:

```powershell
Test-NetConnection example.com -Port 443
```

Investigate:

```text
Process
Source
Destination
Port
Protocol
DNS relationship
```

---

# 57. Investigation 007 — Phishing

Use a safe phishing sample or training environment.

Investigate:

```text
Headers
Sender
URLs
Attachments
Authentication
Threat Intelligence
```

---

# 58. Investigation 008 — PCAP

Analyse a safe packet capture.

Identify:

```text
Hosts
DNS
Connections
Protocol behaviour
Suspicious traffic
Timeline
```

---

# 59. Weekly Study Structure

A possible weekly pattern:

## Day 1

```text
Theory
Windows / IR / SIEM
```

## Day 2

```text
Splunk / SPL
```

## Day 3

```text
Networking / Wireshark
```

## Day 4

```text
Phishing / Threat Intelligence
```

## Day 5

```text
DFIR / PowerShell / Linux
```

## Weekend

```text
Full investigation lab
+
Documentation
+
Review
```

---

# 60. Three-Month Preparation Structure

A practical high-level plan:

---

## Month 1 — Foundations

Focus:

```text
SOC Fundamentals
Incident Response
Windows Event Logs
Sysmon
Wazuh
Basic SPL
Networking Review
```

Target outcome:

```text
Comfortable reading and explaining endpoint security events.
```

---

## Month 2 — Investigation Skills

Focus:

```text
Splunk
Wireshark
Phishing
Threat Intelligence
MITRE ATT&CK
Authentication investigations
PowerShell investigations
```

Target outcome:

```text
Able to independently investigate common SOC alerts.
```

---

## Month 3 — Exam Readiness

Focus:

```text
Digital Forensics
PCAP analysis
Mixed investigations
Timed practice
Command recall
SPL recall
Report writing
Weak-area revision
```

Target outcome:

```text
Comfortable moving between multiple tools and evidence sources under time pressure.
```

---

# 61. Practice Platforms

Useful external practice can include:

- Security Blue Team free learning content
- Blue Team Labs Online
- CyberDefenders
- LetsDefend
- Splunk training environments
- Safe public PCAP datasets

The home lab remains the persistent environment for practising investigation fundamentals.

---

# 62. Notes Strategy

Keep notes organised by topic.

Suggested structure:

```text
BTL1/
├── Incident Response
├── Windows
├── Sysmon
├── SIEM
├── Splunk
├── Networking
├── Wireshark
├── Phishing
├── Threat Intelligence
├── Digital Forensics
├── MITRE ATT&CK
├── PowerShell
├── Linux
└── Investigations
```

---

# 63. What to Put in Notes

Avoid copying large amounts of theory.

For each topic record:

```text
What is it?
Why does it matter?
What does it look like?
How do I investigate it?
Which command/filter do I use?
What would make it suspicious?
What would I pivot to next?
```

---

# 64. Command Cheat Sheet Goal

By the final preparation stage, maintain concise cheat sheets for:

```text
Windows
PowerShell
Linux
SPL
Wireshark
Event IDs
Sysmon Event IDs
Ports
MITRE ATT&CK
IOC investigation
```

---

# 65. SPL Cheat Sheet Goal

Include:

```text
Basic search
Field filtering
stats
sort
table
dedup
top
rare
timechart
rex
regex
eval
where
```

with examples.

---

# 66. Wireshark Cheat Sheet Goal

Include common filters for:

```text
IP
TCP
UDP
DNS
HTTP
Ports
Source
Destination
TCP streams
```

---

# 67. Exam Investigation Workflow

A useful workflow under time pressure:

```text
Read Question Carefully
        ↓
Identify Evidence Source
        ↓
Establish Time Range
        ↓
Find Relevant Host/User
        ↓
Search Broadly
        ↓
Narrow Results
        ↓
Build Timeline
        ↓
Correlate Evidence
        ↓
Record Findings
        ↓
Answer Only What Evidence Supports
```

---

# 68. Avoid Tunnel Vision

Do not stop at the first suspicious event.

For example:

```text
PowerShell Found
```

should trigger pivots into:

```text
Parent Process
Command Line
Files
DNS
Network
User
Timeline
```

---

# 69. Avoid Over-Interpreting Evidence

Do not write:

```text
The machine was compromised.
```

when the evidence only supports:

```text
Suspicious PowerShell activity was observed.
```

Use confidence-appropriate language.

---

# 70. Evidence Language

Useful wording:

```text
The evidence indicates...

The activity is consistent with...

No additional evidence was identified...

This may represent...

The available telemetry does not confirm...

Further validation would be required...
```

---

# 71. Common Pitfalls

Avoid:

- Memorising without understanding
- Ignoring timestamps
- Searching the wrong time range
- Treating severity as verdict
- Ignoring parent processes
- Ignoring user context
- Looking at one event in isolation
- Forgetting networking fundamentals
- Overusing threat intelligence without context
- Spending too long on one question
- Failing to document evidence

---

# 72. Lab Safety

All security testing should remain:

```text
Inside the authorised lab
```

Do not scan or attack systems without permission.

Use Kali only against:

- Your own VMs
- Purpose-built training platforms
- Explicitly authorised targets

---

# 73. ARM64 Considerations

The current lab runs on Apple Silicon.

This is suitable for:

- Windows telemetry
- Wazuh
- Networking
- Wireshark
- Phishing
- Threat intelligence
- Splunk hosted practice
- Incident response
- DFIR fundamentals

Some x86-only malware-analysis tooling may require separate hardware or hosted environments later.

That is outside the current BTL1 priority.

---

# 74. What Not to Prioritise Yet

Avoid getting distracted by advanced areas too early:

```text
Advanced malware reverse engineering
Exploit development
Advanced red-team tooling
Complex Active Directory attacks
BTL2-level material
```

These can be studied later.

Current priority:

```text
Strong SOC fundamentals.
```

---

# 75. Progress Tracking

Use a simple system:

```text
[ ] Not Started
[~] In Progress
[x] Comfortable
```

Example:

```text
[x] Wazuh installation
[x] Sysmon installation
[x] Sysmon → Wazuh
[x] Basic alert triage
[~] Windows authentication
[~] SPL
[ ] Phishing analysis
[ ] PCAP investigation
[ ] Full timed investigation
```

---

# 76. Current Progress

At the time of writing:

```text
[x] Windows 11 ARM64 lab
[x] Kali ARM64 lab
[x] Ubuntu Wazuh server
[x] Sysmon
[x] Wazuh Windows Agent
[x] Sysmon → Wazuh integration
[x] First SOC investigation
[x] Lab troubleshooting documentation
[x] Snapshot strategy documentation

[~] Windows security event analysis
[~] SIEM investigation
[~] Incident response fundamentals
[~] Splunk / SPL
[~] Networking review

[ ] Phishing investigation
[ ] Threat intelligence workflow
[ ] Full PCAP investigation
[ ] Digital forensics investigation
[ ] Timed mixed investigation
```

---

# 77. Final Readiness Criteria

Before attempting the certification, I want to be able to:

- Explain the SOC investigation lifecycle
- Investigate 4624 / 4625 events
- Interpret common Sysmon events
- Build a process tree
- Use SPL without relying entirely on notes
- Analyse DNS and network connections
- Navigate Wireshark efficiently
- Investigate phishing emails
- Enrich IOCs
- Map behaviour to MITRE ATT&CK
- Use Linux investigation commands
- Use PowerShell for Windows investigation
- Build a timeline
- Distinguish benign from suspicious behaviour
- Write a concise investigation conclusion

---

# 78. Final Practice Stage

The final stage should focus on mixed investigations.

Example:

```text
Failed Logins
      ↓
Successful Login
      ↓
PowerShell Execution
      ↓
DNS Query
      ↓
Outbound Connection
      ↓
File Creation
```

The objective is to correlate multiple pieces of telemetry rather than investigate each event independently.

---

# 79. Exam-Day Mindset

During the exam:

```text
Read carefully.

Do not panic over unfamiliar data.

Start with what is known.

Establish the timeline.

Use evidence.

Pivot logically.

Keep notes.

Answer the question being asked.

Do not make claims the evidence does not support.
```

---

# 80. Key Takeaway

The roadmap is built around one principle:

> **The goal is not to memorise security tools. The goal is to learn how to investigate.**

Tools will change.

The investigation process remains:

```text
Observe
   ↓
Question
   ↓
Search
   ↓
Correlate
   ↓
Interpret
   ↓
Conclude
   ↓
Respond
```

That is the core skill this roadmap is designed to develop.

---

# Related Documentation

Architecture:

```text
../docs/01-architecture.md
```

Sysmon:

```text
../docs/04-sysmon-setup.md
```

Wazuh:

```text
../docs/06-wazuh-server-setup.md
```

Sysmon to Wazuh:

```text
../docs/08-sysmon-to-wazuh.md
```

Troubleshooting:

```text
../docs/09-troubleshooting.md
```

Snapshots and Recovery:

```text
../docs/10-snapshots-and-recovery.md
```

Investigations:

```text
../investigations/README.md
```

Investigation Template:

```text
../investigations/investigation-template.md
```
