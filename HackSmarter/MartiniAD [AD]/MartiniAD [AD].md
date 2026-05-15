# Objective and scope

An adult beverage company "Martini Bars" recently had a corporate breach and the compliance and risk team dictates they perform a penetration test at one of their branch offices. The Hack Smarter team has been authorized to perform an internal black box pentest.

The client has provided you with VPN access to their internal network, but no credentials.

# Host file setup

```python
sudo nxc smb 10.1.33.56 --generate-hosts-file /etc/hosts
[sudo] password for kali:  
SMB         10.1.33.56      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
```

- SMB Signing disabled
- Windows 11 server 2025

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT DC01.DRY.MARTINI.BARS                                                   
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-15 17:10 +0100
Nmap scan report for DC01.DRY.MARTINI.BARS (10.1.33.56)
Host is up (0.097s latency).
Not shown: 65522 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
3268/tcp  open  globalcatLDAP
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
49664/tcp open  unknown
49667/tcp open  unknown
49672/tcp open  unknown
49679/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.88 seconds
```
Ldap is also open but for some reason Nmap missed it!

## Nmap
```python
nmap -p 53,88,135,139,389,445,464,3268,3269,3389,5985 -A --min-rate=2000 -sT dc01.dry.martini.bars   
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-15 17:13 +0100
Nmap scan report for dc01.dry.martini.bars (10.1.33.56)
Host is up (0.096s latency).
rDNS record for 10.1.33.56: DC01.DRY.MARTINI.BARS

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-05-15 16:13:44Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: DRY.MARTINI.BARS, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: DRY.MARTINI.BARS, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server
| rdp-ntlm-info: 
|   Target_Name: DRY
|   NetBIOS_Domain_Name: DRY
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: DRY.MARTINI.BARS
|   DNS_Computer_Name: DC01.DRY.MARTINI.BARS
|   Product_Version: 10.0.26100
|_  System_Time: 2026-05-15T16:13:58+00:00
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.DRY.MARTINI.BARS
| Not valid before: 2026-01-16T01:19:23
|_Not valid after:  2026-07-18T01:19:23
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.99%I=7%D=5/15%Time=6A07463E%P=x86_64-pc-linux-gnu%r(Te
SF:rminalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02\
SF:0\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```


# SMB (445)
Null auth is enabled but im 