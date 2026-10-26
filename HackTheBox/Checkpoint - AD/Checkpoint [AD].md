# Machine info
As is common in real life pentests, you will start the Checkpoint box with credentials for the following account alex.turner / Checkpoint2024!

```python
alex.turner:Checkpoint2024!
```

# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.80.26 --generate-hosts-file /etc/hosts    
SMB         10.129.80.26    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:checkpoint.htb) (signing:True) (SMBv1:None)
```

# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc01.checkpoint.htb -vv

Nmap scan report for dc01.checkpoint.htb (10.129.80.26)
Host is up, received user-set (0.014s latency).
rDNS record for 10.129.80.26: DC01.checkpoint.htb
Scanned at 2026-10-26 16:48:48 GMT for 98s
Not shown: 65514 filtered tcp ports (no-response)
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
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
49664/tcp open  unknown          syn-ack
49672/tcp open  unknown          syn-ack
49674/tcp open  unknown          syn-ack
49679/tcp open  unknown          syn-ack
49680/tcp open  unknown          syn-ack
49683/tcp open  unknown          syn-ack
49710/tcp open  unknown          syn-ack
49718/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 98.98 seconds
```

## Nmap
```python
❯❯❯ nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -Pn -sT dc01.checkpoint.htb    
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-26 16:51 +0000
Nmap scan report for dc01.checkpoint.htb (10.129.80.26)
Host is up (0.013s latency).
rDNS record for 10.129.80.26: DC01.checkpoint.htb

PORT     STATE SERVICE           VERSION
53/tcp   open  domain            Simple DNS Plus
88/tcp   open  kerberos-sec      Microsoft Windows Kerberos (server time: 2026-10-26 23:51:30Z)
135/tcp  open  msrpc             Microsoft Windows RPC
139/tcp  open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: checkpoint.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ldapssl?
3268/tcp open  ldap              Microsoft Windows Active Directory LDAP (Domain: checkpoint.htb, Site: Default-First-Site-Name)
3269/tcp open  globalcatLDAPssl?
5985/tcp open  http              Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf            .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|11|2012|2016 (88%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (88%), Microsoft Windows 11 24H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)

Null auth disabled

Guest account is disabled

But i can try the provided credentials

```python
❯❯❯ nxc smb dc01.checkpoint.htb -u 'alex.turner' -p 'Checkpoint2024!' --smb-timeout 5
SMB         10.129.80.26    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:checkpoint.htb) (signing:True) (SMBv1:None)
SMB         10.129.80.26    445    DC01             [+] checkpoint.htb\alex.turner:Checkpoint2024! 
```

As seen here i can auth as `alex.turner`

## Shares
```python
❯❯❯ nxc smb dc01.checkpoint.htb -u 'alex.turner' -p 'Checkpoint2024!' --smb-timeout 5 --shares
SMB         10.129.80.26    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:checkpoint.htb) (signing:True) (SMBv1:None)
SMB         10.129.80.26    445    DC01             [+] checkpoint.htb\alex.turner:Checkpoint2024! 
SMB         10.129.80.26    445    DC01             [*] Enumerated shares
SMB         10.129.80.26    445    DC01             Share           Permissions     Remark
SMB         10.129.80.26    445    DC01             -----           -----------     ------
SMB         10.129.80.26    445    DC01             ADMIN$                          Remote Admin
SMB         10.129.80.26    445    DC01             C$                              Default share
SMB         10.129.80.26    445    DC01             DevDrop         READ            VS Code extensions share for approved .vsix packages compatible with VS Code engine 1.118.0
SMB         10.129.80.26    445    DC01             IPC$            READ            Remote IPC
SMB         10.129.80.26    445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.80.26    445    DC01             SYSVOL          READ            Logon server share 
SMB         10.129.80.26    445    DC01             VMBackups 
```

Read access on a devdrop share, it stores VS code extensions

## Users
```python
❯❯❯ nxc smb dc01.checkpoint.htb -u 'alex.turner' -p 'Checkpoint2024!' --smb-timeout 5 --rid-brute 20000 | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
alex.turner
ryan.brooks
svc_deploy
james.harper
sarah.mitchell
emily.carter
david.reynolds
jessica.coleman
lauren.flores
michael.torres
kevin.patterson
brian.jenkins
megan.perry
max.palmer
```

Ill use `--rid-brute` to also get machine accounts

# Writable objects as `alex.turner`

```python
❯❯❯ bloodyAD --host dc01.checkpoint.htb -d checkpoint.htb -u alex.turner -p 'Checkpoint2024!' get writable               

distinguishedName: CN=Deleted Objects,DC=checkpoint,DC=htb
DACL: WRITE

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=checkpoint,DC=htb
permission: WRITE

distinguishedName: OU=Employees,DC=checkpoint,DC=htb
permission: CREATE_CHILD

distinguishedName: CN=Alex Turner,OU=Employees,DC=checkpoint,DC=htb
permission: WRITE

distinguishedName: CN=Mark Davies\0ADEL:2217e877-e2a2-47d7-91d4-99ede36f367e,CN=Deleted Objects,DC=checkpoint,DC=htb
permission: WRITE

distinguishedName: DC=checkpoint.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=checkpoint,DC=htb
permission: CREATE_CHILD

distinguishedName: DC=_msdcs.checkpoint.htb,CN=MicrosoftDNS,DC=ForestDnsZones,DC=checkpoint,DC=htb
permission: CREATE_CHILD
```

Some things here are default, but there is also some interesting info

It looks like i can restore the `mark.davies` user, i also have create child on the employees OU, this is likely where i need to restore the user to

Its also very interesting i have WriteDACL on the deleted objects

