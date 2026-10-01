# Directory — PCAP Forensic Analysis & Walkthrough

## Overview

| Field | Details |
|---|---|
| **Room** | TryHackMe — Directory |
| **Category** | Network Forensics / Packet Analysis |
| **Primary Tool** | Wireshark |
| **Key Concepts** | Port Scan Analysis, Kerberos AS-REP Roasting, Hash Extraction, WinRM Analysis, Registry Hive Extraction |
| **Evidence** | `traffic.pcap` |

---

## Scenario

A small music company was targeted by a threat actor. The Art Director discovered a note on their Desktop after the incident.

At the time of the attack, the IT team did not have specialized endpoint collection tools available. However, network traffic had been captured throughout the incident in a packet capture named `traffic.pcap`.

The objective of this investigation is to analyze the packet capture and reconstruct the attack chain, including:

- Initial network reconnaissance
- Open port discovery
- Active Directory user enumeration
- Kerberos AS-REP Roasting
- Credential hash extraction
- Offline password cracking
- WinRM remote access
- Post-exploitation activity
- Registry hive extraction
- Final artifact retrieval

---

# Investigation Methodology

## 1. Initial Reconnaissance & Port Scanning

The first step was to identify how the threat actor profiled the target system.

During a TCP SYN scan, a server responds with a `SYN/ACK` packet when a port is open and accepting connections.

### Wireshark Filter

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 1
```

### Analysis

The filter displays TCP packets where both the `SYN` and `ACK` flags are set.

These responses indicate that the target acknowledged the attacker's connection attempts, allowing us to identify potentially open TCP ports.

### Discovered Open Ports

| Port | Likely Service |
|---:|---|
| `53` | DNS |
| `80` | HTTP |
| `88` | Kerberos |
| `135` | MSRPC |
| `139` | NetBIOS Session Service |
| `389` | LDAP |
| `445` | SMB |
| `464` | Kerberos Password Change |
| `593` | RPC over HTTP |
| `636` | LDAPS |
| `3268` | Global Catalog |
| `3269` | Global Catalog over SSL |
| `5985` | WinRM over HTTP |

The presence of ports such as `88`, `389`, `445`, `3268`, and `5985` strongly suggests a Windows Active Directory environment with remote management services enabled.

---

# 2. User Enumeration & AS-REP Roasting

The next stage involved investigating Kerberos traffic to determine whether the attacker was attempting to identify valid domain accounts.

## Wireshark Filter

```wireshark
krb5
```

### Analysis

Inspection of Kerberos requests (`KRB-REQ`) revealed authentication attempts involving multiple usernames.

For most accounts, the Key Distribution Center (KDC) responded with:

```text
KDC_ERR_PREAUTH_REQUIRED
```

This indicates that Kerberos pre-authentication was required before the KDC would issue an authentication response.

However, one account behaved differently:

```text
larry.doe
```

The account was configured with:

```text
UF_DONT_REQUIRE_PREAUTH
```

This configuration prevents Kerberos pre-authentication from being required for the account.

As a result, the attacker was able to request an AS-REP response without first providing valid pre-authentication data.

The KDC returned a `KRB-REP` response containing encrypted authentication material that could be used for offline password cracking.

### Compromised Account

```text
directory.thm\larry.doe
```

### Attack Technique

**MITRE ATT&CK:** `T1558.004 — Steal or Forge Kerberos Tickets: AS-REP Roasting`

---

# 3. Hash Extraction & Offline Credential Cracking

The encrypted Kerberos response was then extracted from the packet capture.

## Hash Extraction

The relevant response was found in:

```text
Frame: 4817
```

Within the Kerberos protocol fields, the following area contained the encrypted authentication material:

```text
AS-REP
└── enc-part
    └── kerberos.cipher
```

The relevant portion of the cipher was identified during packet inspection.

### Cipher Fragment

```text
55616532b664cd0b50cda8d4ba469f
```

The captured material was then formatted into the appropriate `$krb5asrep$23$` hash structure for offline cracking.

---

## Password Cracking

Hashcat was used with the `rockyou.txt` wordlist.

```bash
hashcat -m 18200 hash.txt rockyou.txt
```

Where:

| Parameter | Meaning |
|---|---|
| `-m 18200` | Hashcat mode for Kerberos 5 AS-REP etype 23 |
| `hash.txt` | File containing the extracted AS-REP hash |
| `rockyou.txt` | Password dictionary |

### Recovered Credentials

```text
Username: larry.doe
Password: Password1!
```

The attacker could now use these credentials to authenticate to services available to the compromised account.

---

# 4. Post-Exploitation & WinRM Analysis

During the initial reconnaissance phase, TCP port `5985` was identified as open.

Port `5985` is commonly used by:

```text
Windows Remote Management (WinRM)
```

The attacker used the previously compromised credentials to establish a remote management session.

## Wireshark Filter

```wireshark
http && ip.addr == <TARGET_IP>
```

### Analysis

WinRM over HTTP uses SOAP/XML messages to communicate between the client and Windows host.

The traffic contains HTTP requests to the WinRM endpoint:

```text
/wsman
```

Inspecting the HTTP streams allowed the commands executed during the remote session to be reconstructed.

---

## Command 1 — Identify Current User

The attacker first executed:

```cmd
whoami
```

This is commonly used to confirm the identity and privileges associated with the current session.

---

## Command 2 — Dump the SYSTEM Registry Hive

The attacker then executed:

```cmd
reg save HKLM\SYSTEM C:\SYSTEM
```

This creates a copy of the Windows `SYSTEM` registry hive.

The `SYSTEM` hive contains security-sensitive configuration information and is also required when extracting and interpreting credentials stored in the SAM database.

---

## Command 3 — Dump the SAM Registry Hive

The attacker subsequently executed:

```cmd
reg save HKLM\SAM C:\SAM
```

This creates a copy of the Windows Security Account Manager (SAM) registry hive.

The SAM contains local Windows account credential hashes.

Obtaining both:

```text
SYSTEM
SAM
```

can allow an attacker to extract and analyze local account password hashes offline.

### Attack Technique

This activity is associated with credential access through registry hive extraction.

Relevant MITRE ATT&CK technique:

**T1003.002 — OS Credential Dumping: Security Account Manager**

---

# 5. Final Artifact Retrieval

After obtaining the registry hives, the attacker continued interacting with the compromised Windows host through the remote session.

The attacker navigated the filesystem, staged files, and ultimately left a ransom note on the Art Director's Desktop.

The final artifact provided the TryHackMe flag.

### Flag

```text
THM{REDACTED}
```

---

# Attack Timeline

The activity can be summarized as follows:

```text
Network Reconnaissance
        │
        ▼
TCP Port Scanning
        │
        ▼
Active Directory Services Identified
        │
        ▼
Kerberos User Enumeration
        │
        ▼
larry.doe Identified
        │
        ▼
AS-REP Roasting
        │
        ▼
Kerberos Cipher Extracted
        │
        ▼
Offline Password Cracking
        │
        ▼
larry.doe : Password1!
        │
        ▼
WinRM Authentication
        │
        ▼
whoami
        │
        ▼
SYSTEM Hive Extraction
        │
        ▼
SAM Hive Extraction
        │
        ▼
Filesystem Activity
        │
        ▼
Ransom Note / Final Artifact
```

---

# Blue Team Takeaways

## 1. Detecting AS-REP Roasting

AS-REP Roasting can be detected by monitoring Kerberos authentication events.

### Relevant Windows Event

```text
Event ID: 4768
A Kerberos authentication ticket was requested
```

Investigate accounts where Kerberos pre-authentication is not required.

Useful indicators include:

```text
Pre-Authentication Type: 0x0
```

and, depending on the environment:

```text
Ticket Encryption Type: 0x17
```

The presence of these characteristics can help identify potential AS-REP Roasting activity.

---

## 2. Detecting WinRM Activity

WinRM activity should be monitored, particularly when remote sessions originate from unexpected hosts or users.

Relevant telemetry can include:

- Windows PowerShell logging
- PowerShell Script Block Logging
- WinRM operational logs
- Process creation events
- Network connections to TCP `5985` and `5986`

### PowerShell Event

```text
Event ID: 4104
PowerShell Script Block Logging
```

Security teams can correlate remote WinRM sessions with subsequent process execution and command-line activity.

---

## 3. Detecting Registry Hive Extraction

Commands such as the following should receive additional scrutiny:

```cmd
reg save HKLM\SAM C:\SAM
```

```cmd
reg save HKLM\SYSTEM C:\SYSTEM
```

Security monitoring should alert on suspicious executions of:

```text
reg.exe
```

when the command line references sensitive registry hives such as:

```text
HKLM\SAM
HKLM\SYSTEM
HKLM\SECURITY
```

These operations can be legitimate administrative activities, so detections should consider the initiating user, host, parent process, timing, and surrounding activity.

---

# Hardening & Remediation

## 1. Enforce Kerberos Pre-Authentication

Audit Active Directory accounts for the:

```text
DONT_REQ_PREAUTH
```

configuration.

Where not required for a specific operational reason, enable Kerberos pre-authentication.

This reduces the opportunity for attackers to obtain AS-REP responses that can be subjected to offline password cracking.

---

## 2. Strengthen Password Policies

Use strong, unique passwords or passphrases for domain accounts.

Longer passwords significantly increase the difficulty of offline dictionary and brute-force attacks.

Password policies should be combined with:

- MFA where supported
- Account monitoring
- Password screening
- Protection of privileged accounts
- Regular credential auditing

---

## 3. Restrict WinRM Access

Limit access to WinRM ports:

```text
5985 — HTTP
5986 — HTTPS
```

Where possible, restrict WinRM access to authorized administrative systems such as:

- Management servers
- Privileged Access Workstations
- Jump hosts

Host-based and network firewalls can be used to prevent unnecessary lateral access.

---

## 4. Monitor Credential Access

Security teams should monitor for suspicious combinations of activity, rather than relying on a single indicator.

For example:

```text
AS-REP request
      +
Successful authentication
      +
WinRM session
      +
reg.exe execution
      +
SAM/SYSTEM hive access
```

A sequence like this provides much stronger evidence of malicious activity than any individual event.

---

# Key Lessons

This investigation demonstrates how a relatively small packet capture can reveal a substantial portion of an attack chain.

The key lessons are:

1. **Network reconnaissance leaves observable patterns.**
2. **Kerberos configuration weaknesses can expose accounts to AS-REP Roasting.**
3. **Captured Kerberos material can be attacked offline without directly knowing the user's password.**
4. **Weak passwords can turn an authentication artifact into valid credentials.**
5. **WinRM can provide attackers with remote command execution.**
6. **Registry hive extraction can expose local credential hashes.**
7. **Packet analysis can reconstruct attacker commands even when endpoint tooling was unavailable.**
8. **Network telemetry and Windows endpoint logs are most effective when correlated together.**

---

# Tools Used

| Tool | Purpose |
|---|---|
| **Wireshark** | PCAP analysis and packet inspection |
| **Hashcat** | Offline password cracking |
| **rockyou.txt** | Password wordlist |
| **Windows Event Logs** | Detection and investigation reference |
| **WinRM** | Remote Windows management protocol |

---

# MITRE ATT&CK Techniques

| Technique | Name | Activity |
|---|---|---|
| `T1046` | Network Service Scanning | TCP port scanning |
| `T1558.004` | Steal or Forge Kerberos Tickets: AS-REP Roasting | Exploitation of an account without Kerberos pre-authentication |
| `T1003.002` | OS Credential Dumping: Security Account Manager | SAM registry hive extraction |
| `T1021.006` | Windows Remote Management | Remote access through WinRM |

---

## Investigation Conclusion

The packet capture allowed the attack to be reconstructed from initial reconnaissance through post-exploitation.

The threat actor:

1. Scanned the target for available services.
2. Identified Active Directory and Kerberos services.
3. Enumerated domain accounts.
4. Identified `larry.doe` as an account that did not require Kerberos pre-authentication.
5. Obtained an AS-REP response for the account.
6. Extracted the encrypted authentication material.
7. Cracked the captured material offline.
8. Recovered the account password.
9. Used the credentials to establish a WinRM session.
10. Executed commands on the Windows host.
11. Extracted the `SYSTEM` and `SAM` registry hives.
12. Continued filesystem activity and left the final artifact.

This investigation demonstrates the value of combining **network forensics, Windows authentication knowledge, Kerberos analysis, credential attack techniques, and detection engineering** when investigating an intrusion.
