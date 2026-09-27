# Objective and scope
You have been hired to perform a red team engagement against Trask Industries. Trask is a forward-thinking technology firm, with a heavy focus on research and development (R&D).

Trask Industries is not aware of the engagement; instead, you have been hired as a new employee in the R&D department. Your task is to begin as a "new employee" and demonstrate impact by elevating your privileges to Domain Admin (if possible).

I have also been provided a onboarding document that i will use for initial access

Also given a wordlist for any hashcracking

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.0.0.5 -vv

Completed Connect Scan at 16:17, 459.70s elapsed (65535 total ports)
Nmap scan report for dc01 (10.0.0.5)
Host is up, received user-set (0.092s latency).
Scanned at 2026-09-27 16:09:38 BST for 460s
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
80/tcp    open  http             syn-ack
88/tcp    open  kerberos-sec     syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
389/tcp   open  ldap             syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5985/tcp  open  wsman            syn-ack
45985/tcp open  unknown          syn-ack
49664/tcp open  unknown          syn-ack
52200/tcp open  unknown          syn-ack
52207/tcp open  unknown          syn-ack
52224/tcp open  unknown          syn-ack
52234/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 459.73 seconds
```

## NMap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985 -A --min-rate=200 -sT -Pn dc01.trask.hsm 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-27 16:18 +0100
Nmap scan report for dc01.trask.hsm (10.0.0.5)
Host is up (0.093s latency).
rDNS record for 10.0.0.5: dc01

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Caddy httpd
|_http-title: Site doesn't have a title.
|_http-server-header: Caddy
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-27 15:18:19Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: trask.hsm, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: trask.hsm, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```



![](Pasted%20image%2020260927160217.png)

I have been given a username and a password, which i will use

```python
k.pryde:KP_TempPass_1988!
```

These credentials dont work as domain credentials

# HTTP (80)

![](Pasted%20image%2020260927162524.png)

A lot of this site is static

There is a request i can make through the `partner with us` option

There is a subdomain `www` but its the same page

## Subdomains
```python
ffuf -u http://trask.hsm/ -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -H 'Host: FUZZ.trask.hsm' -ic -c -t 20 -mc 301,302          

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://trask.hsm/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
 :: Header           : Host: FUZZ.trask.hsm
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 20
 :: Matcher          : Response status: 301,302
________________________________________________

onboarding              [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 99ms]
```

