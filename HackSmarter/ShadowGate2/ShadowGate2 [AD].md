# Objective and scope

ShadowGate provides cybersecurity solutions for global enterprises. They are in the process of getting SOC 2 certified, and have hired Hack Smarter to perform an internal network penetration test. Find all vulnerabilities and, if possible, elevate your privileges to Domain Admin.

You have been provided with VPN access to their network, but no other information.

# Host file setup

```python
sudo nxc smb 10.1.232.232 --generate-hosts-file /etc/hosts                                    
[sudo] password for kali: 
SMB         10.1.232.232    445    SG-DC01          [*] Windows 10 / Server 2019 Build 17763 x64 (name:SG-DC01) (domain:shadowgate.local) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration

## Open ports
```python
nmap -p- --min-rate=1000 -sT sg-dc01.shadowgate.local
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-31 17:09 +0100
Stats: 0:02:06 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 97.77% done; ETC: 17:12 (0:00:03 remaining)
Nmap scan report for sg-dc01.shadowgate.local (10.1.232.232)
Host is up (0.095s latency).
rDNS record for 10.1.232.232: SG-DC01.shadowgate.local
Not shown: 65521 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
593/tcp   open  http-rpc-epmap
1433/tcp  open  ms-sql-s
3268/tcp  open  globalcatLDAP
3389/tcp  open  ms-wbt-server
9389/tcp  open  adws
49665/tcp open  unknown
49669/tcp open  unknown
49694/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 129.76 seconds
```

Also port 389 and 5985 are open and nmap missed them!

## Nmap
```python
nmap -p 53,80,88,135,139,389,445,593,1433,3268,3389,5985,9389 -A --min-rate=1000 -sT sg-dc01.shadowgate.local
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-31 17:13 +0100
Nmap scan report for sg-dc01.shadowgate.local (10.1.232.232)
Host is up (0.095s latency).
rDNS record for 10.1.232.232: SG-DC01.shadowgate.local

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: ShadowGate | Advanced Cyber Security Solutions
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-31 16:14:04Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: shadowgate.local, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-31T16:14:57+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=SG-DC01.shadowgate.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:SG-DC01.shadowgate.local
| Not valid before: 2025-12-07T17:46:45
|_Not valid after:  2026-12-07T17:46:45
445/tcp  open  microsoft-ds?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   10.1.232.232:1433: 
|     Target_Name: SHADOWGATE
|     NetBIOS_Domain_Name: SHADOWGATE
|     NetBIOS_Computer_Name: SG-DC01
|     DNS_Domain_Name: shadowgate.local
|     DNS_Computer_Name: SG-DC01.shadowgate.local
|     DNS_Tree_Name: shadowgate.local
|_    Product_Version: 10.0.17763
| ms-sql-info: 
|   10.1.232.232:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-07-31T16:06:53
|_Not valid after:  2056-07-31T16:06:53
|_ssl-date: 2026-07-31T16:14:57+00:00; -1s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: shadowgate.local, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-31T16:14:57+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=SG-DC01.shadowgate.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:SG-DC01.shadowgate.local
| Not valid before: 2025-12-07T17:46:45
|_Not valid after:  2026-12-07T17:46:45
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: SHADOWGATE
|   NetBIOS_Domain_Name: SHADOWGATE
|   NetBIOS_Computer_Name: SG-DC01
|   DNS_Domain_Name: shadowgate.local
|   DNS_Computer_Name: SG-DC01.shadowgate.local
|   DNS_Tree_Name: shadowgate.local
|   Product_Version: 10.0.17763
|_  System_Time: 2026-07-31T16:14:18+00:00
| ssl-cert: Subject: commonName=SG-DC01.shadowgate.local
| Not valid before: 2026-07-18T09:38:28
|_Not valid after:  2027-01-17T09:38:28
|_ssl-date: 2026-07-31T16:14:57+00:00; -1s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (92%), Microsoft Windows 10 1903 - 22H2 (85%), Microsoft Windows 10 1607 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: Host: SG-DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Null auth is enabled like most DCs, but not able to use it to access shares or dump users

The guest account is also disabled!

# HTTP (80)

