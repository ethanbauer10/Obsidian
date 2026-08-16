# Host file setup
```python
sudo nxc smb 10.129.232.60 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.232.60   445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc.absolute.htb                                                           
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-16 15:26 +0100
Nmap scan report for dc.absolute.htb (10.129.232.60)
Host is up (0.012s latency).
rDNS record for 10.129.232.60: DC.absolute.htb
Not shown: 65509 closed tcp ports (conn-refused)
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
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49677/tcp open  unknown
49690/tcp open  unknown
49691/tcp open  unknown
49695/tcp open  unknown
49712/tcp open  unknown
49715/tcp open  unknown
51753/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 11.52 seconds
```

## Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT dc.absolute.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-16 15:28 +0100
Nmap scan report for dc.absolute.htb (10.129.232.60)
Host is up (0.013s latency).
rDNS record for 10.129.232.60: DC.absolute.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Absolute
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-16 21:28:47Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: absolute.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.absolute.htb, DNS:absolute.htb, DNS:absolute
| Not valid before: 2026-05-26T20:10:43
|_Not valid after:  2043-07-17T21:20:54
|_ssl-date: 2026-08-16T21:29:40+00:00; +6h59m55s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: absolute.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-16T21:29:40+00:00; +6h59m55s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.absolute.htb, DNS:absolute.htb, DNS:absolute
| Not valid before: 2026-05-26T20:10:43
|_Not valid after:  2043-07-17T21:20:54
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: absolute.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.absolute.htb, DNS:absolute.htb, DNS:absolute
| Not valid before: 2026-05-26T20:10:43
|_Not valid after:  2043-07-17T21:20:54
|_ssl-date: 2026-08-16T21:29:40+00:00; +6h59m55s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: absolute.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.absolute.htb, DNS:absolute.htb, DNS:absolute
| Not valid before: 2026-05-26T20:10:43
|_Not valid after:  2043-07-17T21:20:54
|_ssl-date: 2026-08-16T21:29:40+00:00; +6h59m55s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 10|2019|11|2022|2012|2016 (96%)
OS CPE: cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows 10 1909 - 2004 (96%), Microsoft Windows Server 2019 (96%), Microsoft Windows 10 1709 - 22H2 (94%), Microsoft Windows 10 1909 (92%), Microsoft Windows 11 24H2 - 25H2 (92%), Microsoft Windows Server 2022 (92%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows 10 20H2 (90%), Microsoft Windows Server 2016 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)

Null auth is enabled by default on all DCs, but i cannot do anything with it here

Guest account is also disabled!

# HTTP (80)

Nuclei did not find anything

No vhosts

```python
exiftool hero_1.jpg 
ExifTool Version Number         : 13.55
File Name                       : hero_1.jpg
Directory                       : .
File Size                       : 407 kB
File Modification Date/Time     : 2022:06:07 20:45:20+01:00
File Access Date/Time           : 2026:08:16 15:51:10+01:00
File Inode Change Date/Time     : 2026:08:16 15:51:10+01:00
File Permissions                : -rw-rw-r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
Exif Byte Order                 : Little-endian (Intel, II)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Artist                          : James Roberts
Y Cb Cr Positioning             : Centered
Quality                         : 60%
XMP Toolkit                     : Image::ExifTool 11.88
Author                          : James Roberts
Creator Tool                    : Adobe Photoshop CC 2018 Macintosh
Derived From Document ID        : 6413FD608B5C21D0939F910C0EFBBE44
Derived From Instance ID        : 6413FD608B5C21D0939F910C0EFBBE44
Document ID                     : xmp.did:887A47FA048811EA8574B646AF4FC464
Instance ID                     : xmp.iid:887A47F9048811EA8574B646AF4FC464
DCT Encode Version              : 100
APP14 Flags 0                   : [14], Encoded with Blend=1 downsampling
APP14 Flags 1                   : (none)
Color Transform                 : YCbCr
Image Width                     : 1900
Image Height                    : 1150
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
Image Size                      : 1900x1150
Megapixels                      : 2.2
```

Feroxbuster found some images and after using wget to download on of them, i see a username behind the image. I might be able to use this to get some valid usernames

