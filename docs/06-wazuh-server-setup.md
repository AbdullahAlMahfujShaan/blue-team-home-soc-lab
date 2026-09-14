# Wazuh Server Setup (`BTL-WAZUH`)

This document describes the installation and baseline configuration of the **Wazuh all-in-one server** used in the Blue Team Home SOC Lab.

`BTL-WAZUH` acts as the central security monitoring platform for the environment.

It receives telemetry from monitored endpoints, applies detection logic, stores security data, and provides the web interface used for SOC investigations.

---

# 1. Overview

The Wazuh server provides the central SIEM functionality for the lab.

The deployment includes:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard

The server is installed on an Ubuntu Server ARM64 virtual machine running in VMware Fusion.

---

# 2. Virtual Machine Configuration

The Wazuh server VM was configured as:

| Setting | Value |
|---|---|
| VM Name | `BTL-WAZUH` |
| Hostname | `btl-wazuh` |
| Operating System | Ubuntu Server |
| Architecture | ARM64 |
| vCPU | 4 |
| RAM | 8 GB |
| Disk | 80 GB |
| Firmware | UEFI |
| Secure Boot | Not Required |
| TPM | Not Required |
| Networking | VMware NAT |

---

# 3. Why Ubuntu Server ARM64?

The lab host uses Apple Silicon.

Apple Silicon uses the ARM64 architecture, so an ARM64-compatible Linux distribution was required.

Ubuntu Server was selected because it provides:

- ARM64 support
- Low resource overhead
- Stable package management
- SSH administration
- Compatibility with the Wazuh all-in-one deployment
- No unnecessary desktop environment

The server does not require a graphical Linux desktop because Wazuh is managed through:

```text
SSH
+
Web Browser
```

---

# 4. Server Role

Within the lab architecture:

```text
BTL-WIN11
    │
    │ Security Telemetry
    ▼
Wazuh Agent
    │
    ▼
BTL-WAZUH
    │
    ├── Wazuh Manager
    ├── Wazuh Indexer
    └── Wazuh Dashboard
```

The Wazuh server is responsible for centralising and analysing endpoint telemetry.

---

# 5. Ubuntu Server Installation

Create a new virtual machine in VMware Fusion using an ARM64 Ubuntu Server installer.

During installation:

1. Boot from the Ubuntu Server ARM64 installation media.
2. Select the preferred language.
3. Configure keyboard settings.
4. Configure networking.
5. Configure storage.
6. Create the local user account.
7. Configure the hostname.
8. Install OpenSSH Server.
9. Complete installation.
10. Reboot into Ubuntu Server.

---

# 6. Hostname

The server hostname was configured as:

```text
btl-wazuh
```

Verify:

```bash
hostname
```

Expected:

```text
btl-wazuh
```

Additional information:

```bash
hostnamectl
```

---

# 7. Local User

The primary local Ubuntu account used in the lab is:

```text
shaan
```

Credentials are intentionally excluded from this repository.

> Never commit passwords, private keys, API credentials, certificates, or recovery secrets to GitHub.

---

# 8. OpenSSH Server

OpenSSH Server was installed during Ubuntu setup.

This allows the Wazuh server to be administered remotely from the host or another authorised system.

Check the SSH service:

```bash
sudo systemctl status ssh
```

If required:

```bash
sudo systemctl enable --now ssh
```

---

# 9. Initial System Update

Refresh package metadata:

```bash
sudo apt update
```

Upgrade installed packages:

```bash
sudo apt full-upgrade -y
```

Reboot after major updates:

```bash
sudo reboot
```

---

# 10. Remove Unneeded Packages

After upgrading:

```bash
sudo apt autoremove -y
```

This removes packages that are no longer required.

---

# 11. Verify Operating System

Display operating system information:

```bash
cat /etc/os-release
```

Check architecture:

```bash
uname -m
```

Expected architecture:

```text
aarch64
```

or an equivalent ARM64 identifier.

---

# 12. Network Configuration

The server uses VMware NAT networking.

During the initial lab build, the server received an address similar to:

```text
172.16.106.134/24
```

The lab subnet was:

```text
172.16.106.0/24
```

The address was assigned dynamically and may change.

For reproducibility, documentation uses:

```text
<WAZUH-SERVER-IP>
```

instead of relying on a temporary DHCP address.

---

# 13. Network Interface

The primary network interface observed in the lab was:

```text
enp0s20
```

Check interface information:

```bash
ip addr
```

or:

```bash
ip a
```

Example:

```text
enp0s20
    inet 172.16.106.134/24
```

---

# 14. Verify Routing

Display routing information:

```bash
ip route
```

The output should include:

- Default route
- VMware NAT gateway
- Local subnet

---

# 15. Verify Internet Connectivity

Test external IP connectivity:

```bash
ping -c 4 1.1.1.1
```

Test DNS:

```bash
ping -c 4 example.com
```

DNS resolution can also be tested with:

```bash
getent hosts example.com
```

---

# 16. Pre-Wazuh Snapshot

Before installing Wazuh, a VMware snapshot was created.

The snapshot represents:

```text
00 - Clean Ubuntu Server
```

This provides a known-good recovery point containing:

- Fresh Ubuntu Server
- System updates
- OpenSSH
- Working networking
- No Wazuh installation yet

This is useful if the Wazuh installation needs to be repeated from a clean state.

---

# 17. Download the Wazuh Installation Script

The Wazuh quickstart deployment uses the official installation script.

Initially, attempting to run the installer resulted in:

```text
cannot locate wazuh-install.sh
```

The issue was resolved by explicitly downloading the script.

Run:

```bash
curl -L -o wazuh-install.sh https://packages.wazuh.com/4.14/wazuh-install.sh
```

Verify the file exists:

```bash
ls -l wazuh-install.sh
```

---

# 18. Install Wazuh All-in-One

Run the installer:

```bash
sudo bash ./wazuh-install.sh -a
```

The:

```text
-a
```

option performs an all-in-one installation.

This installs the major Wazuh components on the same server.

---

# 19. Wazuh Components Installed

The all-in-one deployment includes:

```text
Wazuh Manager
      │
      ▼
Wazuh Indexer
      │
      ▼
Wazuh Dashboard
```

Each component performs a different role.

---

# 20. Wazuh Manager

The Wazuh Manager receives and analyses endpoint security data.

Responsibilities include:

- Agent communication
- Log processing
- Event decoding
- Detection rules
- Alert generation
- Security analysis

Conceptually:

```text
Endpoint Telemetry
        │
        ▼
Wazuh Manager
        │
        ▼
Decoders / Rules
        │
        ▼
Alerts
```

---

# 21. Wazuh Indexer

The Wazuh Indexer stores and indexes security data.

Its role includes:

- Indexing events
- Storing alerts
- Supporting searches
- Providing data for investigations

Conceptually:

```text
Wazuh Alerts
      │
      ▼
Wazuh Indexer
      │
      ▼
Searchable Security Data
```

---

# 22. Wazuh Dashboard

The Wazuh Dashboard provides the browser-based user interface.

It is used for:

- Agent monitoring
- Threat hunting
- Alert investigation
- Event searches
- Security dashboards
- Rule analysis
- Endpoint visibility

---

# 23. Installation Warning

During installation, the installer displayed a warning similar to:

```text
The current system does not match recommended requirements
```

The installation continued and completed successfully.

The VM had been allocated:

```text
4 vCPU
8 GB RAM
80 GB Disk
```

which was sufficient for this small home lab deployment.

---

# 24. Wazuh Version

The installed Wazuh version was:

```text
4.14.7
```

The exact current version may differ when reproducing the lab in the future.

Always refer to the current Wazuh documentation before installing a newer deployment.

---

# 25. Installation Completion

After successful installation, the installer displayed the Wazuh Dashboard credentials.

These credentials were stored privately and are not included in this repository.

> Never commit Wazuh administrator passwords to source control.

---

# 26. Access the Wazuh Dashboard

From the Mac host, open a browser and navigate to:

```text
https://<WAZUH-SERVER-IP>
```

Example lab address during the initial build:

```text
https://172.16.106.134
```

Because the lab uses a self-signed certificate, the browser may display a certificate warning.

This is expected in the home lab environment.

---

# 27. Self-Signed Certificate Warning

The browser may show a warning because the Wazuh Dashboard uses a certificate not trusted by the local browser.

For a private lab:

```text
Browser Certificate Warning
        │
        ▼
Expected Self-Signed Certificate
```

This does not automatically indicate that the Wazuh server is broken.

In a production environment, certificate trust should be configured properly.

---

# 28. Verify Wazuh Services

Check the Wazuh Manager:

```bash
sudo systemctl status wazuh-manager
```

Check the Wazuh Indexer:

```bash
sudo systemctl status wazuh-indexer
```

Check the Wazuh Dashboard:

```bash
sudo systemctl status wazuh-dashboard
```

Each should report a running/active state.

---

# 29. Quick Service Check

A shorter check can use:

```bash
sudo systemctl is-active wazuh-manager
sudo systemctl is-active wazuh-indexer
sudo systemctl is-active wazuh-dashboard
```

Expected:

```text
active
active
active
```

---

# 30. Start a Wazuh Service

If required:

```bash
sudo systemctl start wazuh-manager
```

```bash
sudo systemctl start wazuh-indexer
```

```bash
sudo systemctl start wazuh-dashboard
```

---

# 31. Restart Wazuh Services

For troubleshooting:

```bash
sudo systemctl restart wazuh-manager
```

```bash
sudo systemctl restart wazuh-indexer
```

```bash
sudo systemctl restart wazuh-dashboard
```

Restart services only when required.

---

# 32. Enable Services at Boot

Verify that services are enabled:

```bash
sudo systemctl enable wazuh-manager
sudo systemctl enable wazuh-indexer
sudo systemctl enable wazuh-dashboard
```

---

# 33. Verify Listening Ports

Use:

```bash
sudo ss -tulnp
```

This displays listening TCP and UDP sockets.

Useful Wazuh-related ports include:

| Port | Protocol | Purpose |
|---:|---|---|
| `1514` | TCP | Agent communication |
| `1515` | TCP | Agent enrollment |
| `55000` | TCP | Wazuh API |
| `443` | TCP | Dashboard / HTTPS |

Exact behaviour depends on the Wazuh deployment and configuration.

---

# 34. Agent Communication Port

Wazuh agents commonly communicate with the manager using:

```text
TCP 1514
```

Conceptually:

```text
BTL-WIN11
    │
    │ TCP 1514
    ▼
BTL-WAZUH
```

---

# 35. Enrollment Port

Agent enrollment may use:

```text
TCP 1515
```

This supports the initial agent registration process.

---

# 36. API Port

The Wazuh API commonly uses:

```text
TCP 55000
```

The API supports programmatic management and integration functions.

---

# 37. Dashboard Connectivity Test

From another lab endpoint, connectivity can be tested using:

```powershell
Test-NetConnection <WAZUH-SERVER-IP> -Port 443
```

From Linux:

```bash
nc -vz <WAZUH-SERVER-IP> 443
```

---

# 38. Test Agent Communication Port

From Windows:

```powershell
Test-NetConnection <WAZUH-SERVER-IP> -Port 1514
```

Expected when reachable:

```text
TcpTestSucceeded : True
```

---

# 39. Server Resource Usage

Check memory:

```bash
free -h
```

Check disk:

```bash
df -h
```

Check CPU and processes:

```bash
top
```

or:

```bash
htop
```

if installed.

Because Wazuh runs several services, resource usage will be higher than a plain Ubuntu Server.

---

# 40. Check Disk Usage

The indexer stores security data, so disk usage should be monitored.

Run:

```bash
df -h
```

Pay particular attention to:

```text
/
```

As the lab generates more telemetry, indexed data will consume additional disk space.

---

# 41. Check Wazuh Manager Logs

Useful logs may be available under:

```text
/var/ossec/logs/
```

For example:

```bash
sudo ls -la /var/ossec/logs/
```

A key log is commonly:

```text
/var/ossec/logs/ossec.log
```

View recent entries:

```bash
sudo tail -n 100 /var/ossec/logs/ossec.log
```

Follow live entries:

```bash
sudo tail -f /var/ossec/logs/ossec.log
```

---

# 42. Check System Logs

For service problems, use `journalctl`.

Example:

```bash
sudo journalctl -u wazuh-manager
```

Recent entries:

```bash
sudo journalctl -u wazuh-manager -n 100
```

Follow live:

```bash
sudo journalctl -u wazuh-manager -f
```

---

# 43. Troubleshooting: Installer Not Found

Initial issue:

```text
cannot locate wazuh-install.sh
```

Resolution:

```bash
curl -L -o wazuh-install.sh https://packages.wazuh.com/4.14/wazuh-install.sh
```

Then:

```bash
sudo bash ./wazuh-install.sh -a
```

---

# 44. Troubleshooting: Dashboard Does Not Load

First verify the VM is powered on.

Check the server IP:

```bash
ip addr
```

Verify the dashboard service:

```bash
sudo systemctl status wazuh-dashboard
```

Verify a listening HTTPS service:

```bash
sudo ss -tulnp
```

Then try:

```text
https://<WAZUH-SERVER-IP>
```

---

# 45. Troubleshooting: IP Address Changed

Because DHCP is currently used, the server address may change after reboot.

Check:

```bash
ip addr
```

If the IP changes, Windows agents configured with the previous manager IP may fail to communicate.

A future improvement may include:

- Static IP configuration
- DHCP reservation
- Internal DNS

For the current lab, the DHCP setup remains sufficient as long as the server address is checked when required.

---

# 46. Troubleshooting: Agent Cannot Connect

Verify Wazuh is running:

```bash
sudo systemctl status wazuh-manager
```

Verify port 1514 is listening:

```bash
sudo ss -tulnp | grep 1514
```

From the Windows endpoint:

```powershell
Test-NetConnection <WAZUH-SERVER-IP> -Port 1514
```

Then inspect:

```text
C:\Program Files (x86)\ossec-agent\ossec.log
```

on the Windows endpoint.

---

# 47. Optional Repository Hardening

After installation, Wazuh packages may remain connected to the package repository.

To reduce the risk of accidental upgrades in a stable lab environment, the repository can optionally be disabled.

Example:

```bash
sudo sed -i "s/^deb /#deb /" /etc/apt/sources.list.d/wazuh.list
```

Then:

```bash
sudo apt update
```

This is optional.

If future upgrades are planned, the repository can be re-enabled intentionally.

---

# 48. Why Avoid Unplanned Wazuh Upgrades?

A security lab benefits from a stable baseline.

Unexpected upgrades may introduce:

- Configuration changes
- Version mismatches
- Agent compatibility problems
- Dashboard changes
- Rule changes
- Troubleshooting complexity

A better workflow is:

```text
Stable Working Lab
       │
       ▼
Create Snapshot / Backup
       │
       ▼
Plan Upgrade
       │
       ▼
Review Release Notes
       │
       ▼
Upgrade Intentionally
```

---

# 49. Wazuh Architecture

The all-in-one architecture can be represented as:

```text
                     BTL-WAZUH
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
 Wazuh Manager      Wazuh Indexer    Wazuh Dashboard
        │                ▲                │
        │                │                │
        └──── Alerts ────┘                │
                         │                │
                         └────────────────┘
```

---

# 50. Complete Telemetry Flow

Once a Windows agent is connected:

```text
BTL-WIN11
    │
    │
    ├── Windows Security Logs
    │
    └── Sysmon Events
          │
          ▼
      Wazuh Agent
          │
          ▼
      TCP 1514
          │
          ▼
      BTL-WAZUH
          │
          ▼
      Wazuh Manager
          │
          ▼
   Decoders / Rules
          │
          ▼
        Alerts
          │
          ▼
      Wazuh Indexer
          │
          ▼
     Wazuh Dashboard
          │
          ▼
    SOC Investigation
```

---

# 51. Wazuh Detection Model

Wazuh does not treat every received event as an alert.

The basic model is:

```text
Raw Event
   │
   ▼
Decoder
   │
   ▼
Rule Evaluation
   │
   ├──── Match ────▶ Alert
   │
   └──── No Match ─▶ No Alert
```

This distinction becomes important during threat hunting and investigation.

---

# 52. Raw Telemetry vs Alerting

For example:

```text
whoami.exe
```

may generate:

```text
Sysmon Event ID 1
```

but the event may not necessarily generate a Wazuh alert.

This demonstrates:

> Raw telemetry and SIEM alerts are not the same thing.

The Wazuh server receives and analyses events, but only matching detection logic becomes an alert.

---

# 53. Dashboard Use

The Wazuh Dashboard is used for:

```text
Agent Status
     │
     ▼
Alert Search
     │
     ▼
Threat Hunting
     │
     ▼
Event Analysis
     │
     ▼
SOC Investigation
```

This becomes the main analyst interface for the home SOC lab.

---

# 54. Initial Server Validation

After Wazuh installation:

- [x] Ubuntu Server running
- [x] Network connectivity verified
- [x] OpenSSH working
- [x] Wazuh installer downloaded
- [x] Wazuh installation completed
- [x] Wazuh Manager running
- [x] Wazuh Indexer running
- [x] Wazuh Dashboard running
- [x] Dashboard accessible from host browser
- [x] Self-signed certificate warning observed
- [x] Admin credentials stored privately
- [x] Server ready for endpoint enrollment

---

# 55. Security Considerations

The Wazuh server contains sensitive security information.

Examples include:

- Endpoint hostnames
- Usernames
- Security alerts
- Process command lines
- Network addresses
- Authentication events
- Security configurations

Avoid exposing the Wazuh Dashboard directly to the public Internet.

The current lab uses private VMware NAT networking.

---

# 56. Credential Handling

Never store Wazuh credentials in:

```text
README.md
GitHub commits
Screenshots
Shell history intentionally shared publicly
Public issue trackers
Documentation examples
```

Use placeholders:

```text
<WAZUH-ADMIN-USER>
<WAZUH-ADMIN-PASSWORD>
<WAZUH-SERVER-IP>
```

where appropriate.

---

# 57. VM Shutdown

When the SIEM is not required:

```bash
sudo poweroff
```

The Wazuh server does not need to remain powered on while studying theory or editing documentation.

---

# 58. Recommended Startup Order

For a normal investigation session:

```text
1. Start BTL-WAZUH
        │
        ▼
2. Wait for Ubuntu to boot
        │
        ▼
3. Verify Wazuh services
        │
        ▼
4. Open Wazuh Dashboard
        │
        ▼
5. Start BTL-WIN11
        │
        ▼
6. Verify Agent becomes Active
```

Start `BTL-KALI` only if the exercise requires controlled test traffic.

---

# 59. Useful Server Commands

## Hostname

```bash
hostname
```

## Network

```bash
ip addr
```

## Routing

```bash
ip route
```

## Memory

```bash
free -h
```

## Disk

```bash
df -h
```

## Listening Ports

```bash
sudo ss -tulnp
```

## Wazuh Manager

```bash
sudo systemctl status wazuh-manager
```

## Wazuh Indexer

```bash
sudo systemctl status wazuh-indexer
```

## Wazuh Dashboard

```bash
sudo systemctl status wazuh-dashboard
```

---

# 60. Skills Practised

This stage provided hands-on experience with:

- Ubuntu Server deployment
- ARM64 Linux virtualisation
- Linux administration
- OpenSSH
- Linux networking
- Package management
- Server resource management
- Wazuh installation
- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- SIEM architecture
- Security event storage
- Detection pipelines
- Service management with `systemctl`
- Troubleshooting with `journalctl`
- TCP port validation
- Security platform deployment

---

# 61. Key Takeaway

The Wazuh server transforms isolated endpoint logs into a central security monitoring environment.

Without the SIEM:

```text
Windows Event
      │
      ▼
Local Event Viewer
```

With Wazuh:

```text
Endpoint Activity
      │
      ▼
Windows / Sysmon
      │
      ▼
Wazuh Agent
      │
      ▼
Wazuh Manager
      │
      ▼
Detection Logic
      │
      ▼
Searchable Alerts
      │
      ▼
SOC Analyst
```

This centralisation makes it possible to practise real SOC investigation workflows.

---

# 62. Next Step

With the Wazuh server operational, the next stage is to deploy the Wazuh Agent on the Windows endpoint.

Continue with:

```text
docs/07-wazuh-windows-agent.md
```

After the agent is active, Sysmon telemetry can be integrated using:

```text
docs/08-sysmon-to-wazuh.md
```

---

## References

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Wazuh Quickstart](https://documentation.wazuh.com/current/quickstart.html)
- [Wazuh Agent Enrollment](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/index.html)
- [Wazuh Installation Packages](https://documentation.wazuh.com/current/installation-guide/packages-list.html)
- [Ubuntu Server](https://ubuntu.com/server)
