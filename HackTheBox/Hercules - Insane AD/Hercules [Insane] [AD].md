# Host file setup
```python
sudo nxc smb 10.129.242.196 --generate-hosts-file /etc/hosts                                         
[sudo] password for kali: 
SMB         10.129.242.196  445    dc               [*]  x64 (name:dc) (domain:hercules.htb) (signing:True) (SMBv1:None) (NTLM:False)
```

NTLM auth is disabled 

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn dc.hercules.htb -vv

Nmap scan report for dc.hercules.htb (10.129.242.196)
Host is up, received user-set (0.014s latency).
Scanned at 2026-09-11 17:52:34 BST for 66s
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
80/tcp    open  http             syn-ack
88/tcp    open  kerberos-sec     syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
389/tcp   open  ldap             syn-ack
443/tcp   open  https            syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5986/tcp  open  wsmans           syn-ack
9389/tcp  open  adws             syn-ack
49664/tcp open  unknown          syn-ack
49668/tcp open  unknown          syn-ack
51753/tcp open  unknown          syn-ack
56289/tcp open  unknown          syn-ack
56313/tcp open  unknown          syn-ack
57888/tcp open  unknown          syn-ack
57897/tcp open  unknown          syn-ack
```

## Nmap
```python
nmap -p 53,80,88,135,139,389,443,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT -Pn dc.hercules.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-11 17:54 +0100
Nmap scan report for dc.hercules.htb (10.129.242.196)
Host is up (0.014s latency).

PORT     STATE    SERVICE       VERSION
53/tcp   open     domain        Simple DNS Plus
80/tcp   open     http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Did not follow redirect to https://dc.hercules.htb/
88/tcp   open     kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-11 16:55:00Z)
135/tcp  open     msrpc         Microsoft Windows RPC
139/tcp  open     netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open     ldap          Microsoft Windows Active Directory LDAP (Domain: hercules.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
|_ssl-date: TLS randomness does not represent time
443/tcp  open     ssl/https     Microsoft-IIS/10.0
| tls-alpn: 
|   h2
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=hercules.htb
| Subject Alternative Name: DNS:hercules.htb
| Not valid before: 2024-12-04T01:34:56
|_Not valid after:  2034-12-04T01:44:56
|_http-server-header: Microsoft-IIS/10.0
445/tcp  open     microsoft-ds?
464/tcp  open     kpasswd5?
593/tcp  open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open     ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: hercules.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
3268/tcp open     ldap          Microsoft Windows Active Directory LDAP (Domain: hercules.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
3269/tcp open     ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: hercules.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
|_ssl-date: TLS randomness does not represent time
5985/tcp filtered wsman
9389/tcp open     mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

## System time
```python
ntpdate dc.hercules.htb
2026-09-11 17:57:45.113530 (+0100) -0.012739 +/- 0.101796 dc.hercules.htb 10.129.242.196 s1 no-leap
CLOCK: adj_systime: Operation not permitted
```

So since NTLM is disabled its worth checking what the clock skew is since ill be working with kerberos, but in this case its running at the same time as me

# SMB (445)

Null auth is not worth testing for here since there is no NTLM

Guest account disabled

# HTTP (80)

```python
curl -s http://hercules.htb/ -v            
* Host hercules.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.129.242.196
*   Trying 10.129.242.196:80...
* Established connection to hercules.htb (10.129.242.196 port 80) from 10.10.14.61 port 46348 
* using HTTP/1.x
> GET / HTTP/1.1
> Host: hercules.htb
> User-Agent: curl/8.21.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 301 Moved Permanently
< Content-Type: text/html; charset=UTF-8
< Location: https://hercules.htb/
< Server: Microsoft-IIS/10.0
< Date: Fri, 11 Sep 2026 17:03:19 GMT
< Content-Length: 144
< 
<head><title>Document Moved</title></head>
* Connection #0 to host hercules.htb:80 left intact
<body><h1>Object Moved</h1>This document may be found <a HREF="https://hercules.htb/">here</a></body>
```

So like nmap said, the webserver on port 80 is redirecting to the server on 443

# HTTPS (443)
