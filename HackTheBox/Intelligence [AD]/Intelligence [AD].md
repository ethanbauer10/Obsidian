# Host file setup
```python
sudo nxc smb 10.129.95.154 --generate-hosts-file /etc/hosts                                                                  
[sudo] password for kali: 
SMB         10.129.95.154   445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```
- SMB signing is enabled
- Null auth enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc.intelligence.htb   
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-14 15:29 +0100
Nmap scan report for dc.intelligence.htb (10.129.95.154)
Host is up (0.015s latency).
rDNS record for 10.129.95.154: DC.intelligence.htb
Not shown: 65517 filtered tcp ports (no-response)
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
49691/tcp open  unknown
49692/tcp open  unknown
49711/tcp open  unknown
49717/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.79 seconds
```
## Nmap
```python
nmap -p 53,80,88,135,139,445,464,593,636,3268,3269,9389 -A --min-rate=2000 -sT dc.intelligence.htb
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-14 15:31 +0100
Nmap scan report for dc.intelligence.htb (10.129.95.154)
Host is up (0.015s latency).
rDNS record for 10.129.95.154: DC.intelligence.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: Intelligence
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-14 21:31:55Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-14T21:33:20+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-14T21:33:20+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-14T21:33:20+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.intelligence.htb
| Not valid before: 2021-04-19T00:43:16
|_Not valid after:  2022-04-19T00:43:16
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (96%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (96%), Microsoft Windows 10 1903 - 21H1 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Null auth is enabled cannot list shares or users

Guest account is disabled

# HTTP (80)
## Nuclei
```python
nuclei -u http://intelligence.htb/                                                                

[iis-shortname-detect] [http] [info] http://intelligence.htb/*~1*/a.aspx
[iis-shortname-detect] [http] [info] http://intelligence.htb/jta3iyc9*~1*/a.aspx
[iis-shortname-detect] [http] [info] http://intelligence.htb/*~1*/a.aspx
[waf-detect:modsecurity] [http] [info] http://intelligence.htb/
[ldap-metadata] [javascript] [info] intelligence.htb:389 ["BaseDN: dc=389","DnsHostName: dc.intelligence.htb","DefaultNamingContext: DC=intelligence,DC=htb","DomainFunctionality: 7","ForestFunctionality: 7","DomainControllerFunctionality: 7"]
[smb-enum] [javascript] [info] intelligence.htb:445 ["OSVersion: 10.0.17763","NetBIOSComputerName: DC","NetBIOSDomainName: intelligence","DNSComputerNamen: dc.intelligence.htb","DNSComputerName: dc.intelligence.htb","ForestName: intelligence.htb"]
[smb-enum-domains] [javascript] [info] intelligence.htb:445 ["DomainName: intelligence.htb"]
[smb-os-detect] [javascript] [info] intelligence.htb:445 ["Windows Server 2019, Version 1809"]
[smb-version-detect:smb-version] [javascript] [info] intelligence.htb:445 ["SMB 2.1"]
[smb2-capabilities] [javascript] [info] intelligence.htb:445 ["["DFSSupport","LargeMTU","Leasing"]"]
[smb2-server-time] [javascript] [info] intelligence.htb:445 ["SystemTime: 2026-04-14T21:39:27.000Z ServerStartTime: 2009-04-22T19:24:48.000Z"]
[ldap-anonymous-login-detect] [javascript] [medium] intelligence.htb:389
[form-detection] [http] [info] http://intelligence.htb/
[microsoft-iis-version] [http] [info] http://intelligence.htb/ ["Microsoft-IIS/10.0"]
[http-missing-security-headers:clear-site-data] [http] [info] http://intelligence.htb/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://intelligence.htb/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://intelligence.htb/
[http-missing-security-headers:referrer-policy] [http] [info] http://intelligence.htb/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://intelligence.htb/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://intelligence.htb/
[http-missing-security-headers:missing-content-type] [http] [info] http://intelligence.htb/
[http-missing-security-headers:strict-transport-security] [http] [info] http://intelligence.htb/
[http-missing-security-headers:content-security-policy] [http] [info] http://intelligence.htb/
[http-missing-security-headers:permissions-policy] [http] [info] http://intelligence.htb/
[http-missing-security-headers:x-frame-options] [http] [info] http://intelligence.htb/
[http-missing-security-headers:x-content-type-options] [http] [info] http://intelligence.htb/
[options-method] [http] [info] http://intelligence.htb/ ["OPTIONS, TRACE, GET, HEAD, POST"]
[tech-detect:ms-iis] [http] [info] http://intelligence.htb/
[caa-fingerprint] [dns] [info] intelligence.htb
```
## Ffuf for subdomains
```python
nothing found
```
## Feroxbuster
```python
feroxbuster -u http://intelligence.htb/ -C 404 

200      GET       56l      165w     1850c http://intelligence.htb/documents/scripts.js
200      GET        1l       44w     2532c http://intelligence.htb/documents/jquery.easing.min.js
200      GET        8l       29w    28898c http://intelligence.htb/documents/favicon.ico
200      GET      106l      659w    26989c http://intelligence.htb/documents/demo-image-01.jpg
200      GET      208l      768w    47856c http://intelligence.htb/documents/2020-01-01-upload.pdf
200      GET      209l      800w    48542c http://intelligence.htb/documents/2020-12-15-upload.pdf
200      GET        7l     1031w    84152c http://intelligence.htb/documents/bootstrap.bundle.min.js
200      GET        2l     1297w    89476c http://intelligence.htb/documents/jquery.min.js
301      GET        2l       10w      157c http://intelligence.htb/documents => http://intelligence.htb/documents/
403      GET       29l       92w     1233c http://intelligence.htb/documents/
200      GET      492l     2733w   186437c http://intelligence.htb/documents/demo-image-02.jpg
200      GET    10345l    19793w   190711c http://intelligence.htb/documents/styles.css
301      GET        2l       10w      157c http://intelligence.htb/Documents => http://intelligence.htb/Documents/
200      GET        5l   108280w  1194960c http://intelligence.htb/documents/all.js
200      GET      129l      430w     7432c http://intelligence.htb/
301      GET        2l       10w      157c http://intelligence.htb/DOCUMENTS => http://intelligence.htb/DOCUMENTS/
400      GET        6l       26w      324c http://intelligence.htb/error%1F_log
400      GET        6l       26w      324c http://intelligence.htb/documents/error%1F_log
400      GET        6l       26w      324c http://intelligence.htb/Documents/error%1F_log
400      GET        6l       26w      324c http://intelligence.htb/DOCUMENTS/error%1F_log
```
Documents directory gives a 403 but i can access files inside

## Website functionality
![](Pasted%20image%2020260414154349.png)
Subscription functionality

Two PDF downloads on the landing page of the website

`http://intelligence.htb/documents/2020-01-01-upload.pdf`

`http://intelligence.htb/documents/2020-12-15-upload.pdf`

```python
exiftool 2020-01-01-upload.pdf          
ExifTool Version Number         : 13.50
File Name                       : 2020-01-01-upload.pdf
Directory                       : .
File Size                       : 27 kB
File Modification Date/Time     : 2021:04:01 18:00:00+01:00
File Access Date/Time           : 2026:04:14 15:48:23+01:00
File Inode Change Date/Time     : 2026:04:14 15:48:23+01:00
File Permissions                : -rw-rw-r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 1
Creator                         : William.Lee
```
Found a user `william.lee`



