![](Pasted%20image%2020260408101620.png)
This is the layout for the network

# Objective and scope
Darkhaven Technologies is a networking organization based throughout the world with locations in NY, CA, Japan, and more. They have segregated their network and would like to do a Red Team engagement to see if a user is able to move throughout the different networks.

A Close Access Team has infiltrated Darkhaven Technologies and dropped a machine for you on the internal network that you can connect to through OpenVPN. This machine should allow you to see the entire global network, as it was dropped on a port that is within the global VLAN. The Close Access Team relayed information that they overheard about the Web Portal being worked on at this time.

# Target Hosts
**dc.darkhaven.tech** - 10.10.10.4
**dc02.corp.darkhaven.tech** - 10.10.10.5
**web.ext.darkhaven.local** - 10.10.10.132
**ca.ext.darkhaven.local** - 10.10.10.134
**share.ext.darhaven.local** - 10.10.10.135
**sql.ext.darkhaven.local** - 10.10.10.133
**dc.ext.darkhaven.local** - 10.10.10.136
**VPN** - 10.10.10.137

# Enumeration
From the objective and scope its giving me a hint that the web server may be my initial access vector

## web.ext.darkhaven.local

### Open ports
```python
nmap -p- --min-rate=2000 -sT web.ext.darkhaven.local -Pn
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-08 10:28 -0400
Nmap scan report for web.ext.darkhaven.local (10.10.10.132)
Host is up (0.094s latency).
Not shown: 65528 filtered tcp ports (no-response)
PORT      STATE SERVICE
80/tcp    open  http
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
2107/tcp  open  msmq-mgmt
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
49680/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.71 seconds
```
### Nmap
```python

```