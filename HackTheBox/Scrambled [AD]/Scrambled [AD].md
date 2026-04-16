# Host file setup
```python
sudo nxc smb 10.129.20.49 --generate-hosts-file /etc/hosts 
[sudo] password for kali: 
SMB         10.129.20.49    445    DC1              [*]  x64 (name:DC1) (domain:scrm.local) (signing:True) (SMBv1:None) (NTLM:False)
```
- NTLM auth is disabled
- SMB signing is enabled

# System time
```python
ntpdate -q dc1.scrm.local                  
2026-04-16 16:21:42.183812 (+0100) -0.237305 +/- 0.006528 dc1.scrm.local 10.129.20.49 s1 no-leap
```
Since NTLM auth is disabled ill have to use kerberos auth
# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc1.scrm.local                                                                 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-16 16:18 +0100
Nmap scan report for dc1.scrm.local (10.129.20.49)
Host is up (0.013s latency).
rDNS record for 10.129.20.49: DC1.scrm.local
Not shown: 65514 filtered tcp ports (no-response)
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
1433/tcp  open  ms-sql-s
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
4411/tcp  open  found
5985/tcp  open  wsman
9389/tcp  open  adws
49667/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49701/tcp open  unknown
49712/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.79 seconds
```
MS-SQL is open

## Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,1433,3268,3269,4411,5985,9389 -A --min-rate=2000 -sT dc1.scrm.local
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-16 16:21 +0100
Nmap scan report for dc1.scrm.local (10.129.20.49)
Host is up (0.015s latency).
rDNS record for 10.129.20.49: DC1.scrm.local

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: Scramble Corp Intranet
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-16 15:21:12Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: scrm.local, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC1.scrm.local
| Not valid before: 2024-09-04T11:14:45
|_Not valid after:  2121-06-08T22:39:53
|_ssl-date: 2026-04-16T15:24:24+00:00; 0s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: scrm.local, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC1.scrm.local
| Not valid before: 2024-09-04T11:14:45
|_Not valid after:  2121-06-08T22:39:53
|_ssl-date: 2026-04-16T15:24:24+00:00; 0s from scanner time.
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-04-16T15:16:09
|_Not valid after:  2056-04-16T15:16:09
| ms-sql-info: 
|   10.129.20.49:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2026-04-16T15:24:24+00:00; 0s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: scrm.local, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-16T15:24:24+00:00; 0s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC1.scrm.local
| Not valid before: 2024-09-04T11:14:45
|_Not valid after:  2121-06-08T22:39:53
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: scrm.local, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC1.scrm.local
| Not valid before: 2024-09-04T11:14:45
|_Not valid after:  2121-06-08T22:39:53
|_ssl-date: 2026-04-16T15:24:24+00:00; 0s from scanner time.
4411/tcp open  found?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, NCP, NULL, NotesRPC, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|     SCRAMBLECORP_ORDERS_V1.0.3;
|   FourOhFourRequest, GetRequest, HTTPOptions, Help, LPDString, RTSPRequest, SIPOptions: 
|     SCRAMBLECORP_ORDERS_V1.0.3;
|_    ERROR_UNKNOWN_COMMAND;
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC1; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Since NTLM auth is disabled null auth is not a possibility

Guest account appears to be disabled

# HTTP (80)

![990](Pasted%20image%2020260416163524.png)

- User `ksimpson`
- User `support`

![810](Pasted%20image%2020260416163703.png)

This form looks like it allows me to create another user

There is also a reference to a sales app and port 4411

![1003](Pasted%20image%2020260416164853.png)

I could try `ksimpson:ksimpson` 

# Compromising `ksimpson`
```python
faketime '16:44:35.396530' nxc ldap dc1.scrm.local -u 'ksimpson' -p 'ksimpson' -k                
LDAP        dc1.scrm.local  389    DC1              [*] None (name:DC1) (domain:scrm.local) (signing:None) (channel binding:Never) (NTLM:False)
LDAP        dc1.scrm.local  389    DC1              [+] scrm.local\ksimpson:ksimpson
```
This user is compromised

## Access as `ksimpson`
### Shares
```python
faketime '16:50:07.318311' nxc smb dc1.scrm.local -u 'ksimpson' -p 'ksimpson' -k --shares
SMB         dc1.scrm.local  445    dc1              [*]  x64 (name:dc1) (domain:scrm.local) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc1.scrm.local  445    dc1              [+] scrm.local\ksimpson:ksimpson 
SMB         dc1.scrm.local  445    dc1              [*] Enumerated shares
SMB         dc1.scrm.local  445    dc1              Share           Permissions     Remark
SMB         dc1.scrm.local  445    dc1              -----           -----------     ------
SMB         dc1.scrm.local  445    dc1              ADMIN$                          Remote Admin
SMB         dc1.scrm.local  445    dc1              C$                              Default share
SMB         dc1.scrm.local  445    dc1              HR                              
SMB         dc1.scrm.local  445    dc1              IPC$            READ            Remote IPC
SMB         dc1.scrm.local  445    dc1              IT                              
SMB         dc1.scrm.local  445    dc1              NETLOGON        READ            Logon server share 
SMB         dc1.scrm.local  445    dc1              Public          READ            
SMB         dc1.scrm.local  445    dc1              Sales                           
SMB         dc1.scrm.local  445    dc1              SYSVOL          READ            Logon server share
```
Read access on a few shares

### Users
```python
faketime '16:50:07.318311' nxc smb dc1.scrm.local -u 'ksimpson' -p 'ksimpson' -k --users-export users.txt
SMB         dc1.scrm.local  445    dc1              [*]  x64 (name:dc1) (domain:scrm.local) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc1.scrm.local  445    dc1              [+] scrm.local\ksimpson:ksimpson 
SMB         dc1.scrm.local  445    dc1              -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         dc1.scrm.local  445    dc1              administrator                 2021-11-08 00:35:59 0       Built-in account for administering the computer/domain 
SMB         dc1.scrm.local  445    dc1              Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         dc1.scrm.local  445    dc1              krbtgt                        2020-01-26 19:15:47 0       Key Distribution Center Service Account 
SMB         dc1.scrm.local  445    dc1              tstar                         2021-11-05 14:55:51 0        
SMB         dc1.scrm.local  445    dc1              asmith                        2020-02-08 22:29:01 0        
SMB         dc1.scrm.local  445    dc1              sjenkins                      2020-02-08 23:11:26 0        
SMB         dc1.scrm.local  445    dc1              sdonington                    2020-02-08 23:11:54 0        
SMB         dc1.scrm.local  445    dc1              backupsvc                     2021-10-31 20:49:04 0       Backup system service 
SMB         dc1.scrm.local  445    dc1              jhall                         2021-10-31 21:09:23 0        
SMB         dc1.scrm.local  445    dc1              rsmith                        2021-10-31 21:09:54 0        
SMB         dc1.scrm.local  445    dc1              ehooker                       2021-11-03 19:02:41 0        
SMB         dc1.scrm.local  445    dc1              khicks                        2021-11-01 15:36:08 0        
SMB         dc1.scrm.local  445    dc1              sqlsvc                        2021-11-03 16:32:02 0       SQL server 
SMB         dc1.scrm.local  445    dc1              miscsvc                       2021-11-03 18:07:47 0       Miscellaneous scheduled tasks and services 
SMB         dc1.scrm.local  445    dc1              ksimpson                      2021-11-04 00:30:57 0        
SMB         dc1.scrm.local  445    dc1              [*] Enumerated 15 local users: SCRM
SMB         dc1.scrm.local  445    dc1              [*] Writing 15 local users to users.txt
```

# `Public` SMB share
```python
faketime '16:56:27.733020' impacket-smbclient scrm.local/ksimpson:ksimpson@dc1.scrm.local -k
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
Type help for list of commands
# shares
ADMIN$
C$
HR
IPC$
IT
NETLOGON
Public
Sales
SYSVOL
# use Public
# dir
*** Unknown syntax: dir
# ls
drw-rw-rw-          0  Thu Nov  4 22:23:19 2021 .
drw-rw-rw-          0  Thu Nov  4 22:23:19 2021 ..
-rw-rw-rw-     630106  Fri Nov  5 17:45:07 2021 Network Security Changes.pdf
```
Found a .pdf file

