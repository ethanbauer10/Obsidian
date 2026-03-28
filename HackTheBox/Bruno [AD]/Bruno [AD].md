# Host file setup
```python
sudo nxc smb 10.129.238.9 --generate-hosts-file /etc/hosts            
[sudo] password for kali: 
SMB         10.129.238.9    445    BRUNODC          [*] Windows Server 2022 Build 20348 x64 (name:BRUNODC) (domain:bruno.vl) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.238.9          
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-27 18:11 +0000
Nmap scan report for 10.129.238.9
Host is up (0.015s latency).
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
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
3389/tcp  open  ms-wbt-server
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
56106/tcp open  unknown
56109/tcp open  unknown
63099/tcp open  unknown
63104/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.33 seconds
```

## Nmap
```python
nmap -p 21,53,80,88,135,139,389,443,445,464,593,636,3268,3269,3389,9389 -A --min-rate=2000 -sT brunodc.bruno.vl
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-27 18:14 +0000
Nmap scan report for brunodc.bruno.vl (10.129.238.9)
Host is up (0.015s latency).
rDNS record for 10.129.238.9: BRUNODC.bruno.vl

PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 06-29-22  04:55PM       <DIR>          app
| 06-29-22  04:33PM       <DIR>          benign
| 06-29-22  01:41PM       <DIR>          malicious
|_06-29-22  04:33PM       <DIR>          queue
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-27 18:14:11Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: bruno.vl, Site: Default-First-Site-Name)
|_ssl-date: 2026-03-27T18:15:35+00:00; 0s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:brunodc.bruno.vl, DNS:bruno.vl, DNS:BRUNO
| Not valid before: 2025-10-09T09:54:08
|_Not valid after:  2105-10-09T09:54:08
443/tcp  open  ssl/https?
| tls-alpn: 
|   h2
|_  http/1.1
| ssl-cert: Subject: commonName=bruno-BRUNODC-CA
| Not valid before: 2022-06-29T13:23:01
|_Not valid after:  2121-06-29T13:33:00
|_ssl-date: TLS randomness does not represent time
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap
|_ssl-date: 2026-03-27T18:15:35+00:00; 0s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:brunodc.bruno.vl, DNS:bruno.vl, DNS:BRUNO
| Not valid before: 2025-10-09T09:54:08
|_Not valid after:  2105-10-09T09:54:08
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: bruno.vl, Site: Default-First-Site-Name)
|_ssl-date: 2026-03-27T18:15:35+00:00; 0s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:brunodc.bruno.vl, DNS:bruno.vl, DNS:BRUNO
| Not valid before: 2025-10-09T09:54:08
|_Not valid after:  2105-10-09T09:54:08
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: bruno.vl, Site: Default-First-Site-Name)
|_ssl-date: 2026-03-27T18:15:35+00:00; 0s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:brunodc.bruno.vl, DNS:bruno.vl, DNS:BRUNO
| Not valid before: 2025-10-09T09:54:08
|_Not valid after:  2105-10-09T09:54:08
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-03-27T18:15:35+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=brunodc.bruno.vl
| Not valid before: 2026-03-26T18:09:34
|_Not valid after:  2026-09-25T18:09:34
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: BRUNODC; OS: Windows; CPE: cpe:/o:microsoft:windows
```
The web server looks to be just default IIS and the same it true on port 443

# SMB (445)
Null auth is enabled but not able to access shares of list users

Guest account is disabled

# HTTP (80,443)
Both pages are default IIS landing pages, nothing too interesting there

# FTP (21)
```python
ftp brunodc.bruno.vl                             
Connected to BRUNODC.bruno.vl.
220 Microsoft FTP Service
Name (brunodc.bruno.vl:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||53578|)
150 Opening ASCII mode data connection.
06-29-22  04:55PM       <DIR>          app
06-29-22  04:33PM       <DIR>          benign
06-29-22  01:41PM       <DIR>          malicious
06-29-22  04:33PM       <DIR>          queue
```
Both `malicious` and `queue` are empty

`benign` had a test.exe file which was nothing

Inside app there is a changelog with a user `svc_scan`

I could try some AS-REP roast 
