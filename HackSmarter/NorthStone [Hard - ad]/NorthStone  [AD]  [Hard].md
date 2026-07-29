# Objective and Scope
North Stone, a premier luxury real estate marketplace, has engaged Hack Smarter for a full-scope penetration test. Your objective: identify critical vulnerabilities and demonstrate real-world business impact by achieving complete domain compromise.

The client has provided you with VPN access to their network, but no credentials.

I have also been provided a wordlist to use!

# Host file setup

```python
sudo nxc smb 10.1.209.181 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.1.209.181    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:northstone.local) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT dc.northstone.local                                        
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-29 17:11 +0100
Nmap scan report for dc.northstone.local (10.1.209.181)
Host is up (0.097s latency).
rDNS record for 10.1.209.181: DC.northstone.local
Not shown: 65519 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
47001/tcp open  winrm
49669/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
49679/tcp open  unknown
55203/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.37 seconds
```

After another check, port `88` and port `5985` are also open for some reason nmap missed it!
## Nmap
```python
nmap -p 53,80,88,135,139,389,445,636,3268,3269,3389 -A --min-rate=2000 -sT dc.northstone.local
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-29 17:13 +0100
Nmap scan report for dc.northstone.local (10.1.209.181)
Host is up (0.097s latency).
rDNS record for 10.1.209.181: DC.northstone.local

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: NorthStone | Coming Soon - US Real Estate Portal
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-29 16:13:50Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: northstone.local, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-29T16:15:18+00:00; -1s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC.northstone.local, DNS:northstone.local, DNS:NORTHSTONE
| Not valid before: 2026-05-02T23:26:31
|_Not valid after:  2027-05-02T23:26:31
445/tcp  open  microsoft-ds?
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: northstone.local, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-29T16:15:18+00:00; 0s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC.northstone.local, DNS:northstone.local, DNS:NORTHSTONE
| Not valid before: 2026-05-02T23:26:31
|_Not valid after:  2027-05-02T23:26:31
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: northstone.local, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-29T16:15:18+00:00; -1s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC.northstone.local, DNS:northstone.local, DNS:NORTHSTONE
| Not valid before: 2026-05-02T23:26:31
|_Not valid after:  2027-05-02T23:26:31
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: northstone.local, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-29T16:15:18+00:00; 0s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC.northstone.local, DNS:northstone.local, DNS:NORTHSTONE
| Not valid before: 2026-05-02T23:26:31
|_Not valid after:  2027-05-02T23:26:31
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-07-29T16:15:18+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=DC.northstone.local
| Not valid before: 2026-06-16T18:37:19
|_Not valid after:  2026-12-16T18:37:19
| rdp-ntlm-info: 
|   Target_Name: NORTHSTONE
|   NetBIOS_Domain_Name: NORTHSTONE
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: northstone.local
|   DNS_Computer_Name: DC.northstone.local
|   DNS_Tree_Name: northstone.local
|   Product_Version: 10.0.17763
|_  System_Time: 2026-07-29T16:14:39+00:00
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (92%), Microsoft Windows 10 1903 - 22H2 (85%), Microsoft Windows 10 1607 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)

