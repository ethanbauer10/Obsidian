# Host file setup
```python
sudo nxc smb 10.129.228.120 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.228.120  445    G0               [*] Windows 10 / Server 2019 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```
SMB signing enabled

Null auth enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT g0.flight.htb -Pn
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-31 15:59 +0100
Nmap scan report for g0.flight.htb (10.129.228.120)
Host is up (0.014s latency).
rDNS record for 10.129.228.120: G0.flight.htb
Not shown: 65518 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
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
9389/tcp  open  adws
49667/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49695/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.70 seconds
```

## Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,9389 -A --min-rate=2000 -sT g0.flight.htb
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-31 16:01 +0100
Nmap scan report for g0.flight.htb (10.129.228.120)
Host is up (0.014s latency).
rDNS record for 10.129.228.120: G0.flight.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
|_http-title: g0 Aviation
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-31 22:01:27Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: G0; OS: Windows; CPE: cpe:/o:microsoft:windows
```
Web server is running apache

