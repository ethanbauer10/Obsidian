
# Machine info
Should you need to crack a hash, use a short custom wordlist based on company name and simple mutation rules commonly seen in real life passwords (e.g. year and a special character).

# Host file setup
```python
sudo nxc smb 10.129.234.63 --generate-hosts-file /etc/hosts
SMB         10.129.234.63   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:phantom.vl) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc.phantom.vl                                                                    
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-15 15:54 +0100
Nmap scan report for dc.phantom.vl (10.129.234.63)
Host is up (0.013s latency).
rDNS record for 10.129.234.63: DC.phantom.vl
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
9389/tcp  open  adws
49664/tcp open  unknown
49667/tcp open  unknown
49675/tcp open  unknown
49676/tcp open  unknown
49683/tcp open  unknown
56328/tcp open  unknown
56353/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.28 seconds
```

## Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,3389,5985,9389 -A --min-rate=2000 -sT dc.phantom.vl             
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-15 15:56 +0100
Nmap scan report for dc.phantom.vl (10.129.234.63)
Host is up (0.013s latency).
rDNS record for 10.129.234.63: DC.phantom.vl

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-15 14:57:00Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: phantom.vl, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: phantom.vl, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=DC.phantom.vl
| Not valid before: 2026-08-14T14:36:03
|_Not valid after:  2027-02-13T14:36:03
|_ssl-date: 2026-08-15T14:57:45+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: PHANTOM
|   NetBIOS_Domain_Name: PHANTOM
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: phantom.vl
|   DNS_Computer_Name: DC.phantom.vl
|   DNS_Tree_Name: phantom.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2026-08-15T14:57:05+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Null auth enabled, as with all DCs by default, cannot use it to enumerate shares or users

## Guest access
```python

```

```python

```