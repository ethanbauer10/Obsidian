# Objective and scope

An adult beverage company "Martini Bars" recently had a corporate breach and the compliance and risk team dictates they perform a penetration test at one of their branch offices. The Hack Smarter team has been authorized to perform an internal black box pentest.

The client has provided you with VPN access to their internal network, but no credentials.

# Host file setup

```python
sudo nxc smb 10.1.33.56 --generate-hosts-file /etc/hosts
[sudo] password for kali:  
SMB         10.1.33.56      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
```

- SMB Signing disabled
- Windows 11 server 2025

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT DC01.DRY.MARTINI.BARS                                                   
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-15 17:10 +0100
Nmap scan report for DC01.DRY.MARTINI.BARS (10.1.33.56)
Host is up (0.097s latency).
Not shown: 65522 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
3268/tcp  open  globalcatLDAP
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
49664/tcp open  unknown
49667/tcp open  unknown
49672/tcp open  unknown
49679/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.88 seconds
```
Ldap is also open but for some reason Nmap missed it!

## Nmap
```python
nmap -p 53,88,135,139,389,445,464,3268,3269,3389,5985 -A --min-rate=2000 -sT dc01.dry.martini.bars   
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-15 17:13 +0100
Nmap scan report for dc01.dry.martini.bars (10.1.33.56)
Host is up (0.096s latency).
rDNS record for 10.1.33.56: DC01.DRY.MARTINI.BARS

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-05-15 16:13:44Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: DRY.MARTINI.BARS, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: DRY.MARTINI.BARS, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server
| rdp-ntlm-info: 
|   Target_Name: DRY
|   NetBIOS_Domain_Name: DRY
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: DRY.MARTINI.BARS
|   DNS_Computer_Name: DC01.DRY.MARTINI.BARS
|   Product_Version: 10.0.26100
|_  System_Time: 2026-05-15T16:13:58+00:00
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.DRY.MARTINI.BARS
| Not valid before: 2026-01-16T01:19:23
|_Not valid after:  2026-07-18T01:19:23
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.99%I=7%D=5/15%Time=6A07463E%P=x86_64-pc-linux-gnu%r(Te
SF:rminalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02\
SF:0\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```


# SMB (445)
Null auth is enabled but im not able to use it to enumerate users or shares

## Guest access
```python
nxc smb dc01.dry.martini.bars -u 'Guest' -p ''             
SMB         10.1.33.56      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
SMB         10.1.33.56      445    DC01             [+] DRY.MARTINI.BARS\Guest:
```
As seen here the guest account is enabled!

### Shares
```python
nxc smb dc01.dry.martini.bars -u 'Guest' -p '' --shares
SMB         10.1.33.56      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
SMB         10.1.33.56      445    DC01             [+] DRY.MARTINI.BARS\Guest: 
SMB         10.1.33.56      445    DC01             [*] Enumerated shares
SMB         10.1.33.56      445    DC01             Share           Permissions     Remark
SMB         10.1.33.56      445    DC01             -----           -----------     ------
SMB         10.1.33.56      445    DC01             ADMIN$                          Remote Admin
SMB         10.1.33.56      445    DC01             C$                              Default share
SMB         10.1.33.56      445    DC01             IPC$            READ            Remote IPC
SMB         10.1.33.56      445    DC01             NETLOGON                        Logon server share 
SMB         10.1.33.56      445    DC01             notes           READ,WRITE      
SMB         10.1.33.56      445    DC01             SYSVOL                          Logon server share
```
Read and write on the notes share, this is making me think of a watering hole attack!

### Users
```python
nxc smb dc01.dry.martini.bars -u 'Guest' -p '' --rid-brute 20000       
SMB         10.1.33.56      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
SMB         10.1.33.56      445    DC01             [+] DRY.MARTINI.BARS\Guest: 
SMB         10.1.33.56      445    DC01             498: DRY\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.1.33.56      445    DC01             500: DRY\Administrator (SidTypeUser)
SMB         10.1.33.56      445    DC01             501: DRY\Guest (SidTypeUser)
SMB         10.1.33.56      445    DC01             502: DRY\krbtgt (SidTypeUser)
SMB         10.1.33.56      445    DC01             512: DRY\Domain Admins (SidTypeGroup)
SMB         10.1.33.56      445    DC01             513: DRY\Domain Users (SidTypeGroup)
SMB         10.1.33.56      445    DC01             514: DRY\Domain Guests (SidTypeGroup)
SMB         10.1.33.56      445    DC01             515: DRY\Domain Computers (SidTypeGroup)
SMB         10.1.33.56      445    DC01             516: DRY\Domain Controllers (SidTypeGroup)
SMB         10.1.33.56      445    DC01             517: DRY\Cert Publishers (SidTypeAlias)
SMB         10.1.33.56      445    DC01             518: DRY\Schema Admins (SidTypeGroup)
SMB         10.1.33.56      445    DC01             519: DRY\Enterprise Admins (SidTypeGroup)
SMB         10.1.33.56      445    DC01             520: DRY\Group Policy Creator Owners (SidTypeGroup)
SMB         10.1.33.56      445    DC01             521: DRY\Read-only Domain Controllers (SidTypeGroup)
SMB         10.1.33.56      445    DC01             522: DRY\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.1.33.56      445    DC01             525: DRY\Protected Users (SidTypeGroup)
SMB         10.1.33.56      445    DC01             526: DRY\Key Admins (SidTypeGroup)
SMB         10.1.33.56      445    DC01             527: DRY\Enterprise Key Admins (SidTypeGroup)
SMB         10.1.33.56      445    DC01             528: DRY\Forest Trust Accounts (SidTypeGroup)
SMB         10.1.33.56      445    DC01             529: DRY\External Trust Accounts (SidTypeGroup)
SMB         10.1.33.56      445    DC01             553: DRY\RAS and IAS Servers (SidTypeAlias)
SMB         10.1.33.56      445    DC01             571: DRY\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.1.33.56      445    DC01             572: DRY\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.1.33.56      445    DC01             1000: DRY\DC01$ (SidTypeUser)
SMB         10.1.33.56      445    DC01             1101: DRY\DnsAdmins (SidTypeAlias)
SMB         10.1.33.56      445    DC01             1102: DRY\DnsUpdateProxy (SidTypeGroup)
SMB         10.1.33.56      445    DC01             1104: DRY\mprice (SidTypeUser)
SMB         10.1.33.56      445    DC01             1105: DRY\athena.t0 (SidTypeUser)
SMB         10.1.33.56      445    DC01             1106: DRY\ATHENA_SVC (SidTypeUser)
```
Ive managed to get the users, but now ill adapt this command slightly to export them to a file

```python
nxc smb dc01.dry.martini.bars -u 'Guest' -p '' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 > users.txt
```
This command manipulated the output and placed the output into a users.txt file

```python
cat users.txt       
Administrator
Guest
krbtgt
DC01$
mprice
athena.t0
ATHENA_SVC
```
Now i have some users it might be worth trying some AS-REP roasting

AS-REP roasting failed!

# `notes` share

I have read and write access on this share

```python
smbclient //dc01.dry.martini.bars/notes -U 'Guest'%''                            
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri May 15 17:18:53 2026
  ..                                DHS        0  Sat Jan 17 16:38:33 2026
  notes.txt                           A      129  Sat Jan 17 16:38:47 2026

		7731967 blocks of size 4096. 1522701 blocks available
smb: \> get notes.txt 
getting file \notes.txt of size 129 as notes.txt (0.3 KiloBytes/sec) (average 0.3 KiloBytes/sec)
smb: \> 
```

```python
cat notes.txt 
- Order more gin for lakeside
- Look for an engagement ring
- Check that notes works from Linux Mint

creds
mprice:*martini*
```
Looks like ive found some credentials

```python
nxc ldap dc01.dry.martini.bars -u mprice -p '*martini*'                                   
LDAP        10.1.33.56      389    DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:Enforced) (channel binding:No TLS cert) 
LDAP        10.1.33.56      389    DC01             [+] DRY.MARTINI.BARS\mprice:*martini*
```
The credentials are valid, but i also think it might be worth trying a watering hole attack


