# SSH Security Investigation Lab

This is a small home lab I built to practice basic network and Linux security analysis.

The setup uses two virtual machines in VirtualBox:

- Kali Linux — 192.168.56.10
- Ubuntu Server — 192.168.56.20
- Internal network — 192.168.56.0/24
- SSH — TCP/22

The main idea was to look at the same activity from different sides: network scanning from Kali, packet analysis in Wireshark, and authentication/system logs on Ubuntu.

Lab setup

Kali Linux was used as the client / analysis machine.

Ubuntu Server was used as the target system with OpenSSH enabled.

```text
Kali Linux
192.168.56.10
     |
     |  Nmap / SSH / Wireshark
     |
Ubuntu Server
192.168.56.20
     |
     └── OpenSSH : TCP/22

Tools used
- Nmap
- Wireshark
- tcpdump
- OpenSSH
- journalctl
- grep
- sort
- uniq
- ss
- Git
- GitHub
What I practiced
I started by checking connectivity between the two machines and then scanned the Ubuntu host from Kali.
For example:
sudo nmap -sS -p 22 192.168.56.20

I also checked the SSH service version:
sudo nmap -sV -p 22 192.168.56.20

On Ubuntu, I checked which ports were actually listening:
sudo ss -tulpn

This helped me understand the difference between:
- what the server is listening on locally
- what another machine can see over the network
Packet analysis
I captured the traffic in Wireshark and looked at the TCP behavior during an Nmap SYN scan.
For an open SSH port I observed:
SYN
SYN, ACK
RST

For a closed port I observed:
SYN
RST, ACK

This made it easier to understand how Nmap identifies open and closed TCP ports.
SSH authentication
I generated several failed SSH login attempts and then a successful login.
On Ubuntu I checked the SSH logs with:
sudo journalctl -u ssh --no-pager

To see only failed logins:
sudo journalctl -u ssh --no-pager | grep "Failed password"

To see successful logins:
sudo journalctl -u ssh --no-pager | grep "Accepted"

To see both:
sudo journalctl -u ssh --no-pager | grep -E "Failed password|Accepted"

From these logs I could identify:
- source IP
- username
- failed login attempts
- successful login events
- event timeline
Basic log analysis
I also used a simple pipeline to extract source IP addresses from failed SSH attempts and count them:
sudo journalctl -u ssh --no-pager \
| grep "Failed password" \
| grep -oP 'from \K[0-9.]+' \
| sort \
| uniq -c \
| sort -nr

Post-login activity
After a successful SSH login, I also checked sudo activity.
This helped me see which commands were executed with elevated privileges and connect authentication events with activity after login.
Evidence
The repository contains saved log output in:
evidence/
├── ssh-auth.log
└── sudo-commands.log

Screenshots
Nmap service discovery
[Nmap service discovery](screenshots/01-nmap-ssh-service-discovery.png)
Ubuntu listening ports
[Ubuntu listening ports](screenshots/02-ubuntu-ss-listening-ports.png
Wireshark SYN scan
[Wireshark SYN scan](screenshots/03-wireshark-syn-scan.png)
SSH authentication logs
[SSH authentication logs](screenshots/04-ssh-authentication-logs.png
 
What I learned
This lab helped me get more comfortable with:
- basic TCP/IP behavior
- Nmap scanning
- Wireshark packet analysis
- SSH authentication
- Linux logs
- journalctl
- grep pipelines
- source IP extraction
- simple event counting
- basic correlation between network traffic and host logs
