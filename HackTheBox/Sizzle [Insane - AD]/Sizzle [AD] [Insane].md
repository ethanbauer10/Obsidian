# Host file setup
```python
sudo nxc smb 10.129.19.191 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.19.191   445    SIZZLE           [*] Windows 10 / Server 2016 Build 14393 x64 (name:SIZZLE) (domain:HTB.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
```
- SMB signing is enabled
- Null auth 

```python
nxc smb sizzle.htb.local -u 'amanda' -p 'Ashare1972'
```
# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.129.19.191                                                       
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-15 16:09 +0100
Nmap scan report for 10.129.19.191
Host is up (0.014s latency).
Not shown: 65507 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
53/tcp    open  domain
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
443/tcp   open  https
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
5986/tcp  open  wsmans
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49668/tcp open  unknown
49673/tcp open  unknown
49688/tcp open  unknown
49689/tcp open  unknown
49692/tcp open  unknown
49695/tcp open  unknown
49710/tcp open  unknown
49720/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.19 seconds
```

## Nmap
```python
nmap -p 21,53,80,135,139,389,443,445,464,593,636,3268,3269,5985,5986,9389 -A --min-rate=2000 -sT -Pn sizzle.htb.local
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-15 16:13 +0100
Nmap scan report for sizzle.htb.local (10.129.19.191)
Host is up (0.014s latency).
rDNS record for 10.129.19.191: SIZZLE.HTB.LOCAL

PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title (text/html).
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2021-02-11T12:59:51
|_Not valid after:  2022-02-11T12:59:51
|_ssl-date: 2026-04-15T15:15:29+00:00; 0s from scanner time.
443/tcp  open  ssl/https?
|_ssl-date: 2026-04-15T15:15:29+00:00; 0s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
| ssl-cert: Subject: commonName=sizzle.htb.local
| Not valid before: 2018-07-03T17:58:55
|_Not valid after:  2020-07-02T17:58:55
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2021-02-11T12:59:51
|_Not valid after:  2022-02-11T12:59:51
|_ssl-date: 2026-04-15T15:15:29+00:00; 0s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-15T15:15:29+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2021-02-11T12:59:51
|_Not valid after:  2022-02-11T12:59:51
3269/tcp open  ssl/ldap
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2021-02-11T12:59:51
|_Not valid after:  2022-02-11T12:59:51
|_ssl-date: 2026-04-15T15:15:29+00:00; 0s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
5986/tcp open  ssl/wsmans?
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2021-02-11T12:59:51
|_Not valid after:  2022-02-11T12:59:51
|_ssl-date: 2026-04-15T15:15:29+00:00; 0s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2012|2016|2008|7 (91%)
OS CPE: cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7
Aggressive OS guesses: Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Server 2016 (89%), Microsoft Windows 7 or Windows Server 2008 R2 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: SIZZLE; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Null auth enabled but not able to use it to enumerate shares or users

## Guest access
### Shares
```python
nxc smb sizzle.htb.local -u 'Guest' -p '' --shares
SMB         10.129.19.191   445    SIZZLE           [*] Windows 10 / Server 2016 Build 14393 x64 (name:SIZZLE) (domain:HTB.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.19.191   445    SIZZLE           [+] HTB.LOCAL\Guest: 
SMB         10.129.19.191   445    SIZZLE           [*] Enumerated shares
SMB         10.129.19.191   445    SIZZLE           Share           Permissions     Remark
SMB         10.129.19.191   445    SIZZLE           -----           -----------     ------
SMB         10.129.19.191   445    SIZZLE           ADMIN$                          Remote Admin
SMB         10.129.19.191   445    SIZZLE           C$                              Default share
SMB         10.129.19.191   445    SIZZLE           CertEnroll                      Active Directory Certificate Services share
SMB         10.129.19.191   445    SIZZLE           Department Shares READ            
SMB         10.129.19.191   445    SIZZLE           IPC$            READ            Remote IPC
SMB         10.129.19.191   445    SIZZLE           NETLOGON                        Logon server share 
SMB         10.129.19.191   445    SIZZLE           Operations                      
SMB         10.129.19.191   445    SIZZLE           SYSVOL                          Logon server share
```
Read access on `Department Shares` and `IPC$`
### Users
```python
nxc smb sizzle.htb.local -u 'Guest' -p '' --rid-brute 20000
SMB         10.129.19.191   445    SIZZLE           [*] Windows 10 / Server 2016 Build 14393 x64 (name:SIZZLE) (domain:HTB.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.19.191   445    SIZZLE           [+] HTB.LOCAL\Guest: 
SMB         10.129.19.191   445    SIZZLE           498: HTB\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           500: HTB\Administrator (SidTypeUser)
SMB         10.129.19.191   445    SIZZLE           501: HTB\Guest (SidTypeUser)
SMB         10.129.19.191   445    SIZZLE           502: HTB\krbtgt (SidTypeUser)
SMB         10.129.19.191   445    SIZZLE           503: HTB\DefaultAccount (SidTypeUser)
SMB         10.129.19.191   445    SIZZLE           512: HTB\Domain Admins (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           513: HTB\Domain Users (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           514: HTB\Domain Guests (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           515: HTB\Domain Computers (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           516: HTB\Domain Controllers (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           517: HTB\Cert Publishers (SidTypeAlias)
SMB         10.129.19.191   445    SIZZLE           518: HTB\Schema Admins (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           519: HTB\Enterprise Admins (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           520: HTB\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           521: HTB\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           522: HTB\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           525: HTB\Protected Users (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           526: HTB\Key Admins (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           527: HTB\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           553: HTB\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.19.191   445    SIZZLE           571: HTB\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.19.191   445    SIZZLE           572: HTB\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.19.191   445    SIZZLE           1001: HTB\SIZZLE$ (SidTypeUser)
SMB         10.129.19.191   445    SIZZLE           1102: HTB\DnsAdmins (SidTypeAlias)
SMB         10.129.19.191   445    SIZZLE           1103: HTB\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           1104: HTB\amanda (SidTypeUser)
SMB         10.129.19.191   445    SIZZLE           1603: HTB\mrlky (SidTypeUser)
SMB         10.129.19.191   445    SIZZLE           1604: HTB\sizzler (SidTypeUser)
```
Found all the users

```python
nxc smb sizzle.htb.local -u 'Guest' -p '' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 > users.txt
```
Ive re-ran this using some extra commands to pull the users out of the output

```python
cat users.txt   
Administrator
Guest
krbtgt
DefaultAccount
SIZZLE$
amanda
mrlky
sizzler
```
All the users

