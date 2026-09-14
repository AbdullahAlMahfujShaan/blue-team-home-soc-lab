# SOC Investigations

This directory contains hands-on security investigations completed as part of my **Blue Team Home SOC Lab**.

The goal of these investigations is to practise the same analytical workflow used in a Security Operations Center:

```text
Alert
  ↓
Triage
  ↓
Evidence Collection
  ↓
Correlation
  ↓
Investigation
  ↓
Verdict
  ↓
Documentation
```

Each investigation focuses on understanding what happened, how it was detected, what evidence supports the findings, and whether the activity was benign, suspicious, or malicious.

---

## Investigation Methodology

The general investigation process used in this repository is:

```text
1. Identify the alert
2. Understand the detection logic
3. Identify the affected host
4. Identify the user
5. Analyse relevant processes
6. Review command-line activity
7. Review file activity
8. Review network activity
9. Review DNS activity
10. Correlate surrounding telemetry
11. Build a timeline
12. Determine a verdict
13. Document findings
```

The focus is not simply on closing alerts.

The goal is to understand the behaviour behind them.

---

## Current Investigations

| ID | Investigation | Detection Source | Verdict |
|---|---|---|---|
| `001` | [OneDrive Accessing Windows Explorer](./001-onedrive-explorer-process-access.md) | Wazuh / Sysmon Event ID 10 | Likely Benign / Benign Positive |

---

## Planned Investigations

Future investigations may include:

```text
002-windows-failed-logon-investigation.md
003-brute-force-detection.md
004-powershell-investigation.md
005-dns-investigation.md
006-suspicious-network-connection.md
007-phishing-analysis.md
008-pcap-investigation.md
```

These will cover areas such as:

- Windows authentication
- Failed logons
- Brute-force patterns
- PowerShell activity
- DNS analysis
- Network connections
- Phishing analysis
- Packet captures
- Threat hunting
- Detection engineering

---

## Investigation Classification

Possible verdicts include:

### Benign

Expected and harmless activity.

### Benign Positive

The detection correctly identified the behaviour, but the behaviour was legitimate.

### False Positive

The detection incorrectly identified normal behaviour as suspicious.

### Suspicious

The activity requires additional investigation before a confident verdict can be reached.

### Malicious

Evidence indicates intentional or harmful activity.

### Confirmed Incident

Malicious activity has been confirmed and requires response or containment.

---

## Important Principle

A core principle used throughout these investigations is:

> **An alert is evidence to investigate, not a verdict.**

A high-severity detection does not automatically mean a system has been compromised.

The analyst must add context.

---

## Detection Model

```text
Telemetry
    ↓
Detection Logic
    ↓
Alert
    ↓
Analyst Investigation
    ↓
Verdict
```

This distinction is important because:

```text
Event ≠ Alert
Alert ≠ Incident
Severity ≠ Maliciousness
```

---

## Evidence Sources

Depending on the investigation, evidence may come from:

- Windows Event Logs
- Microsoft Sysmon
- Wazuh
- PowerShell
- Process telemetry
- DNS logs
- Network connections
- File creation events
- Authentication events
- Wireshark / PCAP
- Threat intelligence
- MITRE ATT&CK

---

## Investigation Documentation

Each case should contain:

- Executive summary
- Alert details
- Detection source
- Initial hypothesis
- Relevant telemetry
- Process analysis
- User analysis
- Network activity
- File activity
- Timeline
- Evidence supporting or contradicting the hypothesis
- Final verdict
- Analyst conclusion
- Lessons learned
- MITRE ATT&CK mapping where relevant

---

## Naming Convention

Investigation files use:

```text
<number>-<short-description>.md
```

Examples:

```text
001-onedrive-explorer-process-access.md
002-windows-failed-logon-investigation.md
003-brute-force-detection.md
```

The numbering keeps investigations in chronological order.

---

## Security and Privacy

Before publishing investigations:

- Remove passwords
- Remove API keys
- Remove tokens
- Remove private keys
- Remove personal information
- Remove sensitive customer information
- Remove confidential corporate data
- Sanitize screenshots where required

Private RFC1918 lab addresses may appear in the documentation, but generic placeholders can be used where appropriate.

Examples:

```text
<WAZUH-SERVER-IP>
<WINDOWS-ENDPOINT-IP>
<KALI-IP>
```

---

## Portfolio Goal

The purpose of this folder is to demonstrate practical Blue Team skills such as:

- Alert triage
- SIEM analysis
- Windows telemetry analysis
- Process investigation
- Authentication analysis
- Network analysis
- Threat hunting
- Incident investigation
- Detection interpretation
- Evidence correlation
- Technical documentation

The emphasis is on showing the reasoning behind each investigation rather than only showing the final answer.
