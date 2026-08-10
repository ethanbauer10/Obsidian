# Host file setup

```python
sudo nxc smb 10.129.244.81 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.244.81   445    S200401          [*] Windows Server 2022 Build 20348 x64 (name:S200401) (domain:overwatch.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT s200401.overwatch.htb                 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-10 15:51 +0100
Nmap scan report for s200401.overwatch.htb (10.129.244.81)
Host is up (0.013s latency).
rDNS record for 10.129.244.81: S200401.overwatch.htb
Not shown: 65514 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
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
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
6520/tcp  open  unknown
9389/tcp  open  adws
49664/tcp open  unknown
49669/tcp open  unknown
59048/tcp open  unknown
59049/tcp open  unknown
59056/tcp open  unknown
59143/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.26 seconds
```

## Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,3389,5985,6520,9389 -A --min-rate=2000 -sT s200401.overwatch.htb                                                                  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-10 15:53 +0100
Nmap scan report for s200401.overwatch.htb (10.129.244.81)
Host is up (0.013s latency).
rDNS record for 10.129.244.81: S200401.overwatch.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  tcpwrapped
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-10 14:53:51Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: overwatch.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: overwatch.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: OVERWATCH
|   NetBIOS_Domain_Name: OVERWATCH
|   NetBIOS_Computer_Name: S200401
|   DNS_Domain_Name: overwatch.htb
|   DNS_Computer_Name: S200401.overwatch.htb
|   DNS_Tree_Name: overwatch.htb
|   Product_Version: 10.0.20348
|_  System_Time: 2026-08-10T14:54:29+00:00
| ssl-cert: Subject: commonName=S200401.overwatch.htb
| Not valid before: 2026-05-12T14:36:16
|_Not valid after:  2026-11-11T14:36:16
|_ssl-date: 2026-08-10T14:55:09+00:00; -1s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
6520/tcp open  ms-sql-s      Microsoft SQL Server 2022 16.00.1000.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-08-10T14:47:15
|_Not valid after:  2056-08-10T14:47:15
| ms-sql-info: 
|   10.129.244.81:6520: 
|     Version: 
|       name: Microsoft SQL Server 2022 RTM
|       number: 16.00.1000.00
|       Product: Microsoft SQL Server 2022
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 6520
| ms-sql-ntlm-info: 
|   10.129.244.81:6520: 
|     Target_Name: OVERWATCH
|     NetBIOS_Domain_Name: OVERWATCH
|     NetBIOS_Computer_Name: S200401
|     DNS_Domain_Name: overwatch.htb
|     DNS_Computer_Name: S200401.overwatch.htb
|     DNS_Tree_Name: overwatch.htb
|_    Product_Version: 10.0.20348
|_ssl-date: 2026-08-10T14:55:09+00:00; -1s from scanner time.
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: S200401; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)

Null auth is enab