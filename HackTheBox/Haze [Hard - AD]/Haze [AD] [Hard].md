
# Host file setup
```python
sudo nxc smb 10.129.11.223 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.11.223   445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:haze.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.haze.htb                                                                                                                                   
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-07 16:31 +0100
Nmap scan report for dc01.haze.htb (10.129.11.223)
Host is up (0.013s latency).
rDNS record for 10.129.11.223: DC01.haze.htb
Not shown: 65505 closed tcp ports (conn-refused)
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
5985/tcp  open  wsman
8000/tcp  open  http-alt
8088/tcp  open  radan-http
8089/tcp  open  unknown
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49674/tcp open  unknown
49683/tcp open  unknown
49684/tcp open  unknown
64858/tcp open  unknown
64865/tcp open  unknown
64868/tcp open  unknown
64883/tcp open  unknown
64918/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 11.22 seconds
```

## Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,8000,8088,8089,9389 -A --min-rate=2000 -sT dc01.haze.htb                                                            
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-07 16:33 +0100
Nmap scan report for dc01.haze.htb (10.129.11.223)
Host is up (0.013s latency).
rDNS record for 10.129.11.223: DC01.haze.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-06-07 23:33:45Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: haze.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc01.haze.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.haze.htb
| Not valid before: 2026-06-07T23:20:32
|_Not valid after:  2027-06-07T23:20:32
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: haze.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc01.haze.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.haze.htb
| Not valid before: 2026-06-07T23:20:32
|_Not valid after:  2027-06-07T23:20:32
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: haze.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc01.haze.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.haze.htb
| Not valid before: 2026-06-07T23:20:32
|_Not valid after:  2027-06-07T23:20:32
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: haze.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc01.haze.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.haze.htb
| Not valid before: 2026-06-07T23:20:32
|_Not valid after:  2027-06-07T23:20:32
|_ssl-date: TLS randomness does not represent time
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8000/tcp open  http          Splunkd httpd
|_http-server-header: Splunkd
| http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_Requested resource was http://dc01.haze.htb:8000/en-US/account/login?return_to=%2Fen-US%2F
| http-robots.txt: 1 disallowed entry 
|_/
8088/tcp open  ssl/http      Splunkd httpd
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser
| Not valid before: 2025-03-05T07:29:08
|_Not valid after:  2028-03-04T07:29:08
| http-robots.txt: 1 disallowed entry 
|_/
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Splunkd
|_http-title: 404 Not Found
8089/tcp open  ssl/http      Splunkd httpd
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser
| Not valid before: 2025-03-05T07:29:08
|_Not valid after:  2028-03-04T07:29:08
|_http-title: splunkd
|_http-server-header: Splunkd
|_ssl-date: TLS randomness does not represent time
| http-robots.txt: 1 disallowed entry 
|_/
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (96%), Microsoft Windows Server 2016 (96%), Microsoft Windows 11 24H2 (96%), Microsoft Windows Server 2022 (95%), Microsoft Windows 11 24H2 - 25H2 (95%), Microsoft Windows Server 2012 R2 (93%), Microsoft Windows Server 2019 (92%), Microsoft Windows Server 2016 or Server 2019 (91%), Microsoft Windows Server 2012 (91%), Microsoft Windows 10 1703 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Null auth is enabled but i cannot use it to list shares or dump users

Guest account is disabled

# HTTP (8000)
## Nuclei
```python
nuclei -u http://haze.htb:8000/       

[CVE-2024-36991] [http] [high] http://haze.htb:8000/en-US/modules/messaging/C:../C:../C:../C:../C:../C:../C:../C:../C:../C:../C:../Windows/win.ini
[INF] Skipped haze.htb:9780 from target list as found unresponsive permanently: cause="port closed or filtered" address=haze.htb:9780 chain="connection refused; got err while executing http://haze.htb:9780/api/v1/user_assets/nfc"
[ldap-metadata] [javascript] [info] haze.htb:8000 ["DomainControllerFunctionality: 7","BaseDN: dc=8000","DnsHostName: dc01.haze.htb","DefaultNamingContext: DC=haze,DC=htb","DomainFunctionality: 7","ForestFunctionality: 7"]
[splunk-enterprise-panel] [http] [info] http://haze.htb:8000/en-US/account/login
[http-missing-security-headers:content-security-policy] [http] [info] http://haze.htb:8000/en-GB/account/login?return_to=%2Fen-GB%2F
[http-missing-security-headers:permissions-policy] [http] [info] http://haze.htb:8000/en-GB/account/login?return_to=%2Fen-GB%2F
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://haze.htb:8000/en-GB/account/login?return_to=%2Fen-GB%2F
[http-missing-security-headers:referrer-policy] [http] [info] http://haze.htb:8000/en-GB/account/login?return_to=%2Fen-GB%2F
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://haze.htb:8000/en-GB/account/login?return_to=%2Fen-GB%2F
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://haze.htb:8000/en-GB/account/login?return_to=%2Fen-GB%2F
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://haze.htb:8000/en-GB/account/login?return_to=%2Fen-GB%2F
[http-missing-security-headers:strict-transport-security] [http] [info] http://haze.htb:8000/en-GB/account/login?return_to=%2Fen-GB%2F
[http-missing-security-headers:referrer-policy] [http] [info] http://haze.htb:8000/en-GB/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://haze.htb:8000/en-GB/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://haze.htb:8000/en-GB/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://haze.htb:8000/en-GB/
[http-missing-security-headers:strict-transport-security] [http] [info] http://haze.htb:8000/en-GB/
[http-missing-security-headers:content-security-policy] [http] [info] http://haze.htb:8000/en-GB/
[http-missing-security-headers:permissions-policy] [http] [info] http://haze.htb:8000/en-GB/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://haze.htb:8000/en-GB/
[http-missing-security-headers:permissions-policy] [http] [info] http://haze.htb:8000/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://haze.htb:8000/
[http-missing-security-headers:referrer-policy] [http] [info] http://haze.htb:8000/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://haze.htb:8000/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://haze.htb:8000/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://haze.htb:8000/
[http-missing-security-headers:strict-transport-security] [http] [info] http://haze.htb:8000/
[http-missing-security-headers:content-security-policy] [http] [info] http://haze.htb:8000/
[robots-txt-endpoint] [http] [info] http://haze.htb:8000/robots.txt
[robots-txt] [http] [info] http://haze.htb:8000/robots.txt
[missing-cookie-samesite-strict] [http] [info] http://haze.htb:8000/en-GB/account/login?return_to=%2Fen-GB%2F ["cval=1064979552; Path=/en-GB/account/ splunkweb_uid=D49EBB76-3DE2-417C-9AA2-38EE230638CB; Path=/en-GB/account; Max-Age=157680000; Expires=Fri, 06 Jun 2031 23:44:30 GMT"]
[fingerprinthub-web-fingerprints:splunk] [http] [info] http://haze.htb:8000/en-GB/account/login?return_to=%2Fen-GB%2F
[fingerprinthub-web-fingerprints:splunkd] [http] [info] http://haze.htb:8000/en-GB/account/login?return_to=%2Fen-GB%2F
[missing-cookie-samesite-strict] [http] [info] http://haze.htb:8000/en-GB/ ["session_id_8000=291dda4b4ddb6c2dc26ea8c71ecd2bdd833b8633; expires=Mon, 08 Jun 2026 00:44:30 GMT; HttpOnly; Max-Age=3600; Path=/"]
[fingerprinthub-web-fingerprints:splunk] [http] [info] http://haze.htb:8000/en-GB/
[fingerprinthub-web-fingerprints:splunkd] [http] [info] http://haze.htb:8000/en-GB/
[fingerprinthub-web-fingerprints:splunk] [http] [info] http://haze.htb:8000/
[fingerprinthub-web-fingerprints:splunkd] [http] [info] http://haze.htb:8000/
[caa-fingerprint] [dns] [info] haze.htb
```
CVE-2024-36991?
