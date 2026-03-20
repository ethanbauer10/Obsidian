
# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.7.67 -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-20 15:32 GMT
Nmap scan report for 10.129.7.67
Host is up (0.013s latency).
Not shown: 65515 filtered tcp ports (no-response)
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
1433/tcp  open  ms-sql-s
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49669/tcp open  unknown
49691/tcp open  unknown
49692/tcp open  unknown
49704/tcp open  unknown
49714/tcp open  unknown
49742/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.71 seconds
```
## Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,1433,3268,3269,5985,9389 -A --min-rate=2000 -sT 10.129.7.67 -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-20 15:37 GMT
Nmap scan report for 10.129.7.67
Host is up (0.013s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-20 23:37:34Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
|_ssl-date: 2026-03-20T23:38:54+00:00; +8h00m00s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
|_ssl-date: 2026-03-20T23:38:54+00:00; +8h00m00s from scanner time.
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
|_ms-sql-info: ERROR: Script execution failed (use -d to debug)
|_ms-sql-ntlm-info: ERROR: Script execution failed (use -d to debug)
|_ssl-date: 2026-03-20T23:38:54+00:00; +8h00m00s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-03-20T23:25:18
|_Not valid after:  2056-03-20T23:25:18
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
|_ssl-date: 2026-03-20T23:38:54+00:00; +8h00m00s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2026-03-20T23:38:54+00:00; +8h00m00s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Null authentication is enabled, but cannot list users or access shares
## Guest access
### Shares
```python
nxc smb dc.sequel.htb -u 'Guest' -p '' --shares
SMB         10.129.7.67     445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.7.67     445    DC               [+] sequel.htb\Guest: 
SMB         10.129.7.67     445    DC               [*] Enumerated shares
SMB         10.129.7.67     445    DC               Share           Permissions     Remark
SMB         10.129.7.67     445    DC               -----           -----------     ------
SMB         10.129.7.67     445    DC               ADMIN$                          Remote Admin
SMB         10.129.7.67     445    DC               C$                              Default share
SMB         10.129.7.67     445    DC               IPC$            READ            Remote IPC
SMB         10.129.7.67     445    DC               NETLOGON                        Logon server share 
SMB         10.129.7.67     445    DC               Public          READ            
SMB         10.129.7.67     445    DC               SYSVOL                          Logon server share
```
### Users
```python
nxc smb dc.sequel.htb -u 'Guest' -p '' --rid-brute             
SMB         10.129.7.67     445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.7.67     445    DC               [+] sequel.htb\Guest: 
SMB         10.129.7.67     445    DC               498: sequel\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.7.67     445    DC               500: sequel\Administrator (SidTypeUser)
SMB         10.129.7.67     445    DC               501: sequel\Guest (SidTypeUser)
SMB         10.129.7.67     445    DC               502: sequel\krbtgt (SidTypeUser)
SMB         10.129.7.67     445    DC               512: sequel\Domain Admins (SidTypeGroup)
SMB         10.129.7.67     445    DC               513: sequel\Domain Users (SidTypeGroup)
SMB         10.129.7.67     445    DC               514: sequel\Domain Guests (SidTypeGroup)
SMB         10.129.7.67     445    DC               515: sequel\Domain Computers (SidTypeGroup)
SMB         10.129.7.67     445    DC               516: sequel\Domain Controllers (SidTypeGroup)
SMB         10.129.7.67     445    DC               517: sequel\Cert Publishers (SidTypeAlias)
SMB         10.129.7.67     445    DC               518: sequel\Schema Admins (SidTypeGroup)
SMB         10.129.7.67     445    DC               519: sequel\Enterprise Admins (SidTypeGroup)
SMB         10.129.7.67     445    DC               520: sequel\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.7.67     445    DC               521: sequel\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.7.67     445    DC               522: sequel\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.7.67     445    DC               525: sequel\Protected Users (SidTypeGroup)
SMB         10.129.7.67     445    DC               526: sequel\Key Admins (SidTypeGroup)
SMB         10.129.7.67     445    DC               527: sequel\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.7.67     445    DC               553: sequel\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.7.67     445    DC               571: sequel\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.7.67     445    DC               572: sequel\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.7.67     445    DC               1000: sequel\DC$ (SidTypeUser)
SMB         10.129.7.67     445    DC               1101: sequel\DnsAdmins (SidTypeAlias)
SMB         10.129.7.67     445    DC               1102: sequel\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.7.67     445    DC               1103: sequel\Tom.Henn (SidTypeUser)
SMB         10.129.7.67     445    DC               1104: sequel\Brandon.Brown (SidTypeUser)
SMB         10.129.7.67     445    DC               1105: sequel\Ryan.Cooper (SidTypeUser)
SMB         10.129.7.67     445    DC               1106: sequel\sql_svc (SidTypeUser)
SMB         10.129.7.67     445    DC               1107: sequel\James.Roberts (SidTypeUser)
SMB         10.129.7.67     445    DC               1108: sequel\Nicole.Thompson (SidTypeUser)
SMB         10.129.7.67     445    DC               1109: sequel\SQLServer2005SQLBrowserUser$DC (SidTypeAlias)
```
From this i have made a userlist which i can use to try things like asrep roasting

AS-REP roasting failed

My next move is to access the `Public` 