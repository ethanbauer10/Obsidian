# Host file setup
```python
sudo nxc smb 10.129.229.56 --generate-hosts-file /etc/hosts                                             
[sudo] password for kali: 
SMB         10.129.229.56   445    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 x64 (name:AUTHORITY) (domain:authority.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT authority.authority.htb
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-23 20:13 +0000
Nmap scan report for authority.authority.htb (10.129.229.56)
Host is up (0.014s latency).
rDNS record for 10.129.229.56: AUTHORITY.authority.htb
Not shown: 65507 closed tcp ports (conn-refused)
PORT      STATE SERVICE
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
5985/tcp  open  wsman
8443/tcp  open  https-alt
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49671/tcp open  unknown
49674/tcp open  unknown
49675/tcp open  unknown
49679/tcp open  unknown
49682/tcp open  unknown
49691/tcp open  unknown
49702/tcp open  unknown
61399/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 19.91 seconds
```

## Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,8443,9389 -A --min-rate=2000 -sT authority.authority.htb
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-23 20:14 +0000
Nmap scan report for authority.authority.htb (10.129.229.56)
Host is up (0.015s latency).
rDNS record for 10.129.229.56: AUTHORITY.authority.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-24 00:15:00Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-03-24T00:15:53+00:00; +4h00m00s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
|_ssl-date: 2026-03-24T00:15:53+00:00; +4h00m00s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
|_ssl-date: 2026-03-24T00:15:53+00:00; +4h00m00s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-03-24T00:15:53+00:00; +4h00m00s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8443/tcp open  ssl/http      Apache Tomcat (language: en)
|_http-title: Site doesn't have a title (text/html;charset=ISO-8859-1).
| tls-alpn: 
|_  h2
| ssl-cert: Subject: commonName=172.16.2.118
| Not valid before: 2026-03-22T00:10:22
|_Not valid after:  2028-03-23T11:48:46
|_ssl-date: TLS randomness does not represent time
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 10|2019|2022|2012|2016 (96%)
OS CPE: cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows 10 1909 - 2004 (96%), Microsoft Windows Server 2019 (95%), Windows Server 2019 (92%), Microsoft Windows 10 1909 (92%), Microsoft Windows Server 2022 (92%), Microsoft Windows 10 1709 - 21H2 (91%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows 10 20H2 (90%), Microsoft Windows 10 20H2 - 21H1 (90%), Microsoft Windows Server 2016 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: AUTHORITY; OS: Windows; CPE: cpe:/o:microsoft:windows
```
DC is running at +4h

# HTTP (80)
The landing page is a default IIS server

No subdomains

No vulnerabilities associated with the webserver

No hidden endpoints

# SMB (445)
Null auth enabled but not able to access shares or list users using it

## Guest access
```python
nxc smb authority.authority.htb -u 'Guest' -p ''       
SMB         10.129.229.56   445    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 x64 (name:AUTHORITY) (domain:authority.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.229.56   445    AUTHORITY        [+] authority.htb\Guest: 
```
Guest account is enabled i can use this to enumerate
### Shares
```python
nxc smb authority.authority.htb -u 'Guest' -p '' --shares
SMB         10.129.229.56   445    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 x64 (name:AUTHORITY) (domain:authority.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.229.56   445    AUTHORITY        [+] authority.htb\Guest: 
SMB         10.129.229.56   445    AUTHORITY        [*] Enumerated shares
SMB         10.129.229.56   445    AUTHORITY        Share           Permissions     Remark
SMB         10.129.229.56   445    AUTHORITY        -----           -----------     ------
SMB         10.129.229.56   445    AUTHORITY        ADMIN$                          Remote Admin
SMB         10.129.229.56   445    AUTHORITY        C$                              Default share
SMB         10.129.229.56   445    AUTHORITY        Department Shares                 
SMB         10.129.229.56   445    AUTHORITY        Development     READ            
SMB         10.129.229.56   445    AUTHORITY        IPC$            READ            Remote IPC
SMB         10.129.229.56   445    AUTHORITY        NETLOGON                        Logon server share 
SMB         10.129.229.56   445    AUTHORITY        SYSVOL                          Logon server share
```
Read access on `Development`
### Users
```python
nxc smb authority.authority.htb -u 'Guest' -p '' --rid-brute             
SMB         10.129.229.56   445    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 x64 (name:AUTHORITY) (domain:authority.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.229.56   445    AUTHORITY        [+] authority.htb\Guest: 
SMB         10.129.229.56   445    AUTHORITY        498: HTB\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.229.56   445    AUTHORITY        500: HTB\Administrator (SidTypeUser)
SMB         10.129.229.56   445    AUTHORITY        501: HTB\Guest (SidTypeUser)
SMB         10.129.229.56   445    AUTHORITY        502: HTB\krbtgt (SidTypeUser)
SMB         10.129.229.56   445    AUTHORITY        512: HTB\Domain Admins (SidTypeGroup)
SMB         10.129.229.56   445    AUTHORITY        513: HTB\Domain Users (SidTypeGroup)
SMB         10.129.229.56   445    AUTHORITY        514: HTB\Domain Guests (SidTypeGroup)
SMB         10.129.229.56   445    AUTHORITY        515: HTB\Domain Computers (SidTypeGroup)
SMB         10.129.229.56   445    AUTHORITY        516: HTB\Domain Controllers (SidTypeGroup)
SMB         10.129.229.56   445    AUTHORITY        517: HTB\Cert Publishers (SidTypeAlias)
SMB         10.129.229.56   445    AUTHORITY        518: HTB\Schema Admins (SidTypeGroup)
SMB         10.129.229.56   445    AUTHORITY        519: HTB\Enterprise Admins (SidTypeGroup)
SMB         10.129.229.56   445    AUTHORITY        520: HTB\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.229.56   445    AUTHORITY        521: HTB\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.229.56   445    AUTHORITY        522: HTB\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.229.56   445    AUTHORITY        525: HTB\Protected Users (SidTypeGroup)
SMB         10.129.229.56   445    AUTHORITY        526: HTB\Key Admins (SidTypeGroup)
SMB         10.129.229.56   445    AUTHORITY        527: HTB\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.229.56   445    AUTHORITY        553: HTB\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.229.56   445    AUTHORITY        571: HTB\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.229.56   445    AUTHORITY        572: HTB\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.229.56   445    AUTHORITY        1000: HTB\AUTHORITY$ (SidTypeUser)
SMB         10.129.229.56   445    AUTHORITY        1101: HTB\DnsAdmins (SidTypeAlias)
SMB         10.129.229.56   445    AUTHORITY        1102: HTB\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.229.56   445    AUTHORITY        1601: HTB\svc_ldap (SidTypeUser)
```
Ill put the users in a user file

```python
Administrator
Guest
krbtgt
AUTHORITY$
svc_ldap
```
Not a lot of users in this domain

### `Development` share
```python

```
