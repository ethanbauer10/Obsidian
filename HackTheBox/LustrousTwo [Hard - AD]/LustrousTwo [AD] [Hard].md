# Host file setup
```python
sudo nxc smb 10.129.242.166 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.242.166  445    LUS2DC           [*]  x64 (name:LUS2DC) (domain:Lustrous2.vl) (signing:True) (SMBv1:None) (NTLM:False)
```
- NTLM auth disabled
- SMB signing is enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT lus2dc.lustrous2.vl                                                       
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-07 10:24 -0400
Nmap scan report for lus2dc.lustrous2.vl (10.129.242.166)
Host is up (0.014s latency).
rDNS record for 10.129.242.166: LUS2DC.Lustrous2.vl
Not shown: 65512 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
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
3389/tcp  open  ms-wbt-server
9389/tcp  open  adws
49664/tcp open  unknown
49667/tcp open  unknown
49670/tcp open  unknown
49673/tcp open  unknown
49675/tcp open  unknown
49676/tcp open  unknown
49690/tcp open  unknown
49695/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.77 seconds
```

## Nmap
```python
nmap -p 21,53,80,88,135,139,389,445,464,593,636,3268,3269,3389,9389 -A --min-rate=2000 -sT lus2dc.lustrous2.vl   
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-07 10:28 -0400
Nmap scan report for lus2dc.lustrous2.vl (10.129.242.166)
Host is up (0.015s latency).
rDNS record for 10.129.242.166: LUS2DC.Lustrous2.vl

PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 09-06-24  05:20AM       <DIR>          Development
| 04-14-25  04:44AM       <DIR>          Homes
| 08-31-24  01:57AM       <DIR>          HR
| 08-31-24  01:57AM       <DIR>          IT
| 04-14-25  04:44AM       <DIR>          ITSEC
| 08-31-24  01:58AM       <DIR>          Production
|_08-31-24  01:58AM       <DIR>          SEC
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title.
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Negotiate
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-07 14:28:20Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: Lustrous2.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=LUS2DC.Lustrous2.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:LUS2DC.Lustrous2.vl
| Not valid before: 2025-09-29T14:23:23
|_Not valid after:  2026-09-29T14:23:23
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: Lustrous2.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=LUS2DC.Lustrous2.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:LUS2DC.Lustrous2.vl
| Not valid before: 2025-09-29T14:23:23
|_Not valid after:  2026-09-29T14:23:23
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: Lustrous2.vl, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=LUS2DC.Lustrous2.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:LUS2DC.Lustrous2.vl
| Not valid before: 2025-09-29T14:23:23
|_Not valid after:  2026-09-29T14:23:23
|_ssl-date: TLS randomness does not represent time
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: Lustrous2.vl, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=LUS2DC.Lustrous2.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:LUS2DC.Lustrous2.vl
| Not valid before: 2025-09-29T14:23:23
|_Not valid after:  2026-09-29T14:23:23
|_ssl-date: TLS randomness does not represent time
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=LUS2DC.Lustrous2.vl
| Not valid before: 2026-04-06T14:22:31
|_Not valid after:  2026-10-06T14:22:31
|_ssl-date: 2026-04-07T14:29:44+00:00; 0s from scanner time.
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: LUS2DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
NTLM authentication is disabled!

Guest account looks to be disabled

Since this im having to use kerberos auth i cant test null access either

# FTP (21)
```python
ftp lus2dc.lustrous2.vl                                        
Connected to LUS2DC.Lustrous2.vl.
220 Microsoft FTP Service
Name (lus2dc.lustrous2.vl:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ;s
?Invalid command.
ftp> ls
229 Entering Extended Passive Mode (|||49731|)
125 Data connection already open; Transfer starting.
09-06-24  05:20AM       <DIR>          Development
04-14-25  04:44AM       <DIR>          Homes
08-31-24  01:57AM       <DIR>          HR
08-31-24  01:57AM       <DIR>          IT
04-14-25  04:44AM       <DIR>          ITSEC
08-31-24  01:58AM       <DIR>          Production
08-31-24  01:58AM       <DIR>          SEC
226 Transfer complete.
```
I dont have write access here

```python
ftp> cd ../Homes
250 CWD command successful.
ftp> ls
229 Entering Extended Passive Mode (|||49736|)
125 Data connection already open; Transfer starting.
09-07-24  12:03AM       <DIR>          Aaron.Norman
09-07-24  12:03AM       <DIR>          Adam.Barnes
09-07-24  12:03AM       <DIR>          Amber.Ward
09-07-24  12:03AM       <DIR>          Andrea.Smith
09-07-24  12:03AM       <DIR>          Ann.Lynch
09-07-24  12:03AM       <DIR>          Callum.Oliver
09-07-24  12:03AM       <DIR>          Carly.Walker
09-07-24  12:03AM       <DIR>          Chelsea.Smith
09-07-24  12:03AM       <DIR>          Chloe.Hammond
09-07-24  12:03AM       <DIR>          Christopher.Lawson
09-07-24  12:03AM       <DIR>          Claire.Parry
09-07-24  12:03AM       <DIR>          Darren.Lewis
09-07-24  12:03AM       <DIR>          Deborah.Jones
09-07-24  12:03AM       <DIR>          Dominic.West
09-07-24  12:03AM       <DIR>          Duncan.Smith
09-07-24  12:03AM       <DIR>          Elaine.Gallagher
09-07-24  12:03AM       <DIR>          Eleanor.Gregory
09-07-24  12:03AM       <DIR>          Emma.Bell
09-07-24  12:03AM       <DIR>          Francesca.Norman
09-07-24  12:03AM       <DIR>          Gary.Richards
09-07-24  12:03AM       <DIR>          Gerard.Ward
09-07-24  12:03AM       <DIR>          Glenn.Williams
09-07-24  12:03AM       <DIR>          Graeme.Pritchard
09-07-24  12:03AM       <DIR>          Harriet.Richardson
09-07-24  12:03AM       <DIR>          Henry.Connor
09-07-24  12:03AM       <DIR>          Howard.Robinson
09-07-24  12:03AM       <DIR>          Jacqueline.Phillips
09-07-24  12:03AM       <DIR>          Janice.Collier
09-07-24  12:03AM       <DIR>          Jasmine.Johnson
09-07-24  12:03AM       <DIR>          Joan.Wall
09-07-24  12:03AM       <DIR>          Judith.Francis
09-07-24  12:03AM       <DIR>          Justin.Williams
09-07-24  12:03AM       <DIR>          Kyle.Hussain
09-07-24  12:03AM       <DIR>          Kyle.Lloyd
09-07-24  12:03AM       <DIR>          Lawrence.Bryan
09-07-24  12:03AM       <DIR>          Leah.Elliott
09-07-24  12:03AM       <DIR>          Lewis.Khan
09-07-24  12:03AM       <DIR>          Liam.Wheeler
09-07-24  12:03AM       <DIR>          Lisa.Begum
09-07-24  12:03AM       <DIR>          Louis.Phillips
09-07-24  12:03AM       <DIR>          Lydia.Parker
09-07-24  12:03AM       <DIR>          Malcolm.Yates
09-07-24  12:03AM       <DIR>          Marie.Hill
09-07-24  12:03AM       <DIR>          Martin.Hamilton
09-07-24  12:03AM       <DIR>          Mathew.Roberts
09-07-24  12:03AM       <DIR>          Melissa.Thompson
09-07-24  12:03AM       <DIR>          Nathan.Carter
09-07-24  12:03AM       <DIR>          Nicola.Clarke
09-07-24  12:03AM       <DIR>          Nicola.Hall
09-07-24  12:03AM       <DIR>          Nigel.Lee
09-07-24  12:03AM       <DIR>          Pamela.Taylor
09-07-24  12:03AM       <DIR>          Robert.Russell
09-07-24  12:03AM       <DIR>          Ryan.Davies
09-07-24  12:03AM       <DIR>          Ryan.Moore
09-07-24  12:03AM       <DIR>          Ryan.Rowe
09-07-24  12:03AM       <DIR>          Samantha.Smith
09-07-24  12:03AM       <DIR>          Sara.Matthews
09-07-24  12:03AM       <DIR>          ShareSvc
09-07-24  12:03AM       <DIR>          Sharon.Birch
09-07-24  12:03AM       <DIR>          Sharon.Evans
09-07-24  12:03AM       <DIR>          Stacey.Barber
09-07-24  12:03AM       <DIR>          Stacey.Griffiths
09-07-24  12:03AM       <DIR>          Stephanie.Baxter
09-07-24  12:03AM       <DIR>          Stephanie.Davies
09-07-24  12:03AM       <DIR>          Steven.Sutton
09-07-24  12:03AM       <DIR>          Susan.Johnson
09-07-24  12:03AM       <DIR>          Terence.Jordan
09-07-24  12:03AM       <DIR>          Thomas.Myers
09-07-24  12:03AM       <DIR>          Tony.Davies
09-07-24  12:03AM       <DIR>          Victoria.Williams
09-07-24  12:03AM       <DIR>          Wayne.Taylor
```
This could be a list of users not confirmed!

```python
ftp> cd ITSEC
250 CWD command successful.
ftp> ls
229 Entering Extended Passive Mode (|||49751|)
125 Data connection already open; Transfer starting.
09-07-24  03:50AM                  207 audit_draft.txt
226 Transfer complete.
ftp> get audit_draft.txt
local: audit_draft.txt remote: audit_draft.txt
229 Entering Extended Passive Mode (|||49755|)
125 Data connection already open; Transfer starting.
100% |**************************************************************************|   207       13.92 KiB/s    00:00 ETA
226 Transfer complete.
207 bytes received in 00:00 (13.00 KiB/s)
```
Found a text file

```python

```