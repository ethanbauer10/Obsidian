# Enumeration
## TCP ports
### Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.96.147
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-21 15:08 +0100
Nmap scan report for 10.129.96.147
Host is up (0.012s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
5985/tcp open  wsman
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 66.29 seconds
```

### Nmap
```python
nmap -p 80,5985,8080 -A --min-rate=2000 -sT 10.129.96.147      
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-21 15:10 +0100
Nmap scan report for 10.129.96.147
Host is up (0.013s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Mega Engines
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
8080/tcp open  http    Jetty 9.4.43.v20210629
|_http-server-header: Jetty(9.4.43.v20210629)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1903 - 22H2 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
Two webservers and also winrm

Cannot really do anything with winrm until i get credentials

## UDP ports
### Open ports
```python
nmap -p- -sU --min-rate=2000 -sT 10.129.96.147
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-21 15:10 +0100
Nmap scan report for 10.129.96.147
Host is up (0.013s latency).
Not shown: 65532 filtered tcp ports (no-response), 65531 open|filtered udp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
5985/tcp open  wsman
8080/tcp open  http-proxy
53/udp   open  domain
88/udp   open  kerberos-sec
123/udp  open  ntp
389/udp  open  ldap

Nmap done: 1 IP address (1 host up) scanned in 132.14 seconds
```
Here is where LDAP and kerberos are running

### Nmap
```python
nmap -p 53,88,123,389 -A --min-rate=2000 -sU 10.129.96.147                                                  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-21 15:15 +0100
Nmap scan report for 10.129.96.147
Host is up (0.081s latency).

PORT    STATE    SERVICE      VERSION
53/udp  open     domain       Simple DNS Plus
88/udp  open     kerberos-sec Microsoft Windows Kerberos (server time: 2026-04-21 14:15:28Z)
123/udp open     ntp          NTP v3
| ntp-info: 
|_  
389/udp open     ldap         Microsoft Windows Active Directory LDAP (Domain: object.local, Site: Default-First-Site-Name)
Too many fingerprints match this host to give specific OS details
Network Distance: 2 hops
Service Info: Host: JENKINS; OS: Windows; CPE: cpe:/o:microsoft:windows
```
Domain name is object.local

Ill add that to `/etc/hosts`

# HTTP (80)
## Nuclei
```python
nuclei -u http://object.local/                           

[iis-shortname-detect] [http] [info] http://object.local/*~1*/a.aspx
[iis-shortname-detect] [http] [info] http://object.local/zrb0nlkk*~1*/a.aspx
[iis-shortname-detect] [http] [info] http://object.local/*~1*/a.aspx
[waf-detect:modsecurity] [http] [info] http://object.local/
[microsoft-iis-version] [http] [info] http://object.local/ ["Microsoft-IIS/10.0"]
[missing-sri] [http] [info] http://object.local/ ["https://cpwebassets.codepen.io/assets/common/stopExecutionOnTimeout-1b93190375e9ccc259df3a57c1abc0e64599724ae30d7ea4c6877eb615f89387.js","https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.3/jquery.min.js","https://cdnjs.cloudflare.com/ajax/libs/gsap/1.15.0/TweenMax.min.js","https://s3-us-west-2.amazonaws.com/s.cdpn.io/28963/modernizr.custom.18922.js","https://cdnjs.cloudflare.com/ajax/libs/prefixfree/1.0.7/prefixfree.min.js","https://cdnjs.cloudflare.com/ajax/libs/normalize/5.0.0/normalize.min.css"]
[http-missing-security-headers:x-frame-options] [http] [info] http://object.local/
[http-missing-security-headers:x-content-type-options] [http] [info] http://object.local/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://object.local/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://object.local/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://object.local/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://object.local/
[http-missing-security-headers:permissions-policy] [http] [info] http://object.local/
[http-missing-security-headers:referrer-policy] [http] [info] http://object.local/
[http-missing-security-headers:clear-site-data] [http] [info] http://object.local/
[http-missing-security-headers:missing-content-type] [http] [info] http://object.local/
[http-missing-security-headers:strict-transport-security] [http] [info] http://object.local/
[http-missing-security-headers:content-security-policy] [http] [info] http://object.local/
[options-method] [http] [info] http://object.local/ ["OPTIONS, TRACE, GET, HEAD, POST"]
[tech-detect:ms-iis] [http] [info] http://object.local/
[caa-fingerprint] [dns] [info] object.local
```
## Feroxbuster
```python
Nothing found
```
## Subdomains
```python
Nothing found
```
## Website functionality
![985](Pasted%20image%2020260421152140.png)

Looks to be just a static site with a link to the other web service running on port 8080

![](Pasted%20image%2020260421152057.png)

Found an `ideas` user on here!

# HTTP (8080)

![](Pasted%20image%2020260421152533.png)

This is a Jenkins instance

![419](Pasted%20image%2020260421153223.png)


## Nuclei 
```python

```

## Subdomains
```python
Nothing found
```