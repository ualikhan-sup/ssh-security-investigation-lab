# 🔐 SSH Security Investigation Lab

A hands-on cybersecurity home lab focused on network reconnaissance, TCP traffic analysis, SSH authentication monitoring, and Linux log investigation.

The lab simulates basic activities that a Junior SOC / Security Analyst may encounter: identifying exposed services, analyzing network traffic, investigating failed authentication attempts, and correlating network activity with system logs.



##  Lab Environment

The environment was built in VirtualBox using two isolated virtual machines:

 **Kali Linux** — 192.168.56.10
 **Ubuntu Server** — 192.168.56.20
 **Internal Network:** 192.168.56.0/24
 **SSH Service:** TCP/22

The machines communicate through an isolated VirtualBox internal network.

```text
Kali Linux
192.168.56.10
     |
     |  Nmap / SSH / Network Traffic
     |
     v
Ubuntu Server
192.168.56.20
     |
     └── OpenSSH : TCP/22
