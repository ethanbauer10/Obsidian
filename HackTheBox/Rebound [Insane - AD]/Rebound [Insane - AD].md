# Host file setup
```python
sudo nxc smb 10.129.14.17 --generate-hosts-file /etc/hosts                                     
[sudo] password for kali: 
SMB         10.129.14.17    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```
Few things note:
- Null auth is true
- SMB signing is enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.rebound.htb                                                        
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-04 14:53 +0100
Nmap scan report for dc01.rebound.htb (10.129.14.17)
Host is up (0.015s latency).
rDNS record for 10.129.14.17: DC01.rebound.htb
Not shown: 65509 closed tcp ports (conn-refused)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49673/tcp open  unknown
49690/tcp open  unknown
49691/tcp open  unknown
49696/tcp open  unknown
49709/tcp open  unknown
49724/tcp open  unknown
49743/tcp open  unknown
59390/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 13.68 seconds
```
## Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT dc01.rebound.htb
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-04 14:55 +0100
Nmap scan report for dc01.rebound.htb (10.129.14.17)
Host is up (0.015s latency).
rDNS record for 10.129.14.17: DC01.rebound.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-04 20:55:15Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: rebound.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-04T20:56:09+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.rebound.htb, DNS:rebound.htb, DNS:rebound
| Not valid before: 2025-03-06T19:51:11
|_Not valid after:  2122-04-08T14:05:49
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: rebound.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-04T20:56:09+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.rebound.htb, DNS:rebound.htb, DNS:rebound
| Not valid before: 2025-03-06T19:51:11
|_Not valid after:  2122-04-08T14:05:49
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: rebound.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-04T20:56:09+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.rebound.htb, DNS:rebound.htb, DNS:rebound
| Not valid before: 2025-03-06T19:51:11
|_Not valid after:  2122-04-08T14:05:49
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: rebound.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-04T20:56:09+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.rebound.htb, DNS:rebound.htb, DNS:rebound
| Not valid before: 2025-03-06T19:51:11
|_Not valid after:  2122-04-08T14:05:49
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 10|2019|2012|2022|2016 (96%)
OS CPE: cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows 10 1909 - 2004 (96%), Microsoft Windows Server 2019 (95%), Windows Server 2019 (92%), Microsoft Windows 10 1709 - 21H2 (91%), Microsoft Windows 10 1909 (91%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Server 2022 (91%), Microsoft Windows 10 20H2 (90%), Microsoft Windows 10 20H2 - 21H1 (90%), Microsoft Windows 10 1903 - 21H1 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```
Nmap tell me the DC is running at +7h

# SMB (445)
Null auth is enabled but not able to use it to access shares or 