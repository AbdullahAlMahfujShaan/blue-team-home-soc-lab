# Networking Cheat Sheet for Blue Team / SOC Analysis

This cheat sheet focuses on networking concepts that are most useful for:

- Blue Team investigations
- SOC analysis
- BTL1 preparation
- Wireshark analysis
- Wazuh investigations
- Windows and Linux troubleshooting
- CCNA fundamentals
- Incident response

The goal is not to memorise random port numbers.

The goal is to understand:

```text
Who is communicating?
        ↓
With whom?
        ↓
Using which protocol?
        ↓
On which port?
        ↓
Is the traffic expected?
```

---

# 1. Networking Investigation Mental Model

When investigating network activity, answer:

```text
Source IP
Source Port
Destination IP
Destination Port
Protocol
Direction
Process
Domain
Timestamp
```

Example:

```text
Source:
192.168.1.20:51842

Destination:
8.8.8.8:53

Protocol:
UDP
```

This likely represents:

```text
DNS traffic
```

But always confirm context.

---

# 2. IP Address

An IP address identifies a network interface.

Example IPv4:

```text
192.168.1.10
```

Example IPv6:

```text
2001:db8::10
```

Think:

```text
IP Address = Logical Network Address
```

---

# 3. IPv4

IPv4 uses:

```text
32 bits
```

Example:

```text
192.168.1.10
```

Four octets:

```text
192 . 168 . 1 . 10
```

Each octet:

```text
0–255
```

---

# 4. IPv6

IPv6 uses:

```text
128 bits
```

Example:

```text
2001:db8:abcd:1::10
```

IPv6 was designed partly to solve IPv4 address exhaustion.

Advantages include:

- Much larger address space
- Simplified header design
- Better support for modern networking
- No requirement for NAT purely because of address exhaustion

---

# 5. IPv4 vs IPv6

| Feature | IPv4 | IPv6 |
|---|---|---|
| Size | 32-bit | 128-bit |
| Example | `192.168.1.10` | `2001:db8::10` |
| Address Space | ~4.3 billion | Extremely large |
| Broadcast | Yes | No traditional broadcast |
| NAT | Common | Usually less necessary |
| Header | More complex | Simplified |

---

# 6. Private IPv4 Address Ranges

RFC1918 private ranges:

```text
10.0.0.0/8
```

```text
172.16.0.0/12
```

```text
192.168.0.0/16
```

Examples:

```text
10.10.10.15
172.16.106.134
192.168.1.50
```

These are not directly routable on the public internet.

---

# 7. Public IP Addresses

Anything outside reserved/private ranges may potentially be publicly routable.

Example:

```text
8.8.8.8
```

When investigating:

```text
Private → Public
```

you are usually looking at outbound internet communication.

---

# 8. Loopback

IPv4 loopback:

```text
127.0.0.1
```

Hostname:

```text
localhost
```

IPv6 loopback:

```text
::1
```

Meaning:

```text
This computer itself
```

---

# 9. APIPA

If Windows cannot obtain an IPv4 address through DHCP, it may self-assign:

```text
169.254.0.0/16
```

Example:

```text
169.254.10.50
```

This often indicates:

```text
DHCP problem
```

---

# 10. MAC Address

A MAC address identifies a network interface at the Data Link layer.

Example:

```text
00:11:22:33:44:55
```

Think:

```text
IP  = logical address
MAC = local-link hardware address
```

---

# 11. OSI Model

The OSI model has seven layers.

```text
7 Application
6 Presentation
5 Session
4 Transport
3 Network
2 Data Link
1 Physical
```

Mnemonic:

```text
All
People
Seem
To
Need
Data
Processing
```

---

# 12. OSI Layer 7 — Application

Protocols include:

```text
HTTP
HTTPS
DNS
DHCP
FTP
SSH
SMTP
IMAP
POP3
SMB
RDP
SNMP
```

This layer provides network services to applications.

---

# 13. OSI Layer 6 — Presentation

Functions include:

```text
Encryption
Decryption
Encoding
Compression
Data representation
```

Examples:

```text
TLS concepts
JPEG
UTF-8
```

---

# 14. OSI Layer 5 — Session

Responsible for:

```text
Starting sessions
Maintaining sessions
Ending sessions
```

In practical TCP/IP troubleshooting, Layers 5–7 are often discussed together as the application layer.

---

# 15. OSI Layer 4 — Transport

Main protocols:

```text
TCP
UDP
```

Responsible for:

```text
Ports
Segmentation
Reliability
Flow control
```

---

# 16. OSI Layer 3 — Network

Main protocol:

```text
IP
```

Devices:

```text
Routers
Layer 3 switches
```

Functions:

```text
Logical addressing
Routing
Path selection
```

---

# 17. OSI Layer 2 — Data Link

Examples:

```text
Ethernet
802.11 Wi-Fi
ARP-related local resolution concepts
VLANs
MAC addressing
```

Devices:

```text
Switches
Bridges
NICs
```

---

# 18. OSI Layer 1 — Physical

Examples:

```text
Copper cable
Fiber
Radio signals
Connectors
Electrical signalling
```

Think:

```text
Bits travelling physically
```

---

# 19. TCP/IP Model

Common four-layer model:

```text
Application
Transport
Internet
Network Access
```

Mapping:

| TCP/IP | OSI |
|---|---|
| Application | 5–7 |
| Transport | 4 |
| Internet | 3 |
| Network Access | 1–2 |

---

# 20. Encapsulation

When data travels down the stack:

```text
Application Data
      ↓
TCP/UDP Header
      ↓
IP Header
      ↓
Ethernet Header + Trailer
      ↓
Bits
```

Names:

```text
Application → Data
Transport   → Segment / Datagram
Network     → Packet
Data Link   → Frame
Physical    → Bits
```

---

# 21. Decapsulation

At the destination:

```text
Bits
 ↓
Frame
 ↓
Packet
 ↓
Segment
 ↓
Application Data
```

Each layer removes its relevant header.

---

# 22. TCP

TCP stands for:

```text
Transmission Control Protocol
```

Characteristics:

- Connection-oriented
- Reliable
- Ordered delivery
- Sequence numbers
- Acknowledgements
- Retransmission
- Flow control

Use when reliability matters.

Examples:

```text
HTTPS
SSH
SMTP
SMB
RDP
```

---

# 23. UDP

UDP stands for:

```text
User Datagram Protocol
```

Characteristics:

- Connectionless
- Lower overhead
- No built-in delivery guarantee
- No retransmission at the UDP layer
- Faster for certain applications

Examples:

```text
DNS
DHCP
NTP
VoIP
Streaming
```

---

# 24. TCP vs UDP

| Feature | TCP | UDP |
|---|---|---|
| Connection | Yes | No |
| Reliable | Yes | No built-in guarantee |
| Ordered | Yes | No |
| Acknowledgements | Yes | No |
| Overhead | Higher | Lower |
| Typical Use | Web, SSH, SMB | DNS, DHCP, voice |

---

# 25. TCP Three-Way Handshake

TCP connection establishment:

```text
Client                    Server

SYN  -------------------->

     <---------------- SYN-ACK

ACK  -------------------->
```

Think:

```text
SYN
SYN-ACK
ACK
```

---

# 26. TCP Flags

Important flags:

```text
SYN
ACK
FIN
RST
PSH
URG
```

---

# 27. SYN

```text
SYN
```

Used to begin a TCP connection.

A large number of SYN packets without completed handshakes may warrant investigation.

---

# 28. ACK

```text
ACK
```

Acknowledges received TCP data.

---

# 29. FIN

```text
FIN
```

Used to gracefully close a TCP connection.

---

# 30. RST

```text
RST
```

Immediately resets a TCP connection.

May indicate:

- Connection refused
- Abrupt termination
- Application error
- Firewall or network behaviour

---

# 31. TCP Connection Termination

Typical graceful closure may involve:

```text
FIN
ACK
FIN
ACK
```

---

# 32. Ports

Ports identify services or application endpoints.

Range:

```text
0–65535
```

Think:

```text
IP = Building Address
Port = Door
```

---

# 33. Port Categories

```text
0–1023
Well-known ports

1024–49151
Registered ports

49152–65535
Dynamic / ephemeral ports
```

---

# 34. Source vs Destination Port

Example:

```text
192.168.1.20:51842
        ↓
93.184.216.34:443
```

Usually:

```text
51842 = temporary client source port
443   = server destination port
```

---

# 35. Important Ports

| Port | Protocol | Service |
|---|---|---|
| `20/21` | TCP | FTP |
| `22` | TCP | SSH |
| `23` | TCP | Telnet |
| `25` | TCP | SMTP |
| `53` | TCP/UDP | DNS |
| `67/68` | UDP | DHCP |
| `69` | UDP | TFTP |
| `80` | TCP | HTTP |
| `88` | TCP/UDP | Kerberos |
| `110` | TCP | POP3 |
| `123` | UDP | NTP |
| `135` | TCP | RPC Endpoint Mapper |
| `137-139` | TCP/UDP | NetBIOS |
| `143` | TCP | IMAP |
| `161/162` | UDP | SNMP |
| `389` | TCP/UDP | LDAP |
| `443` | TCP | HTTPS |
| `445` | TCP | SMB |
| `464` | TCP/UDP | Kerberos password |
| `465` | TCP | SMTPS |
| `514` | UDP | Syslog |
| `587` | TCP | SMTP Submission |
| `636` | TCP | LDAPS |
| `993` | TCP | IMAPS |
| `995` | TCP | POP3S |
| `1433` | TCP | Microsoft SQL Server |
| `1521` | TCP | Oracle |
| `2049` | TCP/UDP | NFS |
| `3306` | TCP | MySQL |
| `3389` | TCP/UDP | RDP |
| `5432` | TCP | PostgreSQL |
| `5985` | TCP | WinRM HTTP |
| `5986` | TCP | WinRM HTTPS |
| `8080` | TCP | Common alternate HTTP |

---

# 36. Wazuh Lab Ports

Important current lab ports:

```text
1514/TCP
Wazuh agent communication
```

```text
1515/TCP
Wazuh agent enrollment
```

```text
55000/TCP
Wazuh API
```

Example test:

```powershell
Test-NetConnection <WAZUH-IP> -Port 1514
```

---

# 37. Port Does Not Guarantee Protocol

Important analyst rule:

```text
Port 443 ≠ Definitely HTTPS
```

Applications can use unusual ports.

Malware can communicate over:

```text
443
53
80
```

to blend with normal traffic.

Always inspect:

```text
Protocol behaviour
Process
TLS information
Payload if available
Destination
```

---

# 38. DNS

DNS stands for:

```text
Domain Name System
```

Purpose:

```text
Domain name
     ↓
IP address
```

Example:

```text
example.com
      ↓
93.184.216.34
```

---

# 39. Common DNS Record Types

| Record | Purpose |
|---|---|
| `A` | IPv4 address |
| `AAAA` | IPv6 address |
| `CNAME` | Alias |
| `MX` | Mail server |
| `NS` | Name server |
| `TXT` | Text record |
| `PTR` | Reverse lookup |
| `SOA` | Zone authority |

---

# 40. A Record

Maps:

```text
Domain
  ↓
IPv4
```

Example:

```text
example.com → 93.184.216.34
```

---

# 41. AAAA Record

Maps:

```text
Domain
  ↓
IPv6
```

---

# 42. CNAME

Aliases one hostname to another.

Example:

```text
www.example.com
      ↓
example.hostingprovider.com
```

---

# 43. MX Record

Specifies mail servers for a domain.

Useful during:

```text
Phishing investigations
Email analysis
Domain reconnaissance
```

---

# 44. TXT Records

Used for many things including:

```text
SPF
DKIM-related information
Domain verification
Other metadata
```

Useful during phishing investigations.

---

# 45. DNS Investigation Questions

Ask:

```text
Which process queried the domain?

When?

What IP did it resolve to?

Did the same process connect afterward?

Is the domain expected?

Was it newly observed?

Does it resemble DNS tunnelling?
```

---

# 46. DNS Commands — Windows

```powershell
Resolve-DnsName example.com
```

DNS cache:

```powershell
Get-DnsClientCache
```

---

# 47. DNS Commands — Linux

```bash
dig example.com
```

Short output:

```bash
dig +short example.com
```

Alternative:

```bash
nslookup example.com
```

---

# 48. DHCP

DHCP stands for:

```text
Dynamic Host Configuration Protocol
```

It can provide:

```text
IP address
Subnet mask
Default gateway
DNS server
Lease information
```

---

# 49. DHCP Process — DORA

Remember:

```text
Discover
Offer
Request
Acknowledge
```

or:

```text
D
O
R
A
```

---

# 50. DHCP Ports

```text
UDP 67
Server
```

```text
UDP 68
Client
```

---

# 51. Default Gateway

A default gateway is the router used when sending traffic outside the local subnet.

Example:

```text
Host:
192.168.1.20

Gateway:
192.168.1.1
```

---

# 52. Routing

Routing determines where packets go between networks.

Basic concept:

```text
Destination IP
      ↓
Routing Table
      ↓
Next Hop / Interface
```

---

# 53. Windows Route Command

```cmd
route print
```

PowerShell:

```powershell
Get-NetRoute
```

---

# 54. Linux Route Command

```bash
ip route
```

Example:

```text
default via 172.16.106.2 dev eth0
```

---

# 55. Subnet

A subnet divides an IP network into smaller networks.

Example:

```text
192.168.1.0/24
```

means:

```text
Network:
192.168.1.0

Common usable range:
192.168.1.1–192.168.1.254

Broadcast:
192.168.1.255
```

---

# 56. CIDR

CIDR example:

```text
192.168.1.0/24
```

The:

```text
/24
```

means:

```text
24 bits are network bits.
```

Equivalent subnet mask:

```text
255.255.255.0
```

---

# 57. Common CIDR Values

| CIDR | Subnet Mask | Total Addresses |
|---|---|---:|
| `/8` | `255.0.0.0` | 16,777,216 |
| `/16` | `255.255.0.0` | 65,536 |
| `/24` | `255.255.255.0` | 256 |
| `/25` | `255.255.255.128` | 128 |
| `/26` | `255.255.255.192` | 64 |
| `/27` | `255.255.255.224` | 32 |
| `/28` | `255.255.255.240` | 16 |
| `/29` | `255.255.255.248` | 8 |
| `/30` | `255.255.255.252` | 4 |

Traditional IPv4 subnet exercises often subtract:

```text
Network address
Broadcast address
```

from total addresses for usable host counts.

---

# 58. `/24`

```text
192.168.1.0/24
```

Total addresses:

```text
256
```

Traditional usable hosts:

```text
254
```

---

# 59. `/30`

Often used historically for point-to-point links.

```text
4 addresses
```

Traditional:

```text
2 usable hosts
```

---

# 60. Same Subnet Communication

Suppose:

```text
Host A:
192.168.1.10/24

Host B:
192.168.1.20/24
```

Both belong to:

```text
192.168.1.0/24
```

Therefore:

```text
Direct local communication
```

can occur without sending through a router.

---

# 61. Different Subnets

Example:

```text
Host A:
192.168.1.10/24

Host B:
192.168.2.10/24
```

Different networks.

Traffic generally requires:

```text
Router / Layer 3 device
```

---

# 62. ARP

ARP stands for:

```text
Address Resolution Protocol
```

It helps map:

```text
IPv4 Address
      ↓
MAC Address
```

on a local network.

---

# 63. ARP Example

Host wants:

```text
192.168.1.20
```

It asks:

```text
Who has 192.168.1.20?
```

Device responds with its MAC.

Then Ethernet frames can be sent locally.

---

# 64. ARP Commands — Windows

```cmd
arp -a
```

PowerShell:

```powershell
Get-NetNeighbor
```

---

# 65. ARP Commands — Linux

```bash
ip neigh
```

---

# 66. ARP Security Relevance

Potential issues include:

```text
ARP spoofing
ARP poisoning
Man-in-the-middle attacks
```

Indicators may include:

```text
Unexpected MAC changes
Multiple IPs mapping strangely
Gateway MAC suddenly changing
```

---

# 67. ICMP

ICMP stands for:

```text
Internet Control Message Protocol
```

Used for:

```text
Errors
Diagnostics
Network control information
```

---

# 68. Ping

`ping` commonly uses ICMP Echo.

Windows:

```powershell
ping 8.8.8.8
```

Linux:

```bash
ping -c 4 8.8.8.8
```

---

# 69. Ping Failure Does Not Mean Host Is Down

ICMP can be blocked.

Therefore:

```text
Ping fails
```

does not necessarily mean:

```text
Host unavailable
```

Try relevant application ports as well.

---

# 70. Traceroute

Shows approximate network path.

Windows:

```cmd
tracert 8.8.8.8
```

Linux:

```bash
traceroute 8.8.8.8
```

Useful for:

```text
Routing troubleshooting
Path investigation
Latency troubleshooting
```

---

# 71. NAT

NAT stands for:

```text
Network Address Translation
```

It translates addresses between networks.

Common home setup:

```text
Private IP
     ↓
Router NAT
     ↓
Public IP
```

---

# 72. NAT Example

Internal:

```text
192.168.1.20
```

Router public IP:

```text
203.0.113.10
```

Internet sees:

```text
203.0.113.10
```

rather than the internal private IP.

---

# 73. PAT

PAT means:

```text
Port Address Translation
```

Multiple internal devices can share one public IP by using different port mappings.

Example:

```text
192.168.1.10:51000
        ↓
203.0.113.10:40001
```

and:

```text
192.168.1.11:52000
        ↓
203.0.113.10:40002
```

---

# 74. VMware NAT in the Lab

Current lab uses VMware NAT.

Conceptually:

```text
BTL-WIN11
BTL-KALI
BTL-WAZUH
      ↓
VMware Virtual NAT
      ↓
Mac Host Network
      ↓
Internet
```

Benefits:

- Simple internet access
- Lab separation from the physical LAN
- Easy VM-to-VM networking within the virtual NAT segment

---

# 75. Firewall

A firewall controls network traffic based on rules.

Rules may consider:

```text
Source IP
Destination IP
Source Port
Destination Port
Protocol
Direction
Application
State
```

---

# 76. Stateful Firewall

A stateful firewall tracks connections.

Example:

```text
Internal host initiates HTTPS
       ↓
Firewall records connection
       ↓
Return traffic allowed
```

---

# 77. Inbound vs Outbound

Inbound:

```text
Traffic entering the host/network
```

Outbound:

```text
Traffic leaving the host/network
```

Example:

```text
PC → website
```

is outbound from the PC.

---

# 78. Windows Firewall Commands

Profiles:

```powershell
Get-NetFirewallProfile
```

Rules:

```powershell
Get-NetFirewallRule
```

---

# 79. Connection States

Common TCP states include:

```text
LISTEN
ESTABLISHED
SYN_SENT
SYN_RECEIVED
FIN_WAIT
TIME_WAIT
CLOSE_WAIT
```

---

# 80. LISTEN

Means:

```text
A service is waiting for inbound connections.
```

Example:

```text
0.0.0.0:22 LISTEN
```

may indicate SSH listening on all IPv4 interfaces.

---

# 81. ESTABLISHED

Means:

```text
TCP connection is currently established.
```

Useful during incident response:

```text
Which process currently communicates externally?
```

---

# 82. TIME_WAIT

Normal TCP state after connection closure.

Large quantities are not automatically suspicious.

---

# 83. Windows Connections

```powershell
Get-NetTCPConnection
```

Listening:

```powershell
Get-NetTCPConnection -State Listen
```

Established:

```powershell
Get-NetTCPConnection -State Established
```

---

# 84. Windows `netstat`

```cmd
netstat -ano
```

Important:

```text
-a = all
-n = numeric
-o = PID
```

Then map PID:

```powershell
Get-Process -Id <PID>
```

---

# 85. Linux Connections

```bash
ss -tulnp
```

Meaning:

```text
t = TCP
u = UDP
l = listening
n = numeric
p = process
```

---

# 86. Linux Established TCP

```bash
ss -tnp
```

---

# 87. Network Process Correlation

Example:

```text
Remote IP:
203.0.113.50

Remote Port:
443

PID:
4520
```

Windows:

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 4520"
```

Linux:

```bash
ps -fp 4520
```

The critical question:

```text
Which process owns this connection?
```

---

# 88. HTTP

HTTP:

```text
Hypertext Transfer Protocol
```

Default:

```text
TCP 80
```

HTTP is normally unencrypted.

---

# 89. HTTPS

HTTPS:

```text
HTTP over TLS
```

Default:

```text
TCP 443
```

Provides:

```text
Encryption
Integrity
Server authentication
```

depending on configuration and certificate validation.

---

# 90. TLS

TLS provides encrypted communication.

Useful investigation artefacts include:

```text
Server Name
Certificate
Issuer
Validity
Cipher
TLS version
```

Even when payload is encrypted, metadata can still be valuable.

---

# 91. SSH

SSH:

```text
Secure Shell
```

Default:

```text
TCP 22
```

Used for:

```text
Remote shell
Administration
File transfer
Tunnelling
```

---

# 92. Telnet

Telnet:

```text
TCP 23
```

Historically used for remote terminal access.

Major issue:

```text
No encryption
```

Avoid for secure modern administration.

---

# 93. FTP

FTP traditionally uses:

```text
TCP 21
Control
```

and:

```text
TCP 20
Data in traditional active mode
```

FTP itself is unencrypted.

---

# 94. SFTP

SFTP runs over:

```text
SSH
```

typically:

```text
TCP 22
```

It is not the same protocol as FTP.

---

# 95. SMTP

Email sending protocol.

Common ports:

```text
25
587
465
```

Typical use:

```text
25  = server-to-server SMTP
587 = mail submission
465 = implicit TLS SMTP
```

---

# 96. IMAP

Email retrieval / mailbox synchronisation.

```text
143
```

Encrypted:

```text
993
```

---

# 97. POP3

Email retrieval.

```text
110
```

Encrypted:

```text
995
```

---

# 98. SMB

Server Message Block:

```text
TCP 445
```

Used for:

```text
Windows file sharing
Printer sharing
Named pipes
Domain-related operations
```

Extremely important in Windows environments.

---

# 99. SMB Security Relevance

SMB may be involved in:

```text
Lateral movement
Remote administration
File access
Credential attacks
Ransomware propagation
```

But SMB is also heavily used legitimately.

Context matters.

---

# 100. RDP

Remote Desktop Protocol:

```text
TCP/UDP 3389
```

Used for remote Windows desktop sessions.

Investigate with:

```text
4624 Logon Type 10
4625 Logon Type 10
```

plus network data.

---

# 101. LDAP

LDAP:

```text
Lightweight Directory Access Protocol
```

Common port:

```text
389
```

Encrypted LDAP:

```text
636
```

Used extensively with directory services such as Active Directory.

---

# 102. Kerberos

Kerberos commonly uses:

```text
TCP/UDP 88
```

Windows domain authentication events include:

```text
4768
4769
4771
```

---

# 103. NTP

Network Time Protocol:

```text
UDP 123
```

Time synchronisation is extremely important for:

```text
Authentication
Logs
SIEM correlation
Forensics
```

---

# 104. SNMP

Simple Network Management Protocol:

```text
UDP 161
```

Traps:

```text
UDP 162
```

Used to monitor network equipment.

---

# 105. RPC

Windows RPC Endpoint Mapper:

```text
TCP 135
```

RPC may also use dynamically allocated high ports.

Important in Windows administration and lateral movement investigations.

---

# 106. WinRM

Windows Remote Management:

```text
5985
HTTP
```

```text
5986
HTTPS
```

Used for:

```text
Remote PowerShell
Administration
Automation
```

---

# 107. VLAN

VLAN means:

```text
Virtual Local Area Network
```

It logically separates Layer 2 networks.

Example:

```text
VLAN 10 — Users
VLAN 20 — Servers
VLAN 30 — Security
VLAN 40 — Guest
```

---

# 108. Why VLANs Matter

Benefits:

```text
Segmentation
Reduced broadcast domains
Security separation
Network organisation
```

---

# 109. Access Port

An access port normally carries one VLAN for an end device.

Example:

```text
Switch port
      ↓
User PC
      ↓
VLAN 10
```

---

# 110. Trunk Port

A trunk carries multiple VLANs.

Common standard:

```text
802.1Q
```

Example:

```text
Switch
  ║
  ║ VLAN 10
  ║ VLAN 20
  ║ VLAN 30
  ║
Switch
```

---

# 111. 802.1Q VLAN Tag

802.1Q adds VLAN information to Ethernet frames.

Typical VLAN ID range:

```text
1–4094
```

Some values are reserved.

---

# 112. Inter-VLAN Routing

Devices on different VLANs require Layer 3 routing.

Example:

```text
VLAN 10
   ↓
Router / L3 Switch
   ↓
VLAN 20
```

---

# 113. Switch

A switch primarily works at:

```text
Layer 2
```

and forwards frames using:

```text
MAC addresses
```

---

# 114. MAC Address Table

A switch learns:

```text
Source MAC
       ↓
Incoming Port
```

Then builds a table such as:

```text
MAC Address          Port
00:11:22:33:44:55    Gi1/0/1
AA:BB:CC:DD:EE:FF    Gi1/0/5
```

---

# 115. Router

A router operates primarily at:

```text
Layer 3
```

and forwards traffic using:

```text
IP addresses
Routing table
```

---

# 116. Top of Rack — TOR

Top-of-Rack means a switch located at or near the top of a data centre rack.

Concept:

```text
Servers
│
├── Server
├── Server
├── Server
├── Server
│
└── TOR Switch
       ↓
   Upstream Network
```

Server cables connect to the local TOR rather than running individually across the entire data centre.

---

# 117. TOR Advantages

Benefits include:

- Shorter server cable runs
- Easier cable management
- Simplified rack deployment
- High bandwidth within rack
- Easier scaling
- Reduced cabling complexity

---

# 118. TOR in a Data Centre

Example:

```text
Server NIC
   ↓
TOR Switch
   ↓
Aggregation / Spine Network
   ↓
Other Racks / Services
```

In modern leaf-spine networks, TOR switches commonly act as:

```text
Leaf switches
```

---

# 119. Leaf-Spine Architecture

Concept:

```text
          Spine 1      Spine 2
          / | \        / | \
         /  |  \      /  |  \
      Leaf Leaf Leaf Leaf ...
       |    |    |
     Servers / racks
```

Each leaf typically connects to each spine.

Benefits:

- Predictable latency
- Redundant paths
- Scalable east-west traffic
- High bandwidth

---

# 120. East-West Traffic

Traffic between systems within a data centre/network:

```text
Server A
   ↓
Server B
```

This is:

```text
East-West traffic
```

---

# 121. North-South Traffic

Traffic entering or leaving the data centre/network:

```text
Internet
   ↓
Data Centre
```

or reverse.

This is:

```text
North-South traffic
```

---

# 122. PoE

PoE means:

```text
Power over Ethernet
```

It delivers:

```text
Power
+
Data
```

over Ethernet cable.

Used for:

```text
IP phones
Wireless access points
Cameras
IoT devices
```

---

# 123. Port Security

Cisco switch Port Security can restrict which MAC addresses are permitted on a switch port.

Basic concept:

```text
Switch Port
     ↓
Allowed MAC Address(es)
     ↓
Unexpected MAC?
     ↓
Violation action
```

---

# 124. Port Security Purpose

It can help prevent:

```text
Unauthorised devices
MAC address changes
Too many devices on one access port
```

---

# 125. Port Security Violation Modes

Common Cisco modes:

```text
Protect
Restrict
Shutdown
```

---

# 126. Protect

Drops frames from unauthorised MAC addresses.

Typically:

```text
No port shutdown
```

---

# 127. Restrict

Drops unauthorised frames and may:

```text
Increment violation counter
Generate logging / notification
```

---

# 128. Shutdown

Typically places the port into:

```text
Err-disabled
```

after a violation.

This is often the default violation action.

---

# 129. Sticky MAC

Port Security can dynamically learn MAC addresses and place them into configuration as sticky addresses.

Concept:

```text
Device connects
      ↓
Switch learns MAC
      ↓
MAC becomes authorised
```

---

# 130. DMZ

DMZ means:

```text
Demilitarized Zone
```

It is a separate network segment for systems exposed to untrusted networks.

Example:

```text
Internet
   ↓
Firewall
   ↓
DMZ
   ↓
Firewall
   ↓
Internal Network
```

---

# 131. DMZ Purpose

If a public server is compromised, segmentation helps reduce direct access to the internal network.

Common DMZ systems:

```text
Web servers
Reverse proxies
Mail gateways
DNS servers
```

---

# 132. Proxy

A proxy communicates on behalf of another system.

Forward proxy:

```text
Client
  ↓
Proxy
  ↓
Internet
```

Often used for:

```text
Filtering
Logging
Access control
Privacy
```

---

# 133. Reverse Proxy

A reverse proxy sits in front of servers.

```text
Internet
   ↓
Reverse Proxy
   ↓
Web Servers
```

Used for:

- Load balancing
- TLS termination
- Web protection
- Hiding backend architecture

---

# 134. VPN

VPN means:

```text
Virtual Private Network
```

It creates an encrypted tunnel over another network.

Example:

```text
Remote Laptop
     ↓
Encrypted VPN
     ↓
Corporate Network
```

---

# 135. Split Tunnel vs Full Tunnel

Full tunnel:

```text
All client traffic
      ↓
VPN
```

Split tunnel:

```text
Corporate traffic → VPN
Internet traffic  → Local internet
```

---

# 136. Broadcast

Broadcast traffic targets all hosts in a broadcast domain.

IPv4 example:

```text
255.255.255.255
```

or subnet-specific broadcast such as:

```text
192.168.1.255
```

---

# 137. Unicast

One sender to one receiver:

```text
Host A
  ↓
Host B
```

Most normal communications are unicast.

---

# 138. Multicast

One sender to a selected group of receivers.

Used in:

```text
Streaming
Routing protocols
Service discovery
```

depending on protocol.

---

# 139. Network Troubleshooting Model

Use:

```text
1. Physical
2. IP Configuration
3. Local Connectivity
4. Gateway
5. DNS
6. Remote IP
7. Service Port
8. Application
```

---

# 140. Troubleshooting Example

Website fails.

Start:

```text
Is interface up?
```

Then:

```text
Do I have an IP?
```

Then:

```text
Can I reach gateway?
```

Then:

```text
Can I reach public IP?
```

Then:

```text
Does DNS resolve?
```

Then:

```text
Can I connect to port 443?
```

This isolates the failing layer.

---

# 141. Windows Network Commands

```powershell
ipconfig
```

Detailed:

```powershell
ipconfig /all
```

PowerShell:

```powershell
Get-NetIPConfiguration
```

---

# 142. Linux Network Commands

```bash
ip addr
```

Short:

```bash
ip a
```

Routes:

```bash
ip route
```

---

# 143. Test Gateway

Windows:

```powershell
ping <GATEWAY-IP>
```

Linux:

```bash
ping -c 4 <GATEWAY-IP>
```

---

# 144. Test Internet Without DNS

```text
Ping known IP
```

Example:

```powershell
ping 8.8.8.8
```

If this works but:

```powershell
ping example.com
```

fails, DNS may be the issue.

---

# 145. Test DNS

Windows:

```powershell
Resolve-DnsName example.com
```

Linux:

```bash
dig example.com
```

---

# 146. Test TCP Port — Windows

```powershell
Test-NetConnection example.com -Port 443
```

Useful result:

```text
TcpTestSucceeded : True
```

---

# 147. Test TCP Port — Linux

```bash
nc -vz example.com 443
```

---

# 148. Packet Capture

Packet captures often use:

```text
.pcap
.pcapng
```

Tools:

```text
Wireshark
tcpdump
tshark
```

---

# 149. Wireshark Display Filters

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

---

# 150. TCP Filter

```text
tcp
```

Port:

```text
tcp.port == 443
```

Destination port:

```text
tcp.dstport == 443
```

---

# 151. UDP Filter

```text
udp
```

Port:

```text
udp.port == 53
```

---

# 152. DNS Filter

```text
dns
```

DNS query:

```text
dns.flags.response == 0
```

Responses:

```text
dns.flags.response == 1
```

---

# 153. HTTP Filter

```text
http
```

Requests:

```text
http.request
```

Methods:

```text
http.request.method == "GET"
```

---

# 154. TCP SYN Filter

Initial SYN packets:

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Useful when looking for connection attempts.

---

# 155. TCP Reset Filter

```text
tcp.flags.reset == 1
```

Useful when analysing failed or abruptly terminated connections.

---

# 156. Follow TCP Stream

In Wireshark:

```text
Right-click packet
      ↓
Follow
      ↓
TCP Stream
```

Useful for reconstructing application conversations when traffic is not encrypted.

---

# 157. Network Investigation Workflow

When given a suspicious IP:

```text
Identify IP
   ↓
Internal or External?
   ↓
Which host communicated?
   ↓
Which process?
   ↓
Which port?
   ↓
Which protocol?
   ↓
DNS query before connection?
   ↓
How much traffic?
   ↓
Was it expected?
   ↓
Threat intel / context
```

---

# 158. DNS Investigation Workflow

```text
Suspicious Domain
      ↓
Who queried it?
      ↓
Which process?
      ↓
What IP resolved?
      ↓
Connection afterward?
      ↓
File download?
      ↓
Other hosts querying same domain?
```

---

# 159. Suspicious Connection Workflow

```text
Connection Alert
     ↓
Source Host
     ↓
Owning Process
     ↓
Destination IP
     ↓
Destination Port
     ↓
DNS
     ↓
Process ancestry
     ↓
Files
     ↓
Verdict
```

---

# 160. Internal vs External IP

First ask:

```text
Is this RFC1918 private space?
```

If yes:

```text
Likely internal
```

If public:

```text
Potential internet destination
```

Then investigate context.

---

# 161. Common Network Security Indicators

Examples worth investigating:

```text
Many connection attempts to many ports
Repeated DNS queries for unusual domains
Unusual outbound connections
Connections at unusual times
Unexpected SMB between workstations
Unexpected RDP
Very high DNS request volume
Rare external IP contacted by sensitive host
Connections from unexpected processes
```

---

# 162. Port Scan Pattern

Conceptually:

```text
One source
   ↓
One or many destinations
   ↓
Many destination ports
```

Example:

```text
10.0.0.5 → 10.0.0.10:22
10.0.0.5 → 10.0.0.10:23
10.0.0.5 → 10.0.0.10:80
10.0.0.5 → 10.0.0.10:443
10.0.0.5 → 10.0.0.10:445
```

Could indicate scanning, but monitoring tools may also behave similarly.

---

# 163. Beaconing Pattern

Potential C2 beaconing can resemble:

```text
Host
  ↓
Same destination
  ↓
Regular interval
```

Example:

```text
10:00
10:05
10:10
10:15
10:20
```

Regularity alone does not prove malware, because legitimate software also phones home periodically.

---

# 164. DNS Tunnelling Concept

DNS can be abused to transfer data through unusual query patterns.

Potential indicators:

```text
Very long subdomains
High entropy
Large volume
Many unique queries
Regular beaconing
Unusual record types
```

Again:

```text
Indicator ≠ Verdict
```

---

# 165. Exfiltration Clues

Potential network signs:

```text
Large outbound transfers
Unexpected cloud storage
Rare external destination
Long-running encrypted session
Unusual protocol
Traffic outside normal hours
Sensitive endpoint contacting unknown infrastructure
```

Requires context.

---

# 166. Lateral Movement Clues

Potential protocols:

```text
SMB 445
RDP 3389
WinRM 5985/5986
SSH 22
RPC 135 + dynamic ports
```

Look at:

```text
Source host
Destination host
User
Authentication
Process
Time
```

---

# 167. SMB Investigation Pattern

```text
Source workstation
      ↓
TCP 445
      ↓
Another workstation/server
```

Then check:

```text
User authentication
Shares accessed
Processes started
Service creation
Scheduled tasks
```

---

# 168. RDP Investigation Pattern

```text
Source IP
   ↓
3389
   ↓
Destination Windows Host
```

Correlate with:

```text
4624 / 4625
Logon Type 10
```

---

# 169. PowerShell Remote Management

WinRM-related traffic may use:

```text
5985
5986
```

Correlate with:

```text
PowerShell processes
Authentication events
Remote host
User
```

---

# 170. Common Protocol Map

```text
DNS      → 53
HTTP     → 80
HTTPS    → 443
SSH      → 22
SMB      → 445
RDP      → 3389
Kerberos → 88
LDAP     → 389
LDAPS    → 636
NTP      → 123
WinRM    → 5985/5986
```

---

# 171. Protocol Layer Map

```text
HTTP / HTTPS / DNS / SSH / SMB
                ↓
Application Layer

TCP / UDP
                ↓
Transport Layer

IP / ICMP
                ↓
Network Layer

Ethernet / MAC / VLAN
                ↓
Data Link Layer

Cable / Fiber / Radio
                ↓
Physical Layer
```

---

# 172. Troubleshooting by OSI Layer

## Layer 1

Ask:

```text
Cable connected?
Interface enabled?
Link light?
Wi-Fi connected?
```

## Layer 2

Ask:

```text
Correct VLAN?
MAC learned?
ARP working?
```

## Layer 3

Ask:

```text
Correct IP?
Subnet mask?
Gateway?
Route?
```

## Layer 4

Ask:

```text
TCP or UDP?
Correct port?
Firewall blocking?
```

## Layer 7

Ask:

```text
Application working?
DNS?
TLS?
Authentication?
```

---

# 173. Example Troubleshooting — Wazuh Agent

Problem:

```text
BTL-WIN11 cannot reach Wazuh manager
```

Check:

```text
1. Wazuh agent running?
```

```powershell
Get-Service Wazuh
```

Then:

```text
2. Correct Windows IP?
```

```powershell
ipconfig
```

Then:

```text
3. Correct Wazuh IP?
```

Then:

```text
4. Can host reach Wazuh?
```

```powershell
ping <WAZUH-IP>
```

Then:

```text
5. Is agent port reachable?
```

```powershell
Test-NetConnection <WAZUH-IP> -Port 1514
```

Then inspect:

```text
Agent config
Agent logs
Wazuh services
```

---

# 174. Example Troubleshooting — DNS

Problem:

```text
Website will not load
```

Test IP:

```powershell
ping 8.8.8.8
```

If successful:

```powershell
Resolve-DnsName example.com
```

If DNS fails:

```text
Connectivity works
but name resolution does not.
```

---

# 175. Example Troubleshooting — Service Port

Host responds to ping but application fails.

Test:

```powershell
Test-NetConnection <SERVER-IP> -Port 443
```

Possible result:

```text
Ping succeeds
TCP 443 fails
```

Potential issues:

```text
Service stopped
Firewall
Wrong port
Application not listening
Routing/security rule
```

---

# 176. Blue Team Network Correlation

Network telemetry becomes more valuable when combined with endpoint telemetry.

Example:

```text
Sysmon Event 1
powershell.exe
      ↓
Sysmon Event 22
example.com
      ↓
Sysmon Event 3
93.184.216.34:443
```

This answers:

```text
Which process
resolved which domain
and connected where?
```

---

# 177. Network + Authentication Correlation

Example:

```text
4624
Remote login
    ↓
New process
    ↓
Outbound connection
```

This is more meaningful than a connection alone.

---

# 178. Network + File Correlation

Example:

```text
DNS query
   ↓
HTTPS connection
   ↓
File created
   ↓
File executed
```

This can indicate:

```text
Download
      ↓
Execution
```

but requires evidence.

---

# 179. Important Network Fields in Wazuh/Sysmon

Useful fields include:

```text
sourceIp
sourcePort
destinationIp
destinationPort
destinationHostname
protocol
image
processId
processGuid
user
queryName
```

---

# 180. Important Wireshark Questions

When viewing a packet:

```text
What is the timestamp?

Source?

Destination?

Protocol?

Source port?

Destination port?

TCP flags?

DNS query?

Payload?

What packets came before and after?
```

---

# 181. Five-Tuple

A network connection is often identified by:

```text
Source IP
Source Port
Destination IP
Destination Port
Protocol
```

This is called the:

```text
5-tuple
```

Example:

```text
192.168.1.20
51842
93.184.216.34
443
TCP
```

---

# 182. Why the Five-Tuple Matters

It uniquely describes a communication flow better than just saying:

```text
Connection to port 443
```

because many simultaneous connections may use port 443.

---

# 183. Ephemeral Ports

Client systems usually choose temporary high-numbered source ports.

Example:

```text
Client:
192.168.1.10:53214

Server:
203.0.113.20:443
```

Here:

```text
53214 = ephemeral port
443   = service port
```

---

# 184. Well-Known Port Pitfall

Do not say:

```text
I saw port 53, therefore this is definitely DNS.
```

Better:

```text
Traffic used port 53 and appears consistent with DNS.
```

Validate protocol structure whenever possible.

---

# 185. Encryption Pitfall

Encrypted traffic limits payload visibility.

However, you may still have:

```text
Source IP
Destination IP
Ports
Timing
Volume
TLS certificate
DNS
SNI
Process telemetry
```

Metadata can still be extremely useful.

---

# 186. Network Baseline

To identify abnormal traffic, learn normal traffic.

Baseline examples:

```text
Which DNS servers are normal?

Which cloud services are used?

Which internal subnets exist?

Which systems normally use RDP?

Which servers normally communicate over SMB?

Which destinations are expected for OneDrive?
```

---

# 187. Normal vs Suspicious

Example:

```text
OneDrive.exe → Microsoft cloud → 443
```

may be expected.

But:

```text
notepad.exe → unknown public IP → 4444
```

would be unusual and deserves investigation.

The key is:

```text
Process + Destination + Protocol + Context
```

---

# 188. Questions for Any Network Alert

Ask:

```text
Is the source internal or external?

Is the destination internal or external?

Which side initiated?

What process owns the connection?

Which user owns the process?

Which port?

Which protocol?

Was DNS involved?

Is the destination known?

What happened before the connection?

What happened after?
```

---

# 189. Commands to Know First — Windows

```powershell
ipconfig /all
```

```powershell
Get-NetIPConfiguration
```

```powershell
Get-NetIPAddress
```

```powershell
Get-NetRoute
```

```powershell
Get-NetTCPConnection
```

```powershell
Resolve-DnsName example.com
```

```powershell
Test-NetConnection example.com -Port 443
```

```cmd
arp -a
```

```cmd
netstat -ano
```

---

# 190. Commands to Know First — Linux

```bash
ip addr
```

```bash
ip route
```

```bash
ip neigh
```

```bash
ss -tulnp
```

```bash
ping -c 4 <IP>
```

```bash
dig example.com
```

```bash
nc -vz <IP> <PORT>
```

---

# 191. Wireshark Filters to Know First

```text
ip.addr == <IP>
```

```text
tcp
```

```text
udp
```

```text
dns
```

```text
tcp.port == 443
```

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

```text
tcp.flags.reset == 1
```

```text
http.request
```

---

# 192. Ports to Know First

Memorise these first:

```text
22    SSH
25    SMTP
53    DNS
67/68 DHCP
80    HTTP
88    Kerberos
123   NTP
135   RPC
389   LDAP
443   HTTPS
445   SMB
636   LDAPS
3389  RDP
5985  WinRM HTTP
5986  WinRM HTTPS
```

Then add others gradually.

---

# 193. Protocols to Know First

Prioritise:

```text
ARP
IP
ICMP
TCP
UDP
DNS
DHCP
HTTP
HTTPS
SSH
SMB
RDP
LDAP
Kerberos
NTP
```

---

# 194. BTL1 / SOC Networking Focus

For Blue Team preparation, focus heavily on:

```text
TCP vs UDP
TCP handshake
IP addressing
Private vs public IP
Ports
DNS
HTTP/HTTPS
ARP
DHCP
SMB
RDP
Basic subnetting
Wireshark filters
Network timelines
Process-to-network correlation
```

---

# 195. CCNA Topics That Help Blue Team

CCNA networking knowledge improves investigations through:

```text
Subnetting
VLANs
Routing
Switching
ARP
MAC tables
STP
DHCP
NAT
ACLs
IPv4
IPv6
TCP/UDP
Network troubleshooting
```

Networking fundamentals directly improve SOC analysis.

---

# 196. Example Investigation

Alert:

```text
powershell.exe connected to 203.0.113.50:443
```

Do not conclude:

```text
Malware
```

Investigate:

```text
1. Which endpoint?

2. Which user?

3. What PowerShell command?

4. Parent process?

5. DNS query before connection?

6. Is destination known?

7. Did PowerShell download a file?

8. Was a new file created?

9. Did the file execute?

10. Is this expected administration?
```

---

# 197. Example DNS Investigation

Evidence:

```text
Process:
chrome.exe

Query:
accounts.google.com
```

Likely context:

```text
Normal browser activity
```

Evidence:

```text
Process:
unknown.exe

Query:
x82kk19a9.example.xyz
```

More suspicious.

Why?

```text
Unexpected process
+
Unusual hostname
+
Potentially unusual domain
```

Still requires investigation.

---

# 198. Example Port 445 Investigation

Evidence:

```text
Workstation A
   ↓
Workstation B:445
```

Ask:

```text
Was file sharing expected?

Which account authenticated?

Was a remote service created?

Was PsExec-like activity present?

Was the destination a normal file server?
```

Port 445 alone is not enough.

---

# 199. Example Port 3389 Investigation

Evidence:

```text
External IP
    ↓
Internal Host:3389
```

Investigate:

```text
4625 failures?
4624 Logon Type 10?
Account?
Source IP?
MFA/VPN?
Post-login processes?
```

---

# 200. Example Beaconing Investigation

Observed:

```text
10:00 → 203.0.113.50:443
10:05 → 203.0.113.50:443
10:10 → 203.0.113.50:443
10:15 → 203.0.113.50:443
```

Possible interpretations:

```text
Malware C2
Software updater
Telemetry agent
Cloud sync
Monitoring agent
```

Need:

```text
Process ownership
Destination reputation
Interval analysis
Payload/metadata
Endpoint context
```

---

# 201. Common Analyst Mistakes

Do not assume:

```text
Unknown IP = malicious
```

Do not assume:

```text
443 = safe
```

Do not assume:

```text
High port = suspicious
```

Do not assume:

```text
Ping failure = host down
```

Do not assume:

```text
Private IP = trusted
```

Do not assume:

```text
DNS request = connection occurred
```

Do not analyse a network event without endpoint context if endpoint telemetry is available.

---

# 202. Blue Team Network Mental Model

Think:

```text
IDENTITY
Who is the user?

PROCESS
What application generated traffic?

DNS
What name was resolved?

NETWORK
Where did it connect?

FILE
Was anything downloaded or created?

TIMELINE
What happened before and after?
```

---

# 203. Core Correlation Chain

One of the most useful patterns:

```text
Process Creation
      ↓
DNS Query
      ↓
Network Connection
      ↓
File Creation
      ↓
New Process
```

This can reveal:

```text
Execution
   ↓
Resolution
   ↓
Connection
   ↓
Download
   ↓
Execution
```

---

# 204. Quick Reference — Addressing

```text
127.0.0.1
Loopback
```

```text
169.254.0.0/16
APIPA
```

```text
10.0.0.0/8
Private
```

```text
172.16.0.0/12
Private
```

```text
192.168.0.0/16
Private
```

---

# 205. Quick Reference — TCP vs UDP

```text
TCP
Reliable
Connection-oriented
Ordered
Acknowledgements
```

```text
UDP
Connectionless
Lower overhead
No built-in guarantee
```

---

# 206. Quick Reference — Core Services

```text
22    SSH
53    DNS
80    HTTP
88    Kerberos
123   NTP
389   LDAP
443   HTTPS
445   SMB
636   LDAPS
3389  RDP
5985  WinRM
5986  WinRM TLS
```

---

# 207. Quick Reference — Layers

```text
Application
HTTP, DNS, SSH, SMB

Transport
TCP, UDP

Network
IP, ICMP

Data Link
Ethernet, MAC, VLAN

Physical
Cable, fiber, radio
```

---

# 208. Quick Reference — Troubleshooting

```text
Interface
   ↓
IP address
   ↓
Subnet
   ↓
Gateway
   ↓
Route
   ↓
DNS
   ↓
Port
   ↓
Application
```

---

# 209. Quick Reference — Investigation

```text
Source
Destination
Port
Protocol
Process
User
DNS
Time
Direction
Volume
```

---

# 210. Key Takeaway

Do not memorise networking as isolated facts.

Think in communication flows.

Instead of asking:

```text
What is port 443?
```

ask:

```text
Which process connected to port 443?

Which destination?

Why?

Was DNS used?

Was this normal for that process?

What happened before and after?
```

Instead of asking:

```text
What does DNS do?
```

think:

```text
Process
   ↓
DNS Query
   ↓
IP Resolution
   ↓
Network Connection
```

For Blue Team work, the most useful networking model is:

```text
WHO
  ↓
PROCESS
  ↓
SOURCE
  ↓
DESTINATION
  ↓
PROTOCOL
  ↓
PORT
  ↓
DNS
  ↓
TIMELINE
  ↓
CONTEXT
  ↓
VERDICT
```

Networking tells you where systems communicated.

Endpoint telemetry tells you what caused the communication.

Combining both is what turns network data into an investigation.
