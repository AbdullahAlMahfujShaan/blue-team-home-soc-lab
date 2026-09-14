# Linux Command Cheat Sheet for Blue Team / SOC Analysis

This cheat sheet focuses on Linux commands that are useful for:

- SOC analysis
- Incident response
- Log review
- Network troubleshooting
- File analysis
- Process investigation
- Service troubleshooting
- Evidence collection

The goal is not to memorise every Linux command.

The goal is to understand:

```text
What do I need to investigate?
        ↓
Which command answers that question?
```

---

# 1. Navigation

## Show Current Directory

```bash
pwd
```

Purpose:

```text
Displays the current working directory.
```

Example output:

```text
/home/shaan
```

Use when:

- Confirming your location
- Troubleshooting missing files
- Avoiding commands in the wrong directory

---

## List Files

```bash
ls
```

Detailed view:

```bash
ls -l
```

Include hidden files:

```bash
ls -la
```

Human-readable sizes:

```bash
ls -lah
```

Useful columns include:

```text
Permissions
Owner
Group
Size
Timestamp
Filename
```

---

## Change Directory

```bash
cd /var/log
```

Go home:

```bash
cd ~
```

Go up one directory:

```bash
cd ..
```

Previous directory:

```bash
cd -
```

---

# 2. File Viewing

## Display Entire File

```bash
cat file.txt
```

Useful for small files.

Avoid using `cat` on very large logs.

---

## Read Large Files

```bash
less file.txt
```

Useful controls:

```text
Space     Next page
b         Previous page
/word     Search
n         Next match
q         Quit
```

For large logs, `less` is usually better than `cat`.

---

## First Lines

```bash
head file.txt
```

First 20 lines:

```bash
head -n 20 file.txt
```

---

## Last Lines

```bash
tail file.txt
```

Last 50:

```bash
tail -n 50 file.txt
```

---

## Follow a Log in Real Time

```bash
tail -f /var/log/example.log
```

Useful when testing a service or generating activity.

Exit:

```text
Ctrl+C
```

---

# 3. Searching Text with `grep`

`grep` is one of the most useful Linux commands for analysts.

## Basic Search

```bash
grep "failed" logfile.log
```

---

## Case-Insensitive Search

```bash
grep -i "failed" logfile.log
```

---

## Show Line Numbers

```bash
grep -n "failed" logfile.log
```

---

## Recursive Search

```bash
grep -R "failed" /var/log/
```

Case-insensitive with line numbers:

```bash
grep -Rni "failed" /var/log/
```

---

## Search Multiple Terms

```bash
grep -E "failed|error|denied" logfile.log
```

---

## Exclude a Term

```bash
grep -v "success" logfile.log
```

---

## Count Matches

```bash
grep -c "failed" logfile.log
```

---

## Show Context Around Match

Three lines before and after:

```bash
grep -C 3 "failed" logfile.log
```

Before:

```bash
grep -B 3 "failed" logfile.log
```

After:

```bash
grep -A 3 "failed" logfile.log
```

---

# 4. Pipes

The pipe:

```text
|
```

passes the output of one command into another.

Example:

```bash
ps aux | grep ssh
```

Meaning:

```text
List processes
      ↓
Search output for "ssh"
```

This concept is extremely important in Linux investigations.

---

# 5. Output Redirection

Write output to a file:

```bash
command > output.txt
```

Append instead:

```bash
command >> output.txt
```

Example:

```bash
ip addr > network-state.txt
```

Append another command:

```bash
ip route >> network-state.txt
```

---

# 6. File and Directory Discovery

## Find Files

```bash
find /tmp -type f
```

Search by filename:

```bash
find / -name "example.txt" 2>/dev/null
```

Case-insensitive:

```bash
find / -iname "*.log" 2>/dev/null
```

---

## Find Recently Modified Files

Modified in the last 24 hours:

```bash
find /tmp -type f -mtime -1
```

Modified in the last 60 minutes:

```bash
find /tmp -type f -mmin -60
```

This can be useful during incident response.

---

## Find Executable Files

```bash
find /tmp -type f -executable
```

---

## Find Large Files

```bash
find / -type f -size +100M 2>/dev/null
```

---

# 7. File Metadata

## Identify File Type

```bash
file suspicious_file
```

Example:

```text
ELF 64-bit LSB executable
```

This is useful because file extensions cannot always be trusted.

---

## Show Metadata

```bash
stat suspicious_file
```

Useful fields:

```text
Size
Permissions
Owner
Access time
Modify time
Change time
```

---

# 8. Hashing

## SHA256

```bash
sha256sum suspicious_file
```

Example:

```text
abc123... suspicious_file
```

---

## SHA1

```bash
sha1sum suspicious_file
```

---

## MD5

```bash
md5sum suspicious_file
```

For investigation work, SHA256 is generally preferred.

---

# 9. File Permissions

## View Permissions

```bash
ls -l
```

Example:

```text
-rwxr-xr--
```

Structure:

```text
Owner
Group
Others
```

Permission letters:

```text
r = read
w = write
x = execute
```

---

## Numeric Permissions

```text
4 = read
2 = write
1 = execute
```

Examples:

```text
755
644
600
```

---

## Change Permissions

```bash
chmod 600 file.txt
```

Use carefully during investigations because modifying permissions changes evidence.

---

# 10. Ownership

## Change Owner

```bash
sudo chown user:user file.txt
```

Again:

> Avoid changing evidence unless necessary.

---

# 11. Processes

## List Processes

```bash
ps aux
```

Important fields:

```text
USER
PID
CPU
MEM
COMMAND
```

---

## Search for a Process

```bash
ps aux | grep ssh
```

Better option:

```bash
pgrep -af ssh
```

---

## Process Tree

```bash
pstree
```

Include PIDs:

```bash
pstree -p
```

Useful for understanding process relationships.

---

# 12. Real-Time Process Monitoring

```bash
top
```

If installed:

```bash
htop
```

Useful for:

- CPU usage
- Memory usage
- Suspicious processes
- Resource problems

---

# 13. Process Details

## Inspect Process Directory

```bash
ls -la /proc/<PID>/
```

Example:

```bash
ls -la /proc/1234/
```

---

## Command Line

```bash
cat /proc/<PID>/cmdline
```

More readable:

```bash
tr '\0' ' ' < /proc/<PID>/cmdline
```

---

## Executable Path

```bash
readlink -f /proc/<PID>/exe
```

---

## Current Working Directory

```bash
readlink -f /proc/<PID>/cwd
```

---

# 14. Kill a Process

Graceful termination:

```bash
kill <PID>
```

Force:

```bash
kill -9 <PID>
```

In incident response:

> Do not kill a suspicious process before collecting necessary evidence unless containment urgency requires it.

---

# 15. Network Interfaces

## Show Interfaces

```bash
ip addr
```

Short form:

```bash
ip a
```

Look for:

```text
Interface
IPv4 address
IPv6 address
MAC address
State
```

---

# 16. Routing Table

```bash
ip route
```

Example:

```text
default via 172.16.106.2 dev eth0
```

Useful when troubleshooting connectivity.

---

# 17. ARP / Neighbour Table

```bash
ip neigh
```

This shows known IP-to-MAC mappings.

---

# 18. Listening Ports and Connections

One of the most useful commands:

```bash
ss -tulnp
```

Meaning:

```text
-t TCP
-u UDP
-l Listening
-n Numeric addresses
-p Process information
```

---

## All TCP Connections

```bash
ss -tnp
```

---

## Listening TCP Only

```bash
ss -ltnp
```

---

## Search for a Port

```bash
ss -tulnp | grep 22
```

---

# 19. `netstat`

Older systems may use:

```bash
netstat -tulnp
```

`ss` is generally preferred on modern Linux systems.

---

# 20. Test Connectivity

## Ping

```bash
ping 8.8.8.8
```

Stop:

```text
Ctrl+C
```

---

## Limited Ping

```bash
ping -c 4 8.8.8.8
```

---

# 21. DNS Investigation

## Resolve Domain

```bash
getent hosts example.com
```

---

## `dig`

```bash
dig example.com
```

Short answer:

```bash
dig +short example.com
```

---

## `nslookup`

```bash
nslookup example.com
```

---

# 22. Test TCP Ports

With Netcat:

```bash
nc -vz <IP> <PORT>
```

Example:

```bash
nc -vz 172.16.106.134 1514
```

Successful output may include:

```text
succeeded
```

Useful for testing whether a service port is reachable.

---

# 23. HTTP Requests

## `curl`

```bash
curl https://example.com
```

Show headers:

```bash
curl -I https://example.com
```

Verbose:

```bash
curl -v https://example.com
```

---

## Ignore Certificate Verification

```bash
curl -k https://localhost
```

Useful for internal self-signed lab certificates.

Use carefully outside a lab environment.

---

# 24. Download Files

## `curl`

```bash
curl -L -o file.txt https://example.com/file.txt
```

---

## `wget`

```bash
wget https://example.com/file.txt
```

---

# 25. Services

Linux commonly uses `systemd`.

## Check Service

```bash
systemctl status ssh
```

---

## Start

```bash
sudo systemctl start ssh
```

---

## Stop

```bash
sudo systemctl stop ssh
```

---

## Restart

```bash
sudo systemctl restart ssh
```

---

## Enable at Boot

```bash
sudo systemctl enable ssh
```

---

## Check Active State

```bash
systemctl is-active ssh
```

---

## Check Enabled State

```bash
systemctl is-enabled ssh
```

---

# 26. Wazuh Service Examples

```bash
sudo systemctl status wazuh-manager
```

```bash
sudo systemctl status wazuh-indexer
```

```bash
sudo systemctl status wazuh-dashboard
```

Restart manager:

```bash
sudo systemctl restart wazuh-manager
```

---

# 27. `journalctl`

`journalctl` reads systemd logs.

## All Logs

```bash
journalctl
```

---

## Logs for Service

```bash
sudo journalctl -u wazuh-manager
```

---

## Last 100 Entries

```bash
sudo journalctl -u wazuh-manager -n 100
```

---

## Follow Live

```bash
sudo journalctl -u wazuh-manager -f
```

---

## Since Specific Time

```bash
journalctl --since "2026-09-14 10:00:00"
```

---

## Time Range

```bash
journalctl \
  --since "2026-09-14 10:00:00" \
  --until "2026-09-14 11:00:00"
```

Useful when building a timeline.

---

# 28. Authentication Logs

Depending on distribution:

Ubuntu / Debian commonly use:

```text
/var/log/auth.log
```

Example:

```bash
sudo grep -i "failed" /var/log/auth.log
```

Successful SSH logins:

```bash
sudo grep "Accepted" /var/log/auth.log
```

Failed SSH:

```bash
sudo grep "Failed password" /var/log/auth.log
```

---

# 29. Search Failed SSH Attempts

```bash
sudo grep "Failed password" /var/log/auth.log
```

Count source IPs:

```bash
sudo grep "Failed password" /var/log/auth.log \
| awk '{print $(NF-3)}' \
| sort \
| uniq -c \
| sort -nr
```

This is a useful example of combining Linux tools for threat hunting.

---

# 30. `awk`

`awk` is useful for selecting fields.

Example:

```bash
awk '{print $1}' file.txt
```

Print first field.

Another example:

```bash
ps aux | awk '{print $1, $2, $11}'
```

Shows:

```text
USER PID COMMAND
```

---

# 31. `cut`

Extract a field separated by a delimiter.

Example:

```bash
cut -d ':' -f 1 /etc/passwd
```

This prints usernames.

---

# 32. `sort`

```bash
sort file.txt
```

Reverse:

```bash
sort -r file.txt
```

Numeric:

```bash
sort -n file.txt
```

Reverse numeric:

```bash
sort -nr file.txt
```

---

# 33. `uniq`

Remove adjacent duplicates:

```bash
uniq file.txt
```

Count:

```bash
uniq -c file.txt
```

Usually paired with `sort`:

```bash
sort file.txt | uniq -c
```

---

# 34. `wc`

Count lines:

```bash
wc -l file.txt
```

Words:

```bash
wc -w file.txt
```

Characters:

```bash
wc -c file.txt
```

---

# 35. User Investigation

## Current User

```bash
whoami
```

---

## Logged-In Users

```bash
who
```

More detail:

```bash
w
```

---

## User Details

```bash
id
```

Specific user:

```bash
id shaan
```

---

# 36. `/etc/passwd`

View users:

```bash
cat /etc/passwd
```

Usernames only:

```bash
cut -d ':' -f 1 /etc/passwd
```

---

# 37. Privileged Groups

Check sudo group:

```bash
getent group sudo
```

Depending on distro:

```bash
getent group wheel
```

---

# 38. Login History

```bash
last
```

Failed login history may be available with:

```bash
lastb
```

Usually requires root:

```bash
sudo lastb
```

---

# 39. System Information

## Hostname

```bash
hostname
```

---

## Kernel

```bash
uname -a
```

Architecture only:

```bash
uname -m
```

For the current lab, expected ARM architecture may appear as:

```text
aarch64
```

---

## OS Information

```bash
cat /etc/os-release
```

---

# 40. Date and Time

```bash
date
```

UTC:

```bash
date -u
```

Time configuration:

```bash
timedatectl
```

Timezones matter during incident investigations.

---

# 41. Disk Usage

## Filesystems

```bash
df -h
```

Useful for troubleshooting SIEM storage issues.

---

## Directory Size

```bash
du -sh /var/log
```

Individual contents:

```bash
du -sh /var/log/*
```

Sort by size:

```bash
du -sh /var/log/* | sort -h
```

---

# 42. Memory

```bash
free -h
```

Useful fields:

```text
total
used
free
available
```

---

# 43. CPU

```bash
lscpu
```

Live usage:

```bash
top
```

---

# 44. Mounted Filesystems

```bash
mount
```

Cleaner:

```bash
findmnt
```

---

# 45. Package Investigation

Debian / Ubuntu / Kali:

```bash
dpkg -l
```

Search:

```bash
dpkg -l | grep wazuh
```

---

## Package Details

```bash
apt show <package>
```

---

# 46. Package Updates

```bash
sudo apt update
```

Upgrade:

```bash
sudo apt full-upgrade -y
```

Cleanup:

```bash
sudo apt autoremove -y
```

---

# 47. Command History

```bash
history
```

Search:

```bash
history | grep ssh
```

Bash history file:

```text
~/.bash_history
```

View:

```bash
cat ~/.bash_history
```

Important:

> Command history is useful evidence but is not guaranteed to be complete or trustworthy.

---

# 48. Environment Variables

```bash
env
```

Specific:

```bash
echo $PATH
```

Useful when investigating executable lookup behaviour.

---

# 49. Scheduled Tasks

## Cron

System cron:

```bash
cat /etc/crontab
```

User cron:

```bash
crontab -l
```

Other cron locations:

```bash
ls -la /etc/cron.*
```

Cron can be used for legitimate scheduling or persistence.

---

# 50. Startup / Persistence Locations

Useful areas to inspect include:

```text
/etc/systemd/system/
/usr/lib/systemd/system/
/etc/init.d/
/etc/rc.local
/etc/cron*
~/.config/autostart/
```

Do not assume every entry is malicious.

---

# 51. Systemd Unit Files

List services:

```bash
systemctl list-units --type=service
```

All installed service unit files:

```bash
systemctl list-unit-files --type=service
```

Inspect:

```bash
systemctl cat <service>
```

Example:

```bash
systemctl cat ssh
```

---

# 52. Open Files

If installed:

```bash
lsof
```

Network-related:

```bash
sudo lsof -i
```

Specific process:

```bash
sudo lsof -p <PID>
```

Specific port:

```bash
sudo lsof -i :22
```

---

# 53. Strings

Extract printable strings:

```bash
strings suspicious_file
```

Search:

```bash
strings suspicious_file | grep -i "http"
```

Useful for preliminary binary triage.

This does not replace proper malware analysis.

---

# 54. Hex Inspection

If installed:

```bash
xxd suspicious_file | head
```

or:

```bash
hexdump -C suspicious_file | head
```

Useful for inspecting file headers.

---

# 55. Archives

List ZIP contents:

```bash
unzip -l archive.zip
```

Extract:

```bash
unzip archive.zip
```

List tar archive:

```bash
tar -tf archive.tar
```

Be careful when extracting unknown or malicious samples.

---

# 56. Process and Network Correlation

A useful investigation pattern:

```bash
ss -tnp
```

Find suspicious PID.

Then:

```bash
ps -fp <PID>
```

Then:

```bash
readlink -f /proc/<PID>/exe
```

Then:

```bash
tr '\0' ' ' < /proc/<PID>/cmdline
```

This answers:

```text
Which process made the connection?

What binary is it?

What command line started it?
```

---

# 57. Example Investigation — Suspicious Connection

Suppose:

```bash
ss -tnp
```

shows:

```text
192.168.1.10:45922 → 203.0.113.50:4444
```

with:

```text
pid=2314
```

Investigate:

```bash
ps -fp 2314
```

Then:

```bash
readlink -f /proc/2314/exe
```

Then:

```bash
tr '\0' ' ' < /proc/2314/cmdline
```

Then:

```bash
sudo lsof -p 2314
```

This is a logical investigation pivot.

---

# 58. Example Investigation — Failed SSH

Search:

```bash
sudo grep "Failed password" /var/log/auth.log
```

Count source IPs:

```bash
sudo grep "Failed password" /var/log/auth.log \
| awk '{print $(NF-3)}' \
| sort \
| uniq -c \
| sort -nr
```

Then search successful logins:

```bash
sudo grep "Accepted" /var/log/auth.log
```

Question:

```text
Did repeated failures eventually result in a successful login?
```

---

# 59. Example Investigation — Recently Modified Files

```bash
find /tmp -type f -mmin -60
```

For an interesting file:

```bash
stat <FILE>
```

Then:

```bash
file <FILE>
```

Then:

```bash
sha256sum <FILE>
```

Then:

```bash
strings <FILE> | head
```

---

# 60. Useful One-Liners

Top source IPs in a log:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

Count errors:

```bash
grep -ic "error" logfile.log
```

Find all `.sh` files in `/tmp`:

```bash
find /tmp -type f -name "*.sh"
```

Find world-writable files:

```bash
find / -type f -perm -0002 2>/dev/null
```

Find SUID files:

```bash
find / -perm -4000 -type f 2>/dev/null
```

These may be useful for security auditing, but not every result is malicious.

---

# 61. `sudo`

Run command as elevated user:

```bash
sudo command
```

Example:

```bash
sudo systemctl status wazuh-manager
```

Use elevated permissions only when required.

---

# 62. Standard Streams

Linux commands use:

```text
stdin  = 0
stdout = 1
stderr = 2
```

Suppress errors:

```bash
command 2>/dev/null
```

Example:

```bash
find / -name "example.txt" 2>/dev/null
```

This hides permission-denied messages.

---

# 63. `&&` and `;`

Run second command only if first succeeds:

```bash
command1 && command2
```

Run regardless:

```bash
command1 ; command2
```

Example:

```bash
sudo apt update && sudo apt full-upgrade -y
```

---

# 64. Useful Keyboard Shortcuts

```text
Ctrl+C     Stop current command
Ctrl+L     Clear terminal
Ctrl+A     Beginning of command
Ctrl+E     End of command
Ctrl+R     Search command history
Tab        Autocomplete
Up Arrow   Previous command
```

---

# 65. Investigation Workflow

When investigating a Linux host:

```text
1. Identify Host
2. Check Time
3. Check Logged-In Users
4. Check Processes
5. Check Network Connections
6. Check Listening Ports
7. Check Services
8. Review Authentication Logs
9. Review System Logs
10. Check Recent Files
11. Check Persistence
12. Hash Suspicious Files
13. Build Timeline
14. Document Findings
```

---

# 66. Quick Linux Triage Commands

```bash
hostname
date
who
w
id
ip addr
ip route
ss -tulnp
ps aux
pstree -p
systemctl --failed
df -h
free -h
last
```

These provide a fast overview of system state.

---

# 67. Fast SOC Triage Block

```bash
echo "===== HOST ====="
hostname

echo "===== TIME ====="
date

echo "===== USERS ====="
w

echo "===== IP ====="
ip addr

echo "===== ROUTES ====="
ip route

echo "===== CONNECTIONS ====="
ss -tnp

echo "===== LISTENERS ====="
ss -ltnp

echo "===== PROCESSES ====="
ps aux

echo "===== FAILED SERVICES ====="
systemctl --failed
```

Use this only when appropriate for your own lab or authorised environment.

---

# 68. Evidence Preservation

Before changing suspicious files:

```bash
sha256sum suspicious_file
```

Record metadata:

```bash
stat suspicious_file
```

Consider copying the file to an evidence directory rather than modifying the original.

Example:

```bash
mkdir -p ~/evidence
cp --preserve=all suspicious_file ~/evidence/
```

For serious forensic work, stricter evidence-handling procedures are required.

---

# 69. Important Analyst Principle

Commands themselves are not the investigation.

For example:

```bash
ss -tnp
```

does not answer:

```text
Is the connection malicious?
```

It provides evidence.

You still need context:

```text
Process
Destination
Port
User
Timeline
Purpose
Threat Intelligence
```

---

# 70. Commands to Know First

If starting from scratch, prioritise:

```text
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
pstree
top
ip addr
ip route
ss
curl
systemctl
journalctl
df
free
who
w
last
history
```

---

# 71. Commands to Learn Next

After the basics:

```text
awk
cut
sort
uniq
wc
lsof
strings
xxd
findmnt
dig
nc
pgrep
readlink
```

---

# 72. Blue Team Mental Model

Use Linux commands to answer investigation questions.

Example:

```text
Question:
What process is listening on port 8080?
```

Command:

```bash
ss -ltnp | grep 8080
```

Next question:

```text
What binary belongs to that PID?
```

Command:

```bash
readlink -f /proc/<PID>/exe
```

Next:

```text
What command line launched it?
```

Command:

```bash
tr '\0' ' ' < /proc/<PID>/cmdline
```

This is how commands become an investigation workflow.

---

# 73. Key Takeaway

Do not try to memorise Linux as a giant list of commands.

Think in questions:

```text
Where am I?
pwd

What files exist?
ls

What is inside this file?
cat / less

Where does this text appear?
grep

Where is this file?
find

What is this file?
file

When did it change?
stat

What is its hash?
sha256sum

What is running?
ps

What is connected?
ss

Who is logged in?
w

What happened in the logs?
journalctl / grep
```

That is the most useful Linux mindset for Blue Team and SOC work.
