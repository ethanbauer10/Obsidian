# Host file setup

```python
sudo nxc smb 10.129.230.109 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.230.109  445    hathor           [*]  x64 (name:hathor) (domain:windcorp.htb) (signing:True) (SMBv1:None) (NTLM:False)
```
- NTLM auth is disabled!
- SMB signing is enabled

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT hathor.windcorp.htb  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-17 17:46 +0100
Nmap scan report for hathor.windcorp.htb (10.129.230.109)
Host is up (0.012s latency).
Not shown: 65516 filtered tcp ports (no-response)
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
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49667/tcp open  unknown
54175/tcp open  unknown
61498/tcp open  unknown
61517/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.77 seconds
```

## Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT hathor.windcorp.htb  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-17 17:49 +0100
Nmap scan report for hathor.windcorp.htb (10.129.230.109)
Host is up (0.013s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: Home - mojoPortal
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
| http-robots.txt: 29 disallowed entries (15 shown)
| /CaptchaImage.ashx* /Admin/ /App_Browsers/ /App_Code/ 
| /App_Data/ /App_Themes/ /bin/ /Blog/ViewCategory.aspx$ 
| /Blog/ViewArchive.aspx$ /Data/SiteImages/emoticons /MyPage.aspx 
|_/MyPage.aspx$ /MyPage.aspx* /NeatHtml/ /NeatUpload/
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-05-17 16:49:10Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: windcorp.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=hathor.windcorp.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:hathor.windcorp.htb
| Not valid before: 2026-05-17T16:35:46
|_Not valid after:  2027-05-17T16:35:46
|_ssl-date: 2026-05-17T16:50:34+00:00; +4s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: windcorp.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-05-17T16:50:34+00:00; +4s from scanner time.
| ssl-cert: Subject: commonName=hathor.windcorp.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:hathor.windcorp.htb
| Not valid before: 2026-05-17T16:35:46
|_Not valid after:  2027-05-17T16:35:46
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: windcorp.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-05-17T16:50:34+00:00; +4s from scanner time.
| ssl-cert: Subject: commonName=hathor.windcorp.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:hathor.windcorp.htb
| Not valid before: 2026-05-17T16:35:46
|_Not valid after:  2027-05-17T16:35:46
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: windcorp.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=hathor.windcorp.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:hathor.windcorp.htb
| Not valid before: 2026-05-17T16:35:46
|_Not valid after:  2027-05-17T16:35:46
|_ssl-date: 2026-05-17T16:50:34+00:00; +4s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: HATHOR; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)

Since NTLM auth is disabled, i cannot use null authenticaton

The guest account also appears to be disabled!

# HTTP (80)
