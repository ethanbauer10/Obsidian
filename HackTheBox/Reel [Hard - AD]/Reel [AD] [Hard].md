# Host file setup
```python
sudo nxc smb 10.129.16.113 --generate-hosts-file /etc/hosts                                                 
SMB         10.129.16.113   445    REEL             [*] Windows Server 2012 R2 Standard 9600 x64 (name:REEL) (domain:HTB.LOCAL) (signing:True) (SMBv1:True) (Null Auth:True)
```
- Null auth is enabled
- SMBv1
- SMB signing enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn reel.htb.local         
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-09 11:18 -0400
Nmap scan report for reel.htb.local (10.129.16.113)
Host is up (0.014s latency).
rDNS record for 10.129.16.113: REEL.HTB.LOCAL
Not shown: 65527 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
25/tcp    open  smtp
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
593/tcp   open  http-rpc-epmap
49159/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.70 seconds
```
Interesting port setup for a windows machine
## Nmap
```python
nmap -p 21,22,25,135,139,445,593,49159 -A --min-rate=2000 -sT -Pn reel.htb.local
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-09 11:20 -0400
Stats: 0:03:20 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.91% done; ETC: 11:24 (0:00:00 remaining)
Nmap scan report for reel.htb.local (10.129.16.113)
Host is up (0.015s latency).
rDNS record for 10.129.16.113: REEL.HTB.LOCAL

PORT      STATE SERVICE      VERSION
21/tcp    open  ftp          Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_05-29-18  12:19AM       <DIR>          documents
22/tcp    open  ssh          OpenSSH 7.6 (protocol 2.0)
| ssh-hostkey: 
|   2048 82:20:c3:bd:16:cb:a2:9c:88:87:1d:6c:15:59:ed:ed (RSA)
|   256 23:2b:b8:0a:8c:1c:f4:4d:8d:7e:5e:64:58:80:33:45 (ECDSA)
|_  256 ac:8b:de:25:1d:b7:d8:38:38:9b:9c:16:bf:f6:3f:ed (ED25519)
25/tcp    open  smtp?
| smtp-commands: REEL, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Kerberos, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, X11Probe: 
|     220 Mail Service ready
|   FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, RTSPRequest: 
|     220 Mail Service ready
|     sequence of commands
|     sequence of commands
|   Hello: 
|     220 Mail Service ready
|     EHLO Invalid domain address.
|   Help: 
|     220 Mail Service ready
|     DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
|   SIPOptions: 
|     220 Mail Service ready
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|   TerminalServerCookie: 
|     220 Mail Service ready
|_    sequence of commands
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2012 R2 Standard 9600 microsoft-ds (workgroup: HTB)
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49159/tcp open  msrpc        Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2012|2008|7 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7
Aggressive OS guesses: Microsoft Windows Server 2012 R2 (97%), Microsoft Windows 7 or Windows Server 2008 R2 (91%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: REEL; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# FTP (21)
From the nmap scan i know the service allows anonymous logon

```python
ftp reel.htb.local                                   
Connected to REEL.HTB.LOCAL.
220 Microsoft FTP Service
Name (reel.htb.local:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||41001|)
125 Data connection already open; Transfer starting.
05-29-18  12:19AM       <DIR>          documents
226 Transfer complete.
ftp> cd documents 
250 CWD command successful.
ftp> dir
229 Entering Extended Passive Mode (|||41002|)
125 Data connection already open; Transfer starting.
05-29-18  12:19AM                 2047 AppLocker.docx
05-28-18  02:01PM                  124 readme.txt
10-31-17  10:13PM                14581 Windows Event Forwarding.docx
226 Transfer complete.
```
Ill transfer these files to my machine

Cant write anywhere here

## AppLocker.docx
```python
AppLocker procedure to be documented - hash rules for exe, msi and scripts (ps1,vbs,cmd,bat,js) are in effect.
```

## readme.txt
```python
cat readme.txt                                                                    
please email me any rtf format procedures - I'll review and convert.

new format / converted documents will be saved here.
```

## Windows Event Forwarding.docx
```python

# get winrm config

winrm get winrm/config


# gpo config

O:BAG:SYD:(A;;0xf0005;;;SY)(A;;0x5;;;BA)(A;;0x1;;;S-1-5-32-573)(A;;0x1;;;NS)		// add to GPO
Server=http://WEF.HTB.LOCAL:5985/wsman/SubscriptionManager/WEC,Refresh=60	// add to GPO (60 seconds)

on source computer: gpupdate /force

# prereqs

start Windows Remote Management service on source computer
add builtin\network service account to "Event Log Readers" group on collector server

# list subscriptions / export

C:\Windows\system32>wecutil es > subs.txt

# check subscription status

C:\Windows\system32>wecutil gr "Account Currently Disabled"

Subscription: Account Currently Disabled
        RunTimeStatus: Active
        LastError: 0
        EventSources:
                LAPTOP12.HTB.LOCAL
                        RunTimeStatus: Active
                        LastError: 0
                        LastHeartbeatTime: 2017-07-11T13:27:00.920


# change pre-rendering setting in multiple subscriptions

for /F "tokens=*" %i in (subs.txt) DO wecutil ss "%i" /cf:Events


# export subscriptions to xml

for /F "tokens=*" %i in (subs.txt) DO wecutil gs "%i" /f:xml >> "%i.xml"

# import subscriptions from xml

wecutil cs "Event Log Service Shutdown.xml"
wecutil cs "Event Log was cleared.xml"

# if get error "The locale specific resource for the desired message is not present", change subscriptions to Event format (won't do any hard running command even if they already are in this format)

1.

for /F "tokens=*" %i in (subs.txt) DO wecutil ss "%i" /cf:Events

2.

Under Windows Regional Settings, on the Formats tab, change the format to "English (United States)"

# check subscriptions are being created on the source computer

Event Log: /Applications and Services Logs/Microsoft/Windows/Eventlog-ForwardingPlugin/Operational


#### troubleshooting WEF

collector server -> subscription name -> runtime status

gpupdate /force (force checkin, get subscriptions)

check Microsoft/Windows/Eventlog-ForwardingPlugin/Operational for errors

```

# SMB (139,445)
So null auth is enabled but cannot use it to enumerate shares or groups

Guest account is disabled

# Phishsin