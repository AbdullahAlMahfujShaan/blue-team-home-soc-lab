# Snapshots & Recovery Strategy

This document describes the snapshot, rollback, and recovery strategy used in the **Blue Team Home SOC Lab**.

Snapshots provide known-good recovery points before major configuration changes, security testing, software installation, and troubleshooting.

The purpose of this strategy is to make the lab:

- Repeatable
- Recoverable
- Safer to experiment with
- Easier to troubleshoot
- Easier to rebuild
- Better documented

---

# 1. Overview

The lab currently contains three main virtual machines:

```text
BTL-WIN11
BTL-WAZUH
BTL-KALI
```

Each VM has a different role and therefore requires a slightly different snapshot strategy.

The main principle is:

> Create a recovery point before making a major change.

---

# 2. Why Snapshots Matter

Home security labs are intentionally changed often.

Common activities include:

- Installing security tools
- Changing configuration files
- Testing detection rules
- Modifying network settings
- Running PowerShell exercises
- Generating suspicious activity
- Installing updates
- Troubleshooting broken services
- Testing attack simulations

Any of these changes can make a VM unstable or difficult to recover.

Snapshots provide a way to return to a known working state.

---

# 3. Snapshot Workflow

The general workflow is:

```text
Known Working State
        │
        ▼
Create Snapshot
        │
        ▼
Make Change
        │
        ▼
Test
        │
   ┌────┴─────┐
   │          │
 Success    Failure
   │          │
   ▼          ▼
Continue   Investigate
              │
              ▼
         Restore Snapshot
         if appropriate
```

---

# 4. Important Limitation

Snapshots are useful recovery points, but they are **not full backups**.

A snapshot depends on the underlying virtual machine files.

If the VM files are deleted, corrupted, or lost, snapshots may also become unusable.

Therefore:

```text
Snapshot ≠ Backup
```

A proper backup should eventually include copies of important VM files, documentation, configurations, and investigation reports.

---

# 5. Current Snapshot Inventory

The lab currently contains the following known recovery points.

| Virtual Machine | Snapshot | Purpose |
|---|---|---|
| `BTL-WIN11` | Windows - Sysmon Baseline | Known-good Windows endpoint after Sysmon setup |
| `BTL-KALI` | `00 - Clean Kali` | Clean Kali baseline |
| `BTL-WAZUH` | `00 - Clean Ubuntu Server` | Clean Ubuntu state before Wazuh installation |

---

# 6. `BTL-WIN11` Snapshot

The Windows recovery point was created after:

- Windows 11 installation
- Windows Update
- Hostname configuration
- Lab account creation
- Basic networking validation
- Sysmon installation
- Sysmon configuration
- Sysmon telemetry testing

This snapshot represents:

```text
Windows - Sysmon Baseline
```

---

# 7. Windows Snapshot State

The snapshot contains a working endpoint with:

```text
Windows 11 Pro ARM64
        │
        ├── Hostname: BTL-WIN11
        ├── labadmin
        ├── testuser
        ├── Networking
        ├── Windows Event Logs
        └── Microsoft Sysmon
```

This creates a reliable recovery point before later SIEM and investigation exercises.

---

# 8. Why the Windows Baseline Is Important

The Windows endpoint will receive many future changes.

Examples include:

- Wazuh Agent configuration
- Logging changes
- Detection tests
- PowerShell exercises
- Authentication exercises
- Network testing
- Additional security tooling

If a future exercise causes problems, the VM can be restored to a state where:

```text
Windows + Sysmon
```

are already known to work correctly.

---

# 9. Recommended Future Windows Snapshots

Useful future Windows recovery points may include:

```text
01 - Windows Clean Baseline
02 - Sysmon Baseline
03 - Wazuh Agent Active
04 - Sysmon to Wazuh Verified
05 - Pre Detection Lab
```

Do not create snapshots after every minor action.

Snapshots are most useful at meaningful milestones.

---

# 10. `BTL-KALI` Snapshot

A clean snapshot was created after:

- Kali installation
- Initial system updates
- Xfce setup
- VMware Tools installation
- Basic network validation

The recovery point represents:

```text
00 - Clean Kali
```

---

# 11. Kali Snapshot State

The clean Kali snapshot contains:

```text
Kali Linux ARM64
        │
        ├── Hostname: btl-kali
        ├── Xfce
        ├── Updated Packages
        ├── open-vm-tools
        ├── open-vm-tools-desktop
        └── Working Network
```

No unnecessary security-testing changes should be part of this clean baseline.

---

# 12. Why the Kali Baseline Is Important

Kali will eventually be used to install or test additional tools.

Some packages may:

- Change dependencies
- Modify configuration
- Break networking
- Introduce compatibility problems
- Conflict with other packages
- Create unstable environments

The clean snapshot allows the VM to return to:

```text
Fresh + Updated + Working
```

without reinstalling Kali from scratch.

---

# 13. Recommended Kali Snapshot Strategy

Before major testing environments:

```text
00 - Clean Kali
        │
        ▼
Install / Configure Tools
        │
        ▼
Create Temporary Lab Snapshot
        │
        ▼
Perform Exercise
```

After the exercise:

```text
Keep Changes
```

or:

```text
Restore Clean Snapshot
```

depending on whether the changes are worth preserving.

---

# 14. `BTL-WAZUH` Snapshot

A snapshot was created after:

- Ubuntu Server installation
- Ubuntu updates
- OpenSSH installation
- Network validation

but before Wazuh installation.

The snapshot represents:

```text
00 - Clean Ubuntu Server
```

---

# 15. Wazuh Snapshot State

The snapshot contains:

```text
Ubuntu Server ARM64
        │
        ├── Hostname: btl-wazuh
        ├── Updated Packages
        ├── OpenSSH
        ├── Working Network
        └── No Wazuh Installation
```

---

# 16. Why the Pre-Wazuh Snapshot Is Important

Installing Wazuh introduces several components:

```text
Wazuh Manager
Wazuh Indexer
Wazuh Dashboard
```

If the Wazuh installation fails or becomes severely damaged, the VM can be restored to a clean Ubuntu state.

This is significantly faster than reinstalling the entire operating system.

---

# 17. Recommended Wazuh Snapshot Strategy

A useful future snapshot sequence may be:

```text
00 - Clean Ubuntu Server
        │
        ▼
01 - Wazuh Installed
        │
        ▼
02 - Windows Agent Connected
        │
        ▼
03 - Sysmon Integration Verified
        │
        ▼
04 - Pre Detection Engineering
```

This provides multiple rollback points.

---

# 18. Snapshot Naming Convention

A consistent naming convention makes snapshots easier to understand.

Recommended format:

```text
<number> - <meaningful state>
```

Examples:

```text
00 - Clean Kali
00 - Clean Ubuntu Server
01 - Wazuh Installed
02 - Agent Connected
03 - Sysmon Integrated
```

For Windows:

```text
00 - Clean Windows
01 - Sysmon Baseline
02 - Wazuh Agent Active
```

---

# 19. Include Meaning, Not Just Dates

Avoid names such as:

```text
Snapshot 1
Snapshot 2
Monday
Test
Backup
```

These names provide little information later.

Prefer:

```text
Pre Wazuh Install
Sysmon Working
Agent Active
Pre Detection Lab
```

---

# 20. Snapshot Notes

If VMware Fusion allows snapshot notes or descriptions, record:

- Date
- Configuration state
- What was installed
- What is known to work
- Why the snapshot was created

Example:

```text
Windows 11 ARM64
Sysmon installed
Event IDs 1, 3, and 22 verified
Wazuh Agent not yet installed
```

---

# 21. VMware Fusion Snapshot Limitation

In the current VMware Fusion workflow, snapshot renaming was not available through the expected interface.

Because of this, snapshot meaning is also tracked in the GitHub documentation.

The repository should be treated as the authoritative description of what each recovery point represents.

---

# 22. Before Creating a Snapshot

Before taking a snapshot:

1. Verify the system is working.
2. Close unnecessary applications.
3. Confirm important services are healthy.
4. Record what the snapshot represents.
5. Avoid creating a snapshot of a known broken state unless intentionally preserving it for investigation.

---

# 23. Windows Pre-Snapshot Checks

Before creating a Windows snapshot:

```powershell
hostname
```

Check Wazuh if installed:

```powershell
Get-Service -Name Wazuh
```

Check Sysmon:

```powershell
Get-Service -Name Sysmon64
```

Check network:

```powershell
ipconfig
```

Optionally confirm Event Viewer is functioning.

---

# 24. Wazuh Pre-Snapshot Checks

Before creating a Wazuh server snapshot:

```bash
hostname
```

Check networking:

```bash
ip addr
```

Check disk:

```bash
df -h
```

Check memory:

```bash
free -h
```

Check services:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

---

# 25. Kali Pre-Snapshot Checks

Before creating a Kali snapshot:

```bash
hostname
```

```bash
ip addr
```

```bash
ip route
```

Check package status:

```bash
sudo apt update
```

Verify VMware Tools if relevant:

```bash
systemctl status open-vm-tools
```

---

# 26. When to Create a Snapshot

Good times to create snapshots include:

- After clean OS installation
- After updates
- After a major tool installation
- After successful SIEM integration
- Before a detection-engineering exercise
- Before changing networking
- Before testing complex security tools
- Before a potentially destructive lab
- Before major upgrades

---

# 27. When Not to Create a Snapshot

Do not create a snapshot simply because:

- A command was executed
- A minor setting changed
- A single note was updated
- A normal investigation was performed
- Nothing meaningful changed

Too many snapshots make recovery harder to understand.

---

# 28. Snapshot Decision Rule

Ask:

```text
"If this next step breaks the VM,
would I want to return to the exact state I have now?"
```

If:

```text
YES
```

create a snapshot.

If:

```text
NO
```

a snapshot is probably unnecessary.

---

# 29. Restoring a Snapshot

If a VM becomes unusable:

```text
Identify Last Known-Good Snapshot
        │
        ▼
Review What Will Be Lost
        │
        ▼
Restore Snapshot
        │
        ▼
Boot VM
        │
        ▼
Validate Services
        │
        ▼
Repeat Change Carefully
```

---

# 30. Important Restore Warning

Restoring a snapshot can remove changes made after the snapshot.

This may include:

- New files
- Updated configuration
- New logs
- Investigation evidence
- Installed software
- Wazuh alerts
- Package updates

Before restoring, decide whether anything needs to be preserved.

---

# 31. Preserve Evidence Before Rollback

If a lab exercise produced useful evidence, save it before restoring.

Examples:

```text
Screenshots
Investigation Reports
PCAP Files
Configuration Files
Detection Rules
Command History
Important Logs
```

These can be stored outside the VM or committed to the GitHub repository where appropriate.

---

# 32. GitHub as Recovery Documentation

The GitHub repository acts as a second layer of recovery.

The repo stores:

```text
Installation Steps
Commands
Configurations
Troubleshooting Notes
Architecture
Investigation Reports
Cheat Sheets
```

If a VM must eventually be rebuilt from scratch, the documentation should provide the steps required to recreate it.

---

# 33. Recovery Levels

The lab effectively has several recovery levels.

```text
Level 1
Restart Service

Level 2
Restart VM

Level 3
Fix Configuration

Level 4
Restore Snapshot

Level 5
Rebuild VM

Level 6
Rebuild Entire Lab
```

Always try the least destructive appropriate option first.

---

# 34. Level 1 — Restart a Service

If only one application is broken, restart only that service.

Windows example:

```powershell
Restart-Service -Name Wazuh
```

Linux example:

```bash
sudo systemctl restart wazuh-manager
```

Do not restore a snapshot for a simple service problem.

---

# 35. Level 2 — Restart the VM

If multiple services are affected or the system is unstable:

Windows:

```cmd
shutdown /r /t 0
```

Linux:

```bash
sudo reboot
```

Verify the issue after restart.

---

# 36. Level 3 — Repair Configuration

If the issue is caused by configuration:

```text
Identify incorrect setting
        │
        ▼
Correct configuration
        │
        ▼
Restart relevant service
        │
        ▼
Test
```

This is preferable to immediately restoring a snapshot because troubleshooting itself is valuable experience.

---

# 37. Level 4 — Restore Snapshot

Use a snapshot when:

- Configuration damage is extensive
- A tool installation has severely broken the environment
- Reversing all changes manually is inefficient
- A lab exercise was intentionally disposable
- A clean starting point is required

---

# 38. Level 5 — Rebuild a VM

A rebuild may be appropriate when:

- Snapshot state is unusable
- VM files are corrupted
- Architecture needs changing
- Major configuration mistakes accumulated
- A clean build would be faster than repair

The project documentation should make rebuilds repeatable.

---

# 39. Level 6 — Full Lab Rebuild

A complete lab rebuild should be rare.

The repository provides the rebuild sequence:

```text
VMware Fusion
      │
      ▼
Windows 11
      │
      ▼
Sysmon
      │
      ▼
Kali Linux
      │
      ▼
Ubuntu Server
      │
      ▼
Wazuh
      │
      ▼
Windows Agent
      │
      ▼
Sysmon Integration
```

---

# 40. Recommended Recovery Order

When something fails:

```text
1. Read the error
2. Check logs
3. Check service
4. Check networking
5. Check configuration
6. Restart affected service
7. Restart VM if required
8. Restore snapshot only if necessary
9. Rebuild only as a last resort
```

---

# 41. Do Not Use Snapshots to Avoid Learning

Snapshots should make experimentation safer.

They should not replace troubleshooting.

Bad workflow:

```text
Something Failed
      │
      ▼
Immediately Restore Snapshot
```

Better workflow:

```text
Something Failed
      │
      ▼
Investigate Why
      │
      ▼
Understand Failure
      │
      ▼
Attempt Repair
      │
      ▼
Restore Snapshot if Needed
```

---

# 42. Snapshot Before Detection Labs

Before a more advanced lab, create a known-good point.

For example:

```text
Windows + Sysmon + Wazuh Working
        │
        ▼
Create Snapshot
        │
        ▼
Run Authentication Lab
        │
        ▼
Generate Events
        │
        ▼
Investigate
```

This allows the exercise to be repeated later.

---

# 43. Snapshot Before Security Testing

Before controlled Kali testing:

```text
Validate Windows
        │
        ▼
Validate Wazuh
        │
        ▼
Create Snapshot
        │
        ▼
Start Kali
        │
        ▼
Generate Controlled Activity
```

This reduces risk if a test makes unwanted system changes.

---

# 44. Snapshot Before Wazuh Changes

Useful before:

- Custom Wazuh rules
- Decoder changes
- Indexer configuration
- Version upgrades
- Major agent configuration changes

Example:

```text
Wazuh Stable
      │
      ▼
Snapshot
      │
      ▼
Add Custom Detection
      │
      ▼
Test
```

---

# 45. Wazuh Data Consideration

Restoring the Wazuh VM can also restore its historical security data to an earlier point.

This means newer:

- Alerts
- Agent state
- Indexed events
- Dashboard changes

may disappear.

Preserve useful investigation evidence before rollback.

---

# 46. Windows Event Log Consideration

Restoring `BTL-WIN11` also restores the Windows Event Logs to the snapshot state.

Any events generated after that snapshot may disappear from the VM.

If they are useful for a project:

```text
Document First
Then Restore
```

---

# 47. Kali Snapshot Consideration

Kali is generally the easiest VM to treat as disposable.

A good approach is:

```text
Clean Snapshot
      │
      ▼
Temporary Tools
      │
      ▼
Exercise
      │
      ▼
Restore
```

This keeps Kali from accumulating unnecessary changes over time.

---

# 48. Snapshot Storage

Snapshots consume disk space.

The amount used depends on how much the VM changes after the snapshot.

Large changes such as:

- OS upgrades
- Large package installs
- PCAP captures
- Wazuh index growth
- Large downloads

can significantly increase snapshot storage usage.

---

# 49. Monitor Host Storage

Because the host stores all virtual machines and snapshots, free disk space should be monitored.

Avoid allowing the host drive to become nearly full.

VMs, snapshots, logs, and security data can grow quickly.

---

# 50. Avoid Snapshot Sprawl

Do not maintain dozens of unnecessary snapshots.

A cleaner structure is:

```text
Clean Baseline
Current Stable Baseline
Pre Major Change
```

Delete obsolete snapshots after confirming they are no longer required.

---

# 51. Suggested Long-Term Snapshot Plan

## `BTL-WIN11`

```text
00 - Clean Windows
01 - Sysmon Baseline
02 - Wazuh Integrated
03 - Stable SOC Endpoint
```

## `BTL-KALI`

```text
00 - Clean Kali
01 - Core Tools
```

Temporary snapshots can be created for specific labs and removed later.

## `BTL-WAZUH`

```text
00 - Clean Ubuntu Server
01 - Wazuh Installed
02 - Windows Agent Connected
03 - Stable SIEM Baseline
```

---

# 52. Configuration Backups

Important configuration files should also be copied/documented separately.

Examples:

## Sysmon

```text
C:\Tools\Sysmon\sysmonconfig.xml
```

## Wazuh Windows Agent

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

## Wazuh Server

Relevant custom rules and configuration should be preserved before major changes.

---

# 53. Secrets Must Not Be Backed Up to GitHub

Configuration backups may contain sensitive information.

Before committing anything:

Check for:

- Passwords
- API keys
- Private keys
- Certificates
- Tokens
- Credentials
- Session information

Replace sensitive information with placeholders.

Example:

```text
<WAZUH-SERVER-IP>
<WAZUH-ADMIN-PASSWORD>
<API-TOKEN>
```

---

# 54. Investigation Recovery

For each major investigation, preserve:

```text
Alert Information
        │
        ▼
Relevant Events
        │
        ▼
Timeline
        │
        ▼
Screenshots
        │
        ▼
Analyst Notes
        │
        ▼
Final Verdict
```

Once documented, the VM can safely be restored if required.

---

# 55. Recovery Validation

After restoring a snapshot, do not assume everything is working.

Validate the system.

---

## Windows Validation

```powershell
hostname
Get-Service -Name Sysmon64
Get-Service -Name Wazuh
ipconfig
```

---

## Wazuh Validation

```bash
hostname
ip addr
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

---

## Kali Validation

```bash
hostname
ip addr
ip route
```

---

# 56. Validate Cross-System Communication

After recovery, verify communication.

Windows to Wazuh:

```powershell
Test-NetConnection <WAZUH-SERVER-IP> -Port 1514
```

Dashboard:

```text
https://<WAZUH-SERVER-IP>
```

Check Wazuh Agent status:

```text
Active
```

---

# 57. DHCP Consideration After Restore

Because VMware NAT uses DHCP, restored VMs may receive different addresses.

Always verify:

Windows:

```cmd
ipconfig
```

Linux:

```bash
ip addr
```

Do not assume that an old documented IP address is still correct.

---

# 58. Post-Restore Checklist

After restoring:

- [ ] VM boots successfully
- [ ] Hostname is correct
- [ ] Network interface is active
- [ ] Correct IP address assigned
- [ ] Internet connectivity works
- [ ] DNS resolution works
- [ ] Required services are running
- [ ] Wazuh Dashboard loads
- [ ] Windows Agent connects
- [ ] Sysmon logs are available
- [ ] Expected configuration is present

---

# 59. Recovery Documentation

Whenever a snapshot is restored due to a meaningful failure, document:

```text
What failed?
Why was restoration chosen?
Which snapshot was used?
What data was lost?
Did recovery succeed?
What was learned?
```

This information can be added to:

```text
docs/09-troubleshooting.md
```

---

# 60. Recovery Example

Example scenario:

```text
Custom Wazuh Configuration Added
        │
        ▼
Wazuh Manager Fails
        │
        ▼
Check Logs
        │
        ▼
Configuration Error Identified
        │
        ▼
Attempt Repair
        │
        ▼
Still Broken
        │
        ▼
Preserve Useful Files
        │
        ▼
Restore Known-Good Snapshot
        │
        ▼
Validate Services
        │
        ▼
Apply Change Correctly
```

---

# 61. Snapshot vs Rebuild Decision

Use a snapshot when:

```text
A known-good snapshot exists
AND
the current state is not worth preserving
```

Rebuild when:

```text
No useful snapshot exists
OR
the VM architecture/configuration fundamentally needs to change
```

---

# 62. Recovery Mindset

The goal is not to preserve every possible state forever.

The goal is to maintain:

```text
Known-Good Baselines
+
Repeatable Documentation
+
Useful Investigation Evidence
```

That combination makes the lab resilient.

---

# 63. Current Recovery Architecture

```text
                        GitHub Documentation
                               │
                ┌──────────────┼──────────────┐
                │              │              │
                ▼              ▼              ▼
           BTL-WIN11       BTL-WAZUH       BTL-KALI
                │              │              │
                ▼              ▼              ▼
             Snapshot       Snapshot       Snapshot
                │              │              │
                ▼              ▼              ▼
          Known-Good       Known-Good      Clean Kali
           Endpoint          Server         Baseline
```

---

# 64. Recovery Principles

The lab follows these principles:

```text
1. Troubleshoot before restoring.
2. Snapshot before major changes.
3. Preserve evidence before rollback.
4. Keep snapshot names meaningful.
5. Avoid unnecessary snapshot sprawl.
6. Document known-good states.
7. Treat snapshots as recovery points, not backups.
8. Keep GitHub documentation sufficient for rebuilds.
9. Verify services after every restore.
10. Never store secrets in public recovery documentation.
```

---

# 65. Skills Practised

This recovery strategy develops experience with:

- Virtual machine lifecycle management
- Snapshot planning
- Rollback procedures
- Recovery validation
- Change management
- Configuration management
- Troubleshooting
- Evidence preservation
- Infrastructure documentation
- Disaster recovery concepts
- Security lab safety

---

# 66. Key Takeaway

A useful security lab should be easy to experiment with without becoming fragile.

Snapshots provide short-term recovery.

Documentation provides long-term recovery.

Together:

```text
Snapshots
    +
Documentation
    +
Configuration Knowledge
    =
Repeatable Lab
```

The ultimate goal is to reach a point where every VM could be deleted and rebuilt from the repository documentation if necessary.

---

# 67. Related Documentation

Architecture:

```text
docs/01-architecture.md
```

VMware Fusion:

```text
docs/02-vmware-fusion-setup.md
```

Windows 11:

```text
docs/03-windows-11-setup.md
```

Sysmon:

```text
docs/04-sysmon-setup.md
```

Kali Linux:

```text
docs/05-kali-linux-setup.md
```

Wazuh Server:

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

Troubleshooting:

```text
docs/09-troubleshooting.md
```

---

## References

- [VMware Fusion Documentation](https://techdocs.broadcom.com/us/en/vmware-cis/desktop-hypervisors/fusion-pro.html)
- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Microsoft Sysmon Documentation](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Kali Linux Documentation](https://www.kali.org/docs/)
