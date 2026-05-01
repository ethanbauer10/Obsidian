
# Objective and scope
NovaCart is an e-commerce company that operates a webshop for PC accessories. However, the entire platform is still under active development and expansion.

The IT team is currently working on a Linux-based version of the webshop as well as the development of a mobile app for NovaCart. The team is focused on adding features quickly, and has not prioritized security.

We have been tasked with conducting a penetration test to thoroughly assess the environment, identify vulnerabilities, and evaluate the overall security of the system.

The client has provided you with VPN access to their internal network, but no credentials.

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.0.18.23   
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-01 17:12 +0100
Nmap scan report for 10.0.18.23
Host is up (0.096s latency).
Not shown: 65506 closed tcp ports (conn-refused)
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
5000/tcp  open  upnp
5985/tcp  open  wsman
8080/tcp  open  http-proxy
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49668/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
49672/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49704/tcp open  unknown
49711/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 34.83 seconds
```

## Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,3389,5000,5985,8080,9389 -A --min-rate=2000 -sT 10.0.18.23
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-01 17:16 +0100
Nmap scan report for 10.0.18.23
Host is up (0.096s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: NovaCart | Premium PC Components & Gaming Systems
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-05-01 16:16:18Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: novacart.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: novacart.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=DC.novacart.local
| Not valid before: 2025-12-24T22:48:49
|_Not valid after:  2026-06-25T22:48:49
| rdp-ntlm-info: 
|   Target_Name: NOVACART
|   NetBIOS_Domain_Name: NOVACART
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: novacart.local
|   DNS_Computer_Name: DC.novacart.local
|   DNS_Tree_Name: novacart.local
|   Product_Version: 10.0.17763
|_  System_Time: 2026-05-01T16:16:36+00:00
|_ssl-date: 2026-05-01T16:16:45+00:00; 0s from scanner time.
5000/tcp open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  NTLM
|_http-title: 401 - Unauthorized: Access is denied due to invalid credentials.
| http-ntlm-info: 
|   Target_Name: NOVACART
|   NetBIOS_Domain_Name: NOVACART
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: novacart.local
|   DNS_Computer_Name: DC.novacart.local
|   DNS_Tree_Name: novacart.local
|_  Product_Version: 10.0.17763
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8080/tcp open  http          Jetty 12.0.25
|_http-server-header: Jetty(12.0.25)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10|11|2012|2022|2016 (93%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2019 (93%), Microsoft Windows 10 1709 - 22H2 (92%), Microsoft Windows 10 1909 (90%), Microsoft Windows 10 1909 - 2004 (90%), Microsoft Windows 11 24H2 - 25H2 (89%), Microsoft Windows Server 2012 R2 (89%), Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2016 (88%), Microsoft Windows 10 21H2 (87%), Microsoft Windows Server 2012 Data Center (87%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Three webservers running
- port 80
- port 5000
- port 8080

Winrm and RDP are also open

Nothing interesting in DNS

# HTTP (80)
## Nuclei
```python
nuclei -u http://novacart.local/

[iis-shortname-detect] [http] [info] http://novacart.local/*~1*/a.aspx
[iis-shortname-detect] [http] [info] http://novacart.local/tqqjrdw5*~1*/a.aspx
[iis-shortname-detect] [http] [info] http://novacart.local/*~1*/a.aspx
[waf-detect:aspgeneric] [http] [info] http://novacart.local/
[waf-detect:modsecurity] [http] [info] http://novacart.local/
[rdp-detect] [javascript] [info] novacart.local:3389
[ldap-metadata] [javascript] [info] novacart.local:389 ["DomainFunctionality: 7","ForestFunctionality: 7","DomainControllerFunctionality: 7","BaseDN: dc=389","DnsHostName: DC.novacart.local","DefaultNamingContext: DC=novacart,DC=local"]
[mysql-info] [javascript] [info] novacart.local:3306 ["Version:","Transport: tcp"]
[smb-version-detect:smb-version] [javascript] [info] novacart.local:445 ["SMB 2.1"]
[smb2-server-time] [javascript] [info] novacart.local:445 ["SystemTime: 2026-05-01T16:25:56.000Z ServerStartTime: 2009-04-22T19:24:48.000Z"]
[smb2-capabilities] [javascript] [info] novacart.local:445 ["["DFSSupport","LargeMTU","Leasing"]"]
[ldap-anonymous-login-detect] [javascript] [medium] novacart.local:389
[smb-enum-domains] [javascript] [info] novacart.local:445 ["DomainName: novacart.local"]
[smb-enum] [javascript] [info] novacart.local:445 ["NetBIOSComputerName: DC","NetBIOSDomainName: NOVACART","DNSComputerNamen: DC.novacart.local","DNSComputerName: DC.novacart.local","ForestName: novacart.local","OSVersion: 10.0.17763"]
[smb-os-detect] [javascript] [info] novacart.local:445 ["Windows Server 2019, Version 1809"]
[mysql-detect] [tcp] [info] novacart.local:3306
[rdp-detection:win2016] [tcp] [info] novacart.local:3389
[microsoft-iis-version] [http] [info] http://novacart.local/ ["Microsoft-IIS/10.0"]
[aspnet-version-detect] [http] [info] http://novacart.local/%3f ["4.0.30319"]
[missing-sri] [http] [info] http://novacart.local/ ["https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css","https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&display=swap"]
[addeventlistener-detect] [http] [info] http://novacart.local/
[email-extractor] [http] [info] http://novacart.local/ ["support@novacart.com"]
[old-copyright] [http] [info] http://novacart.local/ ["&copy; 2023"]
[http-missing-security-headers:strict-transport-security] [http] [info] http://novacart.local/
[http-missing-security-headers:content-security-policy] [http] [info] http://novacart.local/
[http-missing-security-headers:permissions-policy] [http] [info] http://novacart.local/
[http-missing-security-headers:x-content-type-options] [http] [info] http://novacart.local/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://novacart.local/
[http-missing-security-headers:referrer-policy] [http] [info] http://novacart.local/
[http-missing-security-headers:clear-site-data] [http] [info] http://novacart.local/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://novacart.local/
[http-missing-security-headers:x-frame-options] [http] [info] http://novacart.local/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://novacart.local/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://novacart.local/
[http-missing-security-headers:missing-content-type] [http] [info] http://novacart.local/
[options-method] [http] [info] http://novacart.local/ ["OPTIONS, TRACE, GET, HEAD, POST"]
[tech-detect:font-awesome] [http] [info] http://novacart.local/
[tech-detect:google-font-api] [http] [info] http://novacart.local/
[tech-detect:ms-iis] [http] [info] http://novacart.local/
[caa-fingerprint] [dns] [info] novacart.local
```

It looks like its identified mysql as being open!

```python
nmap -p 3306 -A dc.novacart.local                 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-01 17:28 +0100
Nmap scan report for dc.novacart.local (10.0.18.23)
Host is up (0.096s latency).
rDNS record for 10.0.18.23: DC.novacart.local

PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL (unauthorized)
```
Nmap is telling me it is as well

There are no subdomains!

After running feroxbuster i did not find anything interesting

![1031](Pasted%20image%2020260501173754.png)

Whats interesting is there is a search function that is pulling from a file, could be vulnerable to some 