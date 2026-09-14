# Kali Linux Setup (`BTL-KALI`)

This document describes the installation and baseline configuration of the Kali Linux virtual machine used in the **Blue Team Home SOC Lab**.

`BTL-KALI` acts as the lab's security testing, traffic-generation, and network-analysis workstation.

---

# 1. Overview

Kali Linux is used in this environment to generate controlled security activity that can later be observed and investigated from the defensive side.

The VM is intended for:

- Network reconnaissance
- Traffic generation
- DNS testing
- Connectivity testing
- Authentication testing
- Network analysis
- Controlled adversary simulation
- Detection validation
- Future PCAP generation
- Blue Team investigation exercises

The VM is not the central monitoring server.

That role belongs to:

```text
BTL-WAZUH
```

Kali instead provides a controlled system from which activity can be generated against other machines inside the authorised home lab.

---

# 2. Virtual Machine Configuration

The Kali VM was configured with:

| Setting | Value |
|---|---|
| VM Name | `BTL-KALI` |
| Hostname | `btl-kali` |
| Operating System | Kali Linux |
| Architecture | ARM64 |
| Guest Type | Debian 13.x 64-bit ARM |
| Desktop Environment | Xfce |
| vCPU | 2 |
| RAM | 4 GB |
| Disk | ~50–60 GB |
| Firmware | UEFI |
| Secure Boot | Disabled |
| TPM | Not Required |
| Networking | VMware NAT |

---

# 3. Why Kali ARM64?

The host system uses Apple Silicon.

Apple Silicon uses the:

```text
ARM64
```

architecture.

For this reason, the ARM64 / Apple Silicon version of Kali Linux was selected instead of a traditional x86-64 image.

This allows Kali to run natively and efficiently through VMware Fusion.

---

# 4. Installation Media

Use the Kali Linux installer intended for:

```text
Apple Silicon / ARM64
```

Official downloads:

https://www.kali.org/get-kali/

Kali also provides VMware-specific guidance for Apple Silicon:

https://www.kali.org/docs/virtualization/install-vmware-silicon-host/

---

# 5. VMware Guest Type

During VM creation, the guest operating system type was configured as:

```text
Debian 13.x 64-bit Arm
```

Kali Linux is Debian-based, so this provides an appropriate VMware guest profile for the ARM64 installation.

---

# 6. Firmware Configuration

The Kali VM uses:

```text
UEFI
```

Secure Boot was not enabled for this lab VM.

Configuration:

```text
UEFI: Enabled
Secure Boot: Disabled
TPM: Not Required
```

---

# 7. Kali Installation

Create a new virtual machine in VMware Fusion and attach the ARM64 Kali installer.

During installation:

1. Boot from the Kali ARM64 installation media.
2. Select the preferred language.
3. Select keyboard layout.
4. Configure network settings.
5. Set the hostname.
6. Create the user account.
7. Configure storage.
8. Select the desktop environment.
9. Complete installation.
10. Reboot into the installed system.

---

# 8. Hostname

The hostname configured for the system is:

```text
btl-kali
```

Verify using:

```bash
hostname
```

Expected output:

```text
btl-kali
```

Additional hostname information can be displayed with:

```bash
hostnamectl
```

---

# 9. Lab User

The local Kali account used in the lab is:

```text
shaan
```

Credentials are intentionally excluded from the repository.

> Never commit passwords, SSH private keys, API tokens, credentials, or other secrets to a public GitHub repository.

---

# 10. Desktop Environment

The selected desktop environment is:

```text
Xfce
```

Xfce provides a lightweight graphical desktop suitable for a virtual machine.

It provides sufficient functionality while keeping resource consumption lower than heavier desktop environments.

---

# 11. Initial System Update

After installation, the package repository information was refreshed:

```bash
sudo apt update
```

The system was then upgraded:

```bash
sudo apt full-upgrade -y
```

After the upgrade, reboot:

```bash
sudo reboot
```

---

# 12. Remove Unneeded Packages

After updating, unused dependencies can be removed:

```bash
sudo apt autoremove -y
```

This helps keep the VM clean after package upgrades.

---

# 13. Verify Operating System

Display Kali release information:

```bash
cat /etc/os-release
```

Architecture can be verified using:

```bash
uname -m
```

Expected architecture:

```text
aarch64
```

or another ARM64-equivalent identifier depending on the tool.

---

# 14. VMware Tools

VMware guest integration packages were installed to improve compatibility with VMware Fusion.

Install:

```bash
sudo apt install open-vm-tools open-vm-tools-desktop -y
```

---

# 15. Why `open-vm-tools`?

`open-vm-tools` provides integration between the Linux guest and VMware.

It can improve:

- Display handling
- Guest integration
- Clock synchronisation
- Clipboard behaviour
- VMware compatibility

The desktop package adds additional functionality for graphical Linux guests.

---

# 16. Reboot After VMware Tools

After installation:

```bash
sudo reboot
```

This ensures the guest integration services start correctly.

---

# 17. Verify VMware Tools

Check the service:

```bash
systemctl status open-vm-tools
```

Depending on package/service naming, VMware Tools functionality can also be confirmed by checking that the VM behaves correctly inside VMware Fusion.

---

# 18. Network Configuration

The Kali VM uses VMware NAT networking.

During the initial lab setup, Kali received an address similar to:

```text
172.16.106.132/24
```

This address is dynamically assigned and may change.

The lab network used during the build was:

```text
172.16.106.0/24
```

---

# 19. Network Interface

The primary interface observed during the lab was:

```text
eth0
```

Display interface information:

```bash
ip addr
```

A shorter command is:

```bash
ip a
```

Example:

```text
eth0
    inet 172.16.106.132/24
```

---

# 20. Check Routing

Display the routing table:

```bash
ip route
```

A typical result includes:

```text
default via <VMWARE-NAT-GATEWAY>
```

and the local subnet:

```text
172.16.106.0/24
```

---

# 21. Verify Connectivity

Test the local TCP/IP stack:

```bash
ping -c 4 127.0.0.1
```

Test the VMware NAT gateway if known:

```bash
ping -c 4 <GATEWAY-IP>
```

Test Internet connectivity:

```bash
ping -c 4 1.1.1.1
```

Test DNS resolution:

```bash
ping -c 4 example.com
```

---

# 22. Check DNS Resolution

Useful commands include:

```bash
getent hosts example.com
```

or:

```bash
nslookup example.com
```

if `nslookup` is installed.

Another option is:

```bash
dig example.com
```

if the DNS utilities package is available.

---

# 23. Current Lab Network

The three primary VMs communicate over VMware NAT.

Example addresses observed during the initial build:

```text
BTL-WIN11  → 172.16.106.131
BTL-KALI   → 172.16.106.132
BTL-WAZUH  → 172.16.106.134
```

Conceptually:

```text
               VMware NAT
                   │
        ┌──────────┼──────────┐
        │          │          │
        ▼          ▼          ▼
   BTL-WIN11   BTL-WAZUH   BTL-KALI
```

Because DHCP is used, these addresses may change.

---

# 24. Verify Communication with Other Lab Systems

When the other systems are powered on, Kali can test basic connectivity.

For example:

```bash
ping -c 4 <WINDOWS-ENDPOINT-IP>
```

and:

```bash
ping -c 4 <WAZUH-SERVER-IP>
```

Whether ICMP succeeds may depend on the destination firewall configuration.

A failed ping does not automatically mean the system is unreachable.

---

# 25. TCP Connectivity Testing

A better test for a specific service is often a TCP connection.

For example:

```bash
nc -vz <WAZUH-SERVER-IP> 1514
```

or:

```bash
nc -vz <WAZUH-SERVER-IP> 443
```

depending on the service being tested.

This helps distinguish:

```text
Host reachable
```

from:

```text
Specific service reachable
```

---

# 26. Useful Baseline Commands

The following commands are useful when working with Kali:

## Current User

```bash
whoami
```

---

## Hostname

```bash
hostname
```

---

## IP Addresses

```bash
ip addr
```

---

## Routing Table

```bash
ip route
```

---

## Running Processes

```bash
ps aux
```

---

## Listening Ports

```bash
ss -tulnp
```

---

## Disk Usage

```bash
df -h
```

---

## Memory Usage

```bash
free -h
```

---

## System Information

```bash
uname -a
```

---

# 27. Kali's Role in a Blue Team Lab

Although Kali Linux is commonly associated with offensive security, in this project it is mainly used to help create **controlled, observable activity**.

For example:

```text
BTL-KALI
    │
    │ Controlled Test Activity
    ▼
BTL-WIN11
    │
    │ Endpoint Telemetry
    ▼
Sysmon / Windows Logs
    │
    ▼
Wazuh Agent
    │
    ▼
BTL-WAZUH
    │
    ▼
SOC Investigation
```

This lets the lab practise both sides of an investigation without using external systems.

---

# 28. Controlled Activity Generation

Future exercises may use Kali to generate:

- Network scans
- DNS queries
- TCP connections
- Failed authentication attempts
- Web requests
- Port discovery
- Traffic suitable for PCAP analysis
- Other authorised test activity

The purpose is not simply to perform an attack.

The main objective is to answer:

```text
What telemetry did the activity create?
        │
        ▼
Was it detected?
        │
        ▼
How would an analyst investigate it?
        │
        ▼
What evidence confirms the activity?
```

---

# 29. Authorisation Boundary

All testing must remain inside systems owned and controlled within the lab.

The environment is intended for:

- Education
- Defensive security
- Detection validation
- Network analysis
- SOC training
- Incident investigation

Do not direct testing toward systems without explicit authorisation.

---

# 30. Kali Is Not the SIEM

Kali is intentionally separated from the monitoring infrastructure.

```text
BTL-KALI
    │
    │ Generates Activity
    ▼
BTL-WIN11
```

while:

```text
BTL-WAZUH
    │
    │ Collects / Analyses
    ▼
Security Telemetry
```

This separation provides a clearer lab architecture.

---

# 31. Kali and Windows Testing

A future controlled scenario might look like:

```text
BTL-KALI
    │
    │ Authentication Attempt
    ▼
BTL-WIN11
    │
    ▼
Windows Security Log
    │
    ├── 4625
    ├── 4625
    ├── 4625
    │
    ▼
Wazuh
    │
    ▼
SOC Investigation
```

This allows authentication telemetry to be studied in a repeatable environment.

---

# 32. Kali and Network Telemetry

Another future exercise might generate a connection from Kali to Windows:

```text
BTL-KALI
    │
    │ TCP Traffic
    ▼
BTL-WIN11
    │
    ▼
Network Telemetry
    │
    ▼
Sysmon / PCAP
    │
    ▼
Analyst Investigation
```

This creates a practical way to study network evidence.

---

# 33. Future Wireshark Usage

Kali can also be used for future network-analysis exercises.

Potential workflow:

```text
Generate Traffic
      │
      ▼
Capture Traffic
      │
      ▼
Open PCAP
      │
      ▼
Analyse with Wireshark
      │
      ▼
Identify:
- Source
- Destination
- Protocol
- Ports
- DNS
- TCP sessions
- Suspicious patterns
```

This will be added later as the network-forensics section of the lab develops.

---

# 34. ARM64 Tool Compatibility

Because the Kali VM runs ARM64, some tools may behave differently from traditional x86-64 Kali systems.

Many standard networking and security utilities work normally, but some:

- Binaries
- Exploits
- Precompiled tools
- Malware-analysis utilities
- Third-party packages

may only be available for x86-64.

Always verify architecture compatibility before installing third-party software.

---

# 35. Do Not Force x86 Tools Into the ARM VM

If a tool is designed specifically for:

```text
x86
```

or:

```text
x86-64 / AMD64
```

do not assume that it will work correctly in the ARM64 environment.

Instead:

1. Check whether an ARM64 version exists.
2. Check whether source code can be compiled for ARM64.
3. Use a hosted lab if appropriate.
4. Use a separate x86-64 system later if necessary.

---

# 36. Snapshot

After completing:

- Kali installation
- System updates
- VMware Tools installation
- Basic network validation

a snapshot was created.

The snapshot represents:

```text
00 - Clean Kali
```

This provides a clean recovery point before security-testing tools or lab-specific changes are introduced.

---

# 37. Why a Clean Snapshot Matters

A clean snapshot allows the system to be restored if future exercises:

- Break configuration
- Install unwanted packages
- Modify networking
- Cause tool conflicts
- Produce an unstable state

Workflow:

```text
Clean Kali Snapshot
        │
        ▼
Install / Test Tool
        │
        ▼
Perform Lab
        │
    ┌───┴────┐
    │        │
 Success   Problem
    │        │
    ▼        ▼
Continue   Restore Snapshot
```

---

# 38. Shutting Down Kali

When Kali is not required:

```bash
sudo poweroff
```

Alternatively:

```bash
sudo shutdown now
```

The VM does not need to remain powered on during normal Wazuh or Windows-only investigations.

---

# 39. When Kali Should Be Powered On

Example lab usage:

```text
Studying Theory
      │
      ▼
Kali OFF
```

```text
Investigating Existing Wazuh Alert
      │
      ▼
Kali OFF
```

```text
Generating Network Activity
      │
      ▼
Kali ON
```

```text
Testing Detection Logic
      │
      ▼
Kali ON
```

Only run the VM when required to conserve host resources.

---

# 40. Basic Package Management

Refresh package information:

```bash
sudo apt update
```

Upgrade packages:

```bash
sudo apt full-upgrade -y
```

Install a package:

```bash
sudo apt install <package-name>
```

Remove a package:

```bash
sudo apt remove <package-name>
```

Remove unused dependencies:

```bash
sudo apt autoremove -y
```

---

# 41. Useful Network Tools

Several standard Kali tools will become useful as the lab expands.

Examples include:

```text
ping
ip
ss
curl
wget
nc
dig
nslookup
traceroute
tcpdump
Wireshark
Nmap
```

These tools should be learned individually rather than treated as a single "hacking toolkit."

The focus should remain on understanding the network activity they generate.

---

# 42. Example Network Investigation Mindset

If Kali runs:

```bash
curl https://example.com
```

the Blue Team questions are:

```text
Which DNS query occurred?
        │
        ▼
Which IP address was resolved?
        │
        ▼
Which TCP connection was created?
        │
        ▼
Which destination port was used?
        │
        ▼
Can the activity be identified in a PCAP?
```

This approach connects networking knowledge with SOC investigation skills.

---

# 43. Baseline Validation Checklist

After setup, verify:

- [x] Kali Linux ARM64 installed
- [x] Hostname configured as `btl-kali`
- [x] Xfce desktop installed
- [x] User account created
- [x] UEFI working
- [x] System updated
- [x] `open-vm-tools` installed
- [x] `open-vm-tools-desktop` installed
- [x] VMware NAT networking working
- [x] IP address assigned
- [x] Internet connectivity working
- [x] DNS resolution working
- [x] Clean snapshot created

---

# 44. Current Lab Role

The architecture currently looks like:

```text
                         BTL-KALI
                            │
                            │
                    Controlled Activity
                            │
                            ▼
                        BTL-WIN11
                            │
                   ┌────────┴────────┐
                   │                 │
                   ▼                 ▼
            Windows Events        Sysmon
                   │                 │
                   └────────┬────────┘
                            │
                            ▼
                       Wazuh Agent
                            │
                            ▼
                       BTL-WAZUH
                            │
                            ▼
                    SOC Investigation
```

---

# 45. Skills Practised

This stage provided hands-on experience with:

- Kali Linux installation
- ARM64 virtualisation
- Linux administration
- Linux package management
- VMware guest tools
- Linux networking
- IP addressing
- Routing
- DNS
- Connectivity testing
- Linux command-line usage
- Snapshot management
- Security lab architecture
- Controlled testing methodology

---

# 46. Key Takeaway

The purpose of Kali in this lab is not simply to run offensive tools.

Its main purpose is to provide a controlled source of security-relevant activity.

The learning workflow is:

```text
Generate Activity
       │
       ▼
Observe Telemetry
       │
       ▼
Detect
       │
       ▼
Investigate
       │
       ▼
Understand What Happened
```

This keeps the lab focused on **Blue Team and SOC skills**.

---

# 47. Next Step

The next infrastructure component is the central Wazuh SIEM server.

Continue with:

```text
docs/06-wazuh-server-setup.md
```

After the Wazuh server is operational, the Windows endpoint can be enrolled using:

```text
docs/07-wazuh-windows-agent.md
```

---

## References

- [Kali Linux Downloads](https://www.kali.org/get-kali/)
- [Kali Linux Documentation](https://www.kali.org/docs/)
- [Kali Linux on VMware Apple Silicon](https://www.kali.org/docs/virtualization/install-vmware-silicon-host/)
- [VMware open-vm-tools Documentation](https://github.com/vmware/open-vm-tools)
