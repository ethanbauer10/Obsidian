# Starting credentials
```python
j.fleischman:J0elTHEM4n1990!
```

# Host file setup
```python
sudo nxc smb 10.129.232.88 --generate-hosts-file /etc/hosts                 
[sudo] password for kali: 
SMB         10.129.232.88   445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.232.88                                 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-24 18:05 +0000
Nmap scan report for 10.129.232.88
Host is up (0.014s latency).
Not shown: 65517 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
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
49667/tcp open  unknown
49689/tcp open  unknown
49690/tcp open  unknown
49697/tcp open  unknown
49707/tcp open  unknown
49720/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.31 seconds
```

## Nmap
```python
nmap -p 53,88,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT dc01.fluffy.htb
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-24 18:07 +0000
Nmap scan report for dc01.fluffy.htb (10.129.232.88)
Host is up (0.015s latency).
rDNS record for 10.129.232.88: DC01.fluffy.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-25 01:07:43Z)
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: fluffy.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.fluffy.htb
| Not valid before: 2025-04-17T16:04:17
|_Not valid after:  2026-04-17T16:04:17
|_ssl-date: 2026-03-25T01:09:10+00:00; +7h00m01s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fluffy.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-03-25T01:09:09+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.fluffy.htb
| Not valid before: 2025-04-17T16:04:17
|_Not valid after:  2026-04-17T16:04:17
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: fluffy.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-03-25T01:09:10+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.fluffy.htb
| Not valid before: 2025-04-17T16:04:17
|_Not valid after:  2026-04-17T16:04:17
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fluffy.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-03-25T01:09:09+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.fluffy.htb
| Not valid before: 2025-04-17T16:04:17
|_Not valid after:  2026-04-17T16:04:17
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```
DC is running at +7h

# SMB (445)
Null auth is enabled but cant access shares or list users

## Guest access
### Shares
```python
nxc smb dc01.fluffy.htb -u 'Guest' -p '' --shares 
SMB         10.129.232.88   445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.88   445    DC01             [+] fluffy.htb\Guest: 
SMB         10.129.232.88   445    DC01             [*] Enumerated shares
SMB         10.129.232.88   445    DC01             Share           Permissions     Remark
SMB         10.129.232.88   445    DC01             -----           -----------     ------
SMB         10.129.232.88   445    DC01             ADMIN$                          Remote Admin
SMB         10.129.232.88   445    DC01             C$                              Default share
SMB         10.129.232.88   445    DC01             IPC$            READ            Remote IPC
SMB         10.129.232.88   445    DC01             IT                              
SMB         10.129.232.88   445    DC01             NETLOGON                        Logon server share 
SMB         10.129.232.88   445    DC01             SYSVOL                          Logon server share
```
Limited access on shares

### Users
```python
nxc smb dc01.fluffy.htb -u 'Guest' -p '' --rid-brute             
SMB         10.129.232.88   445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.88   445    DC01             [+] fluffy.htb\Guest: 
SMB         10.129.232.88   445    DC01             498: FLUFFY\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.232.88   445    DC01             500: FLUFFY\Administrator (SidTypeUser)
SMB         10.129.232.88   445    DC01             501: FLUFFY\Guest (SidTypeUser)
SMB         10.129.232.88   445    DC01             502: FLUFFY\krbtgt (SidTypeUser)
SMB         10.129.232.88   445    DC01             512: FLUFFY\Domain Admins (SidTypeGroup)
SMB         10.129.232.88   445    DC01             513: FLUFFY\Domain Users (SidTypeGroup)
SMB         10.129.232.88   445    DC01             514: FLUFFY\Domain Guests (SidTypeGroup)
SMB         10.129.232.88   445    DC01             515: FLUFFY\Domain Computers (SidTypeGroup)
SMB         10.129.232.88   445    DC01             516: FLUFFY\Domain Controllers (SidTypeGroup)
SMB         10.129.232.88   445    DC01             517: FLUFFY\Cert Publishers (SidTypeAlias)
SMB         10.129.232.88   445    DC01             518: FLUFFY\Schema Admins (SidTypeGroup)
SMB         10.129.232.88   445    DC01             519: FLUFFY\Enterprise Admins (SidTypeGroup)
SMB         10.129.232.88   445    DC01             520: FLUFFY\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.232.88   445    DC01             521: FLUFFY\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.232.88   445    DC01             522: FLUFFY\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.232.88   445    DC01             525: FLUFFY\Protected Users (SidTypeGroup)
SMB         10.129.232.88   445    DC01             526: FLUFFY\Key Admins (SidTypeGroup)
SMB         10.129.232.88   445    DC01             527: FLUFFY\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.232.88   445    DC01             553: FLUFFY\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.232.88   445    DC01             571: FLUFFY\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.232.88   445    DC01             572: FLUFFY\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.232.88   445    DC01             1000: FLUFFY\DC01$ (SidTypeUser)
SMB         10.129.232.88   445    DC01             1101: FLUFFY\DnsAdmins (SidTypeAlias)
SMB         10.129.232.88   445    DC01             1102: FLUFFY\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.232.88   445    DC01             1103: FLUFFY\ca_svc (SidTypeUser)
SMB         10.129.232.88   445    DC01             1104: FLUFFY\ldap_svc (SidTypeUser)
SMB         10.129.232.88   445    DC01             1601: FLUFFY\p.agila (SidTypeUser)
SMB         10.129.232.88   445    DC01             1603: FLUFFY\winrm_svc (SidTypeUser)
SMB         10.129.232.88   445    DC01             1604: FLUFFY\Service Account Managers (SidTypeGroup)
SMB         10.129.232.88   445    DC01             1605: FLUFFY\j.coffey (SidTypeUser)
SMB         10.129.232.88   445    DC01             1606: FLUFFY\j.fleischman (SidTypeUser)
SMB         10.129.232.88   445    DC01             1607: FLUFFY\Service Accounts (SidTypeGroup)
```
Here is the list of users but since i have credentials i can use nxc `users-export` function to dump all the users to a file for me 

## Using provided creds
```python
j.fleischman:J0elTHEM4n1990!
```

### Shares
```python
nxc smb dc01.fluffy.htb -u 'j.fleischman' -p 'J0elTHEM4n1990!' --shares
SMB         10.129.232.88   445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.88   445    DC01             [+] fluffy.htb\j.fleischman:J0elTHEM4n1990! 
SMB         10.129.232.88   445    DC01             [*] Enumerated shares
SMB         10.129.232.88   445    DC01             Share           Permissions     Remark
SMB         10.129.232.88   445    DC01             -----           -----------     ------
SMB         10.129.232.88   445    DC01             ADMIN$                          Remote Admin
SMB         10.129.232.88   445    DC01             C$                              Default share
SMB         10.129.232.88   445    DC01             IPC$            READ            Remote IPC
SMB         10.129.232.88   445    DC01             IT              READ,WRITE      
SMB         10.129.232.88   445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.232.88   445    DC01             SYSVOL          READ            Logon server share
```
Write access on the `IT` share

### Users
```python
nxc smb dc01.fluffy.htb -u 'j.fleischman' -p 'J0elTHEM4n1990!' --users-export users.txt
SMB         10.129.232.88   445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.88   445    DC01             [+] fluffy.htb\j.fleischman:J0elTHEM4n1990! 
SMB         10.129.232.88   445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-
SMB         10.129.232.88   445    DC01             Administrator                 2025-04-17 15:45:01 0       Built-in account for administering the computer/domain
SMB         10.129.232.88   445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain
SMB         10.129.232.88   445    DC01             krbtgt                        2025-04-17 16:00:02 0       Key Distribution Center Service Account
SMB         10.129.232.88   445    DC01             ca_svc                        2025-04-17 16:07:50 0        
SMB         10.129.232.88   445    DC01             ldap_svc                      2025-04-17 16:17:00 0        
SMB         10.129.232.88   445    DC01             p.agila                       2025-04-18 14:37:08 0        
SMB         10.129.232.88   445    DC01             winrm_svc                     2025-05-18 00:51:16 0        
SMB         10.129.232.88   445    DC01             j.coffey                      2025-04-19 12:09:55 0        
SMB         10.129.232.88   445    DC01             j.fleischman                  2025-05-16 14:46:55 0        
SMB         10.129.232.88   445    DC01             [*] Enumerated 9 local users: FLUFFY
SMB         10.129.232.88   445    DC01             [*] Writing 9 local users to users.txt
```
All users dumped to a user file