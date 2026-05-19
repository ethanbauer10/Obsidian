
# Host file setup

```python
sudo nxc smb 10.129.229.57 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.229.57   445    RESEARCH         [*] Windows 10 / Server 2019 Build 17763 x64 (name:RESEARCH) (domain:search.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```
- Windows version tells me this is going to be vulnerable to ZeroLogon

# Enumeration

## Open ports

```python
nmap -p- --min-rate=2000 -sT research.search.htb                                                              
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-19 18:14 +0100
Nmap scan report for research.search.htb (10.129.229.57)
Host is up (0.013s latency).
rDNS record for 10.129.229.57: RESEARCH.search.htb
Not shown: 65514 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
443/tcp   open  https
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
8172/tcp  open  unknown
9389/tcp  open  adws
49667/tcp open  unknown
49693/tcp open  unknown
49694/tcp open  unknown
49710/tcp open  unknown
49719/tcp open  unknown
49745/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.28 seconds
```

## Nmap

```python
nmap -p 53,80,88,135,139,389,443,445,464,593,636,3268,3269,8172,9389 -A --min-rate=2000 -sT research.search.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-19 18:17 +0100
Nmap scan report for research.search.htb (10.129.229.57)
Host is up (0.013s latency).
rDNS record for 10.129.229.57: RESEARCH.search.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: Search &mdash; Just Testing IIS
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-05-19 17:17:41Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: search.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-05-19T17:19:38+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
443/tcp  open  ssl/http      Microsoft IIS httpd 10.0
| tls-alpn: 
|   h2
|_  http/1.1
|_http-server-header: Microsoft-IIS/10.0
|_ssl-date: 2026-05-19T17:19:38+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: search.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-05-19T17:19:38+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: search.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-05-19T17:19:38+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: search.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
|_ssl-date: 2026-05-19T17:19:38+00:00; -1s from scanner time.
8172/tcp open  ssl/unknown
| ssl-cert: Subject: commonName=WMSvc-SHA2-RESEARCH
| Not valid before: 2020-04-07T09:05:25
|_Not valid after:  2030-04-05T09:05:25
|_ssl-date: 2026-05-19T17:19:38+00:00; -1s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1903 - 22H2 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: RESEARCH; OS: Windows; CPE: cpe:/o:microsoft:windows
```


# SMB (445)

Null auth is enabled, but not able to use it to list shares or list users

Guest account is also disabled!

# HTTP (80)

The website appears to be mostly sta