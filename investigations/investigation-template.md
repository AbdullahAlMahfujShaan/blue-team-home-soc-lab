# Investigation XXX — Investigation Title

## Investigation Summary

| Field | Value |
|---|---|
| Investigation ID | `INV-XXX` |
| Date | `<YYYY-MM-DD>` |
| Endpoint | `<HOSTNAME>` |
| User | `<USER>` |
| Detection Source | `<Wazuh / Sysmon / Windows / Splunk / Other>` |
| Rule / Alert ID | `<RULE-ID>` |
| Severity | `<SEVERITY>` |
| Relevant Event ID | `<EVENT-ID>` |
| Initial Classification | `<INITIAL CLASSIFICATION>` |
| Final Verdict | `<VERDICT>` |
| Escalation Required | `<Yes / No>` |

---

# 1. Executive Summary

Briefly describe what triggered the investigation.

Example:

```text
A security alert was generated after suspicious activity was observed on
the endpoint <HOSTNAME>.
```

Summarise:

- What happened
- Why it was suspicious
- What evidence was reviewed
- Final verdict

Keep this section concise.

---

# 2. Investigation Objective

The objective of this investigation was to determine whether:

```text
<Observed Activity>
```

represented:

1. Legitimate activity
2. Suspicious activity
3. Malicious activity
4. A confirmed security incident

---

# 3. Detection Source

Describe where the alert originated.

Example:

```text
Windows Activity
      ↓
Sysmon
      ↓
Wazuh Agent
      ↓
Wazuh Manager
      ↓
Detection Rule
      ↓
Alert
```

Relevant detection details:

```text
Detection Platform:
<PLATFORM>

Rule ID:
<RULE-ID>

Alert Severity:
<SEVERITY>
```

---

# 4. Alert Description

Alert description:

```text
<ALERT DESCRIPTION>
```

Initial questions:

```text
What behaviour triggered the alert?

Why does the detection consider it suspicious?

Which technique could this behaviour represent?
```

---

# 5. Affected Host

```text
Hostname:
<HOSTNAME>

Operating System:
<OS>

IP Address:
<IP>
```

Additional context:

```text
<HOST CONTEXT>
```

---

# 6. User Context

Relevant user:

```text
<DOMAIN\USER>
```

Determine:

```text
Was the user expected?

Was the user interactive?

Was the account privileged?

Was the activity consistent with the user's normal behaviour?
```

---

# 7. Relevant Event Information

```text
Event ID:
<EVENT-ID>

Timestamp:
<TIMESTAMP>

Process ID:
<PID>

Process GUID:
<PROCESS-GUID>
```

Add or remove fields depending on the investigation.

---

# 8. Process Analysis

## Source Process

```text
<SOURCE PROCESS>
```

Path:

```text
<SOURCE PATH>
```

Command line:

```text
<COMMAND LINE>
```

Parent process:

```text
<PARENT PROCESS>
```

Questions:

```text
Is the process expected?

Is the path legitimate?

Is the parent process logical?

Is the command line suspicious?

Is the process signed?
```

---

# 9. Target Process

If applicable:

```text
<TARGET PROCESS>
```

Path:

```text
<TARGET PATH>
```

Describe why interaction with this target process matters.

---

# 10. File Activity

Relevant files:

```text
<FILE PATH>
```

Relevant event type:

```text
<File Created / Modified / Deleted / Executed>
```

Questions:

```text
Was the file expected?

Was it created in a suspicious location?

Was the extension suspicious?

Was the file executed?

Does it have a known hash?
```

---

# 11. Network Activity

Relevant destination:

```text
Destination IP:
<IP>

Destination Port:
<PORT>

Protocol:
<TCP / UDP>

Domain:
<DOMAIN>
```

Questions:

```text
Is the destination expected?

Is the port normal for the application?

Was the connection inbound or outbound?

Did the process initiate the connection?

Is the destination known to be malicious?
```

---

# 12. DNS Activity

Relevant DNS queries:

```text
<DOMAIN>
```

Questions:

```text
Was the domain expected?

Was it recently registered?

Does the process normally contact it?

Were there unusually long or random subdomains?
```

---

# 13. Authentication Activity

If relevant:

```text
Event ID:
<4624 / 4625 / Other>

Logon Type:
<LOGON TYPE>

Source IP:
<SOURCE IP>

Account:
<ACCOUNT>
```

Questions:

```text
Was the login successful?

How many failures occurred?

Was the source expected?

Was the logon type normal?
```

---

# 14. Timeline

Build a simple chronological timeline.

| Time | Event |
|---|---|
| `<TIME>` | `<EVENT>` |
| `<TIME>` | `<EVENT>` |
| `<TIME>` | `<EVENT>` |

Example:

```text
09:00:01 — User logged in
09:01:15 — PowerShell launched
09:01:30 — File created
09:01:45 — DNS query observed
09:01:46 — Outbound network connection established
09:02:00 — Wazuh alert generated
```

---

# 15. Initial Hypotheses

## Hypothesis 1 — Benign Activity

Describe why the activity could be legitimate.

```text
<EVIDENCE>
```

---

## Hypothesis 2 — Suspicious Activity

Describe what could make the behaviour suspicious.

```text
<EVIDENCE>
```

---

## Hypothesis 3 — Malicious Activity

Describe the malicious scenario the detection is intended to identify.

```text
<EVIDENCE>
```

---

# 16. Evidence Supporting Benign Activity

Examples:

- Known executable
- Expected file path
- Expected user
- Normal parent process
- Expected destination
- Valid digital signature
- No suspicious correlated activity

Evidence:

```text
<EVIDENCE>
```

---

# 17. Evidence Supporting Malicious Activity

Examples:

- Unknown executable
- Suspicious path
- Encoded command
- Unexpected parent process
- Malicious destination
- Credential access
- Persistence
- Suspicious DLL
- Unusual authentication pattern

Evidence:

```text
<EVIDENCE>
```

---

# 18. Evidence Gaps

Document anything that could not be verified.

Examples:

```text
Digital signature not checked.
Hash reputation not checked.
Full PCAP unavailable.
EDR telemetry unavailable.
```

This is important because an analyst should distinguish between:

```text
No evidence found
```

and:

```text
Evidence was not available
```

---

# 19. MITRE ATT&CK Mapping

Relevant technique:

```text
<TACTIC / TECHNIQUE>
```

Example:

```text
T1059.001 — PowerShell
```

or:

```text
T1055 — Process Injection
```

Explain why the technique may be relevant.

Do not claim a MITRE technique was confirmed unless the evidence actually supports it.

---

# 20. Final Verdict

## Classification

```text
<Benign / Benign Positive / False Positive / Suspicious / Malicious / Confirmed Incident>
```

## Confidence

```text
<Low / Moderate / High>
```

## Escalation

```text
<Required / Not Required>
```

---

# 21. Analyst Conclusion

Summarise the reasoning behind the verdict.

Example structure:

```text
The alert was generated because <activity> matched detection logic associated
with <technique>.

Investigation showed <key evidence>.

No evidence of <malicious behaviour> was identified during the reviewed
time period.

The activity was therefore classified as <verdict>.
```

---

# 22. Recommended Response

If benign:

```text
No containment required.
Continue monitoring.
Consider detection tuning only if the behaviour is consistently understood.
```

If suspicious:

```text
Collect additional telemetry.
Validate hashes.
Review process ancestry.
Review network activity.
Search other endpoints.
```

If malicious:

```text
Escalate incident.
Isolate affected endpoint.
Preserve evidence.
Identify affected accounts.
Block malicious indicators.
Determine scope.
Begin incident response.
```

---

# 23. Detection Tuning Considerations

Consider whether the alert should:

```text
Remain unchanged
Be enriched
Be tuned
Be suppressed under specific conditions
```

Never suppress a detection solely because one alert was benign.

Document any proposed tuning logic.

---

# 24. Lessons Learned

## Lesson 1

```text
<LESSON>
```

## Lesson 2

```text
<LESSON>
```

## Lesson 3

```text
<LESSON>
```

---

# 25. Skills Practised

Possible examples:

- SIEM alert triage
- Windows Event Log analysis
- Sysmon analysis
- Process investigation
- Network analysis
- Authentication analysis
- Threat hunting
- Timeline building
- MITRE ATT&CK mapping
- Incident documentation
- Evidence correlation

---

# 26. Key Takeaway

Summarise the main lesson from the investigation.

Example:

> **An alert is evidence to investigate, not a verdict.**

---

# 27. Related Documentation

Architecture:

```text
../docs/01-architecture.md
```

Windows endpoint:

```text
../docs/03-windows-11-setup.md
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

---

## References

- [Microsoft Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Wazuh Documentation](https://documentation.wazuh.com/)
- [MITRE ATT&CK](https://attack.mitre.org/)
