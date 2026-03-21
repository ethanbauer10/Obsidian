# Hostname setup
```python
sudo nxc smb 10.129.234.42 --generate-hosts-file /etc/hosts      
[sudo] password for kali: 
SMB         10.129.234.42   445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:None) (Null Auth:True)
```
This has created me an entry in `/etc/hosts`

# Enuumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.234.42
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-21 17:13 +0000
Nmap scan report for 10.129.234.42
Host is up (0.014s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
50168/tcp open  unknown
50182/tcp open  unknown
50200/tcp open  unknown
50888/tcp open  unknown
51970/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.34 seconds
```
Whats interesting here is that there is SSH (22) open
## Nmap
```python
nmap -p 22,53,88,135,139,445,464,593,3268,3269,3389,9389 -A --min-rate=2000 -sT AWSJPDC0522.shibuya.vl
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-21 17:17 +0000
Nmap scan report for AWSJPDC0522.shibuya.vl (10.129.234.42)
Host is up (0.015s latency).

PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_9.5 (protocol 2.0)
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-21 17:17:12Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: shibuya.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=AWSJPDC0522.shibuya.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:AWSJPDC0522.shibuya.vl
| Not valid before: 2026-03-21T17:03:50
|_Not valid after:  2027-03-21T17:03:50
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: shibuya.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=AWSJPDC0522.shibuya.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:AWSJPDC0522.shibuya.vl
| Not valid before: 2026-03-21T17:03:50
|_Not valid after:  2027-03-21T17:03:50
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: SHIBUYA
|   NetBIOS_Domain_Name: SHIBUYA
|   NetBIOS_Computer_Name: AWSJPDC0522
|   DNS_Domain_Name: shibuya.vl
|   DNS_Computer_Name: AWSJPDC0522.shibuya.vl
|   DNS_Tree_Name: shibuya.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2026-03-21T17:17:55+00:00
|_ssl-date: 2026-03-21T17:18:35+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=AWSJPDC0522.shibuya.vl
| Not valid before: 2026-03-20T17:12:59
|_Not valid after:  2026-09-19T17:12:59
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: AWSJPDC0522; OS: Windows; CPE: cpe:/o:microsoft:windows
```
Not a lot i can do with SSH until i get credentials

# SMB (445)
Null auth is enabled be default on domain controllers
Guest account is disabled


