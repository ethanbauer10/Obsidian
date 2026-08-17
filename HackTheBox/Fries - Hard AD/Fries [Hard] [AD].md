# Machine Information
Please allow up to 7 minutes for services to load. As is common in real life Windows penetration tests, you will start the Fries box with credentials for the following account : d.cooper@fries.htb / D4LE11maan!!

```python
d.cooper:D4LE11maan!!
```

# Host file setup
```python
sudo nxc smb 10.129.244.72 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.244.72   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:fries.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.fries.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-17 18:40 +0100
Nmap scan report for dc01.fries.htb (10.129.244.72)
Host is up (0.015s latency).
rDNS record for 10.129.244.72: DC01.fries.htb
Not shown: 65510 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
443/tcp   open  https
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
2179/tcp  open  vmrdp
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49667/tcp open  unknown
49681/tcp open  unknown
49682/tcp open  unknown
49684/tcp open  unknown
49693/tcp open  unknown
49917/tcp open  unknown
49934/tcp open  unknown
49975/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.32 seconds
```

## Nmap
```python
nmap -p 22,53,80,88,135,139,389,443,445,464,593,636,2179,3268,3269,5985 -A --min-rate=2000 -sT dc01.fries.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-17 18:43 +0100
Nmap scan report for dc01.fries.htb (10.129.244.72)
Host is up (0.015s latency).
rDNS record for 10.129.244.72: DC01.fries.htb

PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b3:a8:f7:5d:60:e8:66:16:ca:92:f6:76:ba:b8:33:c2 (ECDSA)
|_  256 07:ef:11:a6:a0:7d:2b:4d:e8:68:79:1a:7b:a7:a9:cd (ED25519)
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://fries.htb/
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-18 00:43:31Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: fries.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fries.htb, DNS:fries.htb, DNS:FRIES
| Not valid before: 2026-06-05T16:23:40
|_Not valid after:  2106-06-05T16:23:40
|_ssl-date: 2026-08-18T00:44:56+00:00; +6h59m57s from scanner time.
443/tcp  open  ssl/http      nginx 1.18.0 (Ubuntu)
| tls-nextprotoneg: 
|_  http/1.1
|_http-server-header: nginx/1.18.0 (Ubuntu)
| tls-alpn: 
|_  http/1.1
|_http-title: Site doesn't have a title (text/html;charset=ISO-8859-1).
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=pwm.fries.htb/organizationName=Fries Foods LTD/stateOrProvinceName=Madrid/countryName=SP
| Not valid before: 2025-06-01T22:06:09
|_Not valid after:  2026-06-01T22:06:09
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fries.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-18T00:44:57+00:00; +6h59m57s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fries.htb, DNS:fries.htb, DNS:FRIES
| Not valid before: 2026-06-05T16:23:40
|_Not valid after:  2106-06-05T16:23:40
2179/tcp open  vmrdp?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: fries.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-18T00:44:57+00:00; +6h59m57s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fries.htb, DNS:fries.htb, DNS:FRIES
| Not valid before: 2026-06-05T16:23:40
|_Not valid after:  2106-06-05T16:23:40
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fries.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fries.htb, DNS:fries.htb, DNS:FRIES
| Not valid before: 2026-06-05T16:23:40
|_Not valid after:  2106-06-05T16:23:40
|_ssl-date: 2026-08-18T00:44:57+00:00; +6h59m57s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (91%), MikroTik RouterOS 7.X (91%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6
Aggressive OS guesses: Linux 4.15 - 5.19 (91%), Linux 5.0 - 5.14 (91%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (91%), Linux 2.6.32 - 3.13 (85%), Linux 3.10 - 4.11 (85%), Linux 3.2 - 4.14 (85%), Linux 3.4 - 3.10 (85%), Linux 4.15 (85%), Linux 5.14 - 6.8 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OSs: Linux, Windows; CPE: cpe:/o:linux:linux_kernel, cpe:/o:microsoft:windows
```

Some interesting services running

# SMB (445)

Null auth doesnt give me anything

Guest account is also disabled

## Using provided credentials
```python
nxc smb dc01.fries.htb -u 'd.cooper' -p 'D4LE11maan!!' 
SMB         10.129.244.72   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:fries.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.72   445    DC01             [-] fries.htb\d.cooper:D4LE11maan!! STATUS_LOGON_FAILURE
```

So these credentials dont work on SMB

# SSH (22)

```python
ssh d.cooper@dc01.fries.htb            
The authenticity of host 'dc01.fries.htb (10.129.244.72)' can't be established.
ED25519 key fingerprint is: SHA256:++SuiiJ+ZwG7d5q6fb9KqhQRx1gGhVOfGR24bbTuipg
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'dc01.fries.htb' (ED25519) to the list of known hosts.
d.cooper@dc01.fries.htb's password:
```

So the credentials get me access here either!

Also worth noting that its using password auth

# HTTP (80)

![](Pasted%20image%2020260817185300.png)

No login portal here so i cannot use credentials!

Also not a lot on the website in terms of functionality

## Subdomains
```python
ffuf -u http://fries.htb/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.fries.htb' -ic -c -t 30 -fs 154

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://fries.htb/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.fries.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 30
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 154
________________________________________________

code                    [Status: 200, Size: 13593, Words: 1048, Lines: 272, Duration: 41ms]
:: Progress: [114438/114438] :: Job [1/1] :: 2112 req/sec :: Duration: [0:01:16] :: Errors: 0 ::
```

Found a subdomain `code`

# `code` subdomain 

This subdomain is a Gitea instance

There are no public or archived repos

![](Pasted%20image%2020260817193726.png)

There are two users however

