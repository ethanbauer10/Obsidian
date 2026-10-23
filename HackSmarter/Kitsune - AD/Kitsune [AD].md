# Objective and initial access
You have been hired to perform an internal pentest against the client's Domain Controller. Your task is to identify all vulnerabilities and (if possible) fully compromise the Domain.

The client has provided you with VPN access to their environment, but no other information.

# Host file setup
```python
❯❯❯ sudo nxc smb 10.1.176.128 --generate-hosts-file /etc/hosts
SMB         10.1.176.128    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:kitsune.local) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration

## Nmap
```python
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-23 16:07:53Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: kitsune.local, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.kitsune.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.kitsune.local
| Not valid before: 2026-10-12T21:14:40
|_Not valid after:  2027-10-12T21:14:40
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: kitsune.local, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.kitsune.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.kitsune.local
| Not valid before: 2026-10-12T21:14:40
|_Not valid after:  2027-10-12T21:14:40
|_ssl-date: TLS randomness does not represent time
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: kitsune.local, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.kitsune.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.kitsune.local
| Not valid before: 2026-10-12T21:14:40
|_Not valid after:  2027-10-12T21:14:40
|_ssl-date: TLS randomness does not represent time
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: kitsune.local, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.kitsune.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.kitsune.local
| Not valid before: 2026-10-12T21:14:40
|_Not valid after:  2027-10-12T21:14:40
|_ssl-date: TLS randomness does not represent time
3389/tcp open  ms-wbt-server
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.kitsune.local
| Not valid before: 2026-10-11T08:21:43
|_Not valid after:  2027-04-12T08:21:43
| rdp-ntlm-info: 
|   Target_Name: KITSUNE
|   NetBIOS_Domain_Name: KITSUNE
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: kitsune.local
|   DNS_Computer_Name: DC01.kitsune.local
|   Product_Version: 10.0.26100
|_  System_Time: 2026-10-23T16:08:41+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.991%I=7%D=10/23%Time=6ADB865E%P=x86_64-pc-linux-gnu%r(
SF:TerminalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x0
SF:2\0\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)

Null auth is enabled as with all domain controllers, but cannot use it to enumerate

The guest account is enabled, i can use this to enumerate

## Shares
```python
❯❯❯ nxc smb dc01.kitsune.local -u 'Guest' -p '' --shares
SMB         10.1.176.128    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:kitsune.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.176.128    445    DC01             [+] kitsune.local\Guest: 
SMB         10.1.176.128    445    DC01             [*] Enumerated shares
SMB         10.1.176.128    445    DC01             Share           Permissions     Remark
SMB         10.1.176.128    445    DC01             -----           -----------     ------
SMB         10.1.176.128    445    DC01             ADMIN$                          Remote Admin
SMB         10.1.176.128    445    DC01             C$                              Default share
SMB         10.1.176.128    445    DC01             Files           READ            Global file share
SMB         10.1.176.128    445    DC01             IPC$            READ            Remote IPC
SMB         10.1.176.128    445    DC01             NETLOGON                        Logon server share 
SMB         10.1.176.128    445    DC01             SYSVOL                          Logon server share
```

Read access on the `Files` share

## Users
```python
❯❯❯ nxc smb dc01.kitsune.local -u 'Guest' -p '' --rid-brute 20000 | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
ranch
schlop
pebble
ross
protagonist
svc_gmsa$
```

Ill use `--rid-brute` in order to also dump machine accounts

# `Files` share

```python
❯❯❯ smbclient //dc01.kitsune.local/Files -U 'Guest'%''
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Oct 13 05:08:20 2026
  ..                                DHS        0  Thu Oct 15 11:49:47 2026
  10-11-Calendar_October.pdf          A     3190  Tue Oct 13 03:23:43 2026
  10-11-Financial_Report.pdf          A     4076  Tue Oct 13 03:23:49 2026
  10-11-Timesheet.pdf                 A     3854  Tue Oct 13 03:21:23 2026
  logins.psafe3                       A      840  Tue Oct 13 02:59:03 2026

		12974591 blocks of size 4096. 3451899 blocks available
smb: \> recurse on 
smb: \> prompt off
smb: \> mget *
```

Ill download all the files here


