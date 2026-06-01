
# Host file setup

```python
sudo nxc smb 10.129.234.48 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.234.48   445    DC-JPQ225        [*]  x64 (name:DC-JPQ225) (domain:cicada.vl) (signing:True) (SMBv1:None) (NTLM:False)
```
- NTLM auth disabled
- SMB signing is enabled

# Enumeration

## Open ports

```python
nmap -p- --min-rate=2000 -sT DC-JPQ225.cicada.vl
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-01 18:22 +0100
Nmap scan report for DC-JPQ225.cicada.vl (10.129.234.48)
Host is up (0.013s latency).
Not shown: 65512 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
111/tcp   open  rpcbind
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
2049/tcp  open  nfs
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
9389/tcp  open  adws
49217/tcp open  unknown
49219/tcp open  unknown
49236/tcp open  unknown
49310/tcp open  unknown
49664/tcp open  unknown
49668/tcp open  unknown
49773/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.77 seconds
```

## Nmap

```python
nmap -p 53,80,88,111,135,139,389,445,464,593,636,2049,3268,3269,3389,9389 -A --min-rate=2000 -sT DC-JPQ225.cicada.vl
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-01 18:24 +0100
Nmap scan report for DC-JPQ225.cicada.vl (10.129.234.48)
Host is up (0.015s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-06-01 17:24:18Z)
111/tcp  open  rpcbind?
| rpcinfo: 
|   program version    port/proto  service
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|_  100005  1,2,3       2049/udp6  mountd
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-06-01T17:11:54
|_Not valid after:  2027-06-01T17:11:54
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-06-01T17:11:54
|_Not valid after:  2027-06-01T17:11:54
|_ssl-date: TLS randomness does not represent time
2049/tcp open  mountd        1-3 (RPC #100005)
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-06-01T17:11:54
|_Not valid after:  2027-06-01T17:11:54
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-06-01T17:11:54
|_Not valid after:  2027-06-01T17:11:54
|_ssl-date: TLS randomness does not represent time
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-06-01T17:25:44+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Not valid before: 2026-05-31T17:19:31
|_Not valid after:  2026-11-30T17:19:31
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC-JPQ225; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)

Since NTLM is disabled ill have to use kerberos, that means i cannot test for null auth

The Guest account is disabled

# HTTP (80)

The web server is just a default IIS server

But there is a `/certsrv` endpoint, so potentially vulnerable to ESC8??

# RPC (111)

I cannot connect using a null session

# NFS (2049)

```python
nxc nfs DC-JPQ225.cicada.vl            
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Supported NFS versions: (2, 3, 4) (root escape:False)
```
Its not vulnerable to root escape!

```python
nxc nfs DC-JPQ225.cicada.vl --shares
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Enumerating NFS Shares
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl UID        Perms    Storage Usage    Share                          Access List    
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl ---        -----    -------------    -----                          -----------    
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 rw-      12.1GB/15.4GB    /profiles                      Everyone
```
Found the share `/profiles`


## Accessing `/profiles` share
```python
nxc nfs DC-JPQ225.cicada.vl --enum-shares
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Enumerating NFS Shares Directories
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [+] /profiles
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl UID        Perms    File Size      File Path                                     Access List    
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl ---        -----    ---------      ---------                                     -----------    
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 ---      402.0B         /profiles/Administrator/Documents/desktop.ini Everyone       
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 rwx      1.4MB          /profiles/Administrator/vacation.png          Everyone       
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 rw-      -              /profiles/Rosie.Powell/Documents/$RECYCLE.BIN/ Everyone       
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 rwx      402.0B         /profiles/Rosie.Powell/Documents/desktop.ini  Everyone       
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 rwx      1.7MB          /profiles/Rosie.Powell/marketing.png          Everyone
```


```python
nxc nfs DC-JPQ225.cicada.vl --share '/profiles' --ls '/'
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl UID        Perms  File Size     File Path
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl ---        -----  ---------     ---------
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   4.0KB         /profiles/.
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   4.0KB         /profiles/..
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Administrator
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Daniel.Marshall
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Debra.Wright
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Jane.Carter
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Jordan.Francis
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Joyce.Andrews
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Katie.Ward
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Megan.Simpson
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Richard.Gibbons
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Rosie.Powell
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Shirley.West
```
I can also list out all the users with profiles, using root escape

Now i can use this to go through all the different users profiles!

So it looks like the administrator and `rosie.powell` both have images in their directories!

```python
nxc nfs DC-JPQ225.cicada.vl --share '/profiles' --get-file 'Rosie.Powell/marketing.png' marketing.png 
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Downloading Rosie.Powell/marketing.png to marketing.png
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl File successfully downloaded from Rosie.Powell/marketing.png to marketing.png
```
I cannot download the other PNG, i think its becuase it exists in the administrator directory!

![](Pasted%20image%2020260601195959.png)

Looks like ive found a password, that i can try to spray against the users that have profiles!

# Password spray

```python
nxc nfs DC-JPQ225.cicada.vl --share '/profiles' --ls '/' | grep '64.0B' | cut -d '/' -f 3 > users.txt
```
First ill use the command earlier to list the users with a profile, then modify the output and put it into users.txt

```python

```