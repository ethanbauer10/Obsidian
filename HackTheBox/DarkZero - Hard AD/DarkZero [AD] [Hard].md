
# Machine info
As is common in real life pentests, you will start the DarkZero box with credentials for the following account john.w / RFulUtONCOL!

```python
john.w:RFulUtONCOL!
```

# Host file setup
```python
sudo nxc smb 10.129.48.21 --generate-hosts-file /etc/hosts 
[sudo] password for kali: 
SMB         10.129.48.21    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:darkzero.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.darkzero.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-20 17:09 +0100
Nmap scan report for dc01.darkzero.htb (10.129.48.21)
Host is up (0.014s latency).
rDNS record for 10.129.48.21: DC01.darkzero.htb
Not shown: 65513 filtered tcp ports (no-response)
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
2179/tcp  open  vmrdp
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49666/tcp open  unknown
49676/tcp open  unknown
49677/tcp open  unknown
49897/tcp open  unknown
49926/tcp open  unknown
53177/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.26 seconds
```

## Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,1433,2179,3268,3269,5985,9389 -A --min-rate=2000 -sT dc01.darkzero.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-20 17:11 +0100
Nmap scan report for dc01.darkzero.htb (10.129.48.21)
Host is up (0.014s latency).
rDNS record for 10.129.48.21: DC01.darkzero.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-20 16:11:52Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: darkzero.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2026-08-20T15:56:07
|_Not valid after:  2027-08-20T15:56:07
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: darkzero.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2026-08-20T15:56:07
|_Not valid after:  2027-08-20T15:56:07
|_ssl-date: TLS randomness does not represent time
1433/tcp open  ms-sql-s      Microsoft SQL Server 2022 16.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   10.129.48.21:1433: 
|     Target_Name: darkzero
|     NetBIOS_Domain_Name: darkzero
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: darkzero.htb
|     DNS_Computer_Name: DC01.darkzero.htb
|     DNS_Tree_Name: darkzero.htb
|_    Product_Version: 10.0.26100
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-08-20T16:07:37
|_Not valid after:  2056-08-20T16:07:37
| ms-sql-info: 
|   10.129.48.21:1433: 
|     Version: 
|       name: Microsoft SQL Server 2022 RTM
|       number: 16.00.1000.00
|       Product: Microsoft SQL Server 2022
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2026-08-20T16:13:16+00:00; +1s from scanner time.
2179/tcp open  vmrdp?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: darkzero.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2026-08-20T15:56:07
|_Not valid after:  2027-08-20T15:56:07
|_ssl-date: TLS randomness does not represent time
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: darkzero.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2026-08-20T15:56:07
|_Not valid after:  2027-08-20T15:56:07
|_ssl-date: TLS randomness does not represent time
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
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

Null auth is enabled, but cannot use it to enumerate shares or list users

The guest account is also disabled

## Using provided credentials
```python
nxc smb dc01.darkzero.htb -u 'john.w' -p 'RFulUtONCOL!' --smb-timeout 2
SMB         10.129.48.21    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:darkzero.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.48.21    445    DC01             [+] darkzero.htb\john.w:RFulUtONCOL!
```

The credentials are valid here

```python
nxc smb dc01.darkzero.htb -u 'john.w' -p 'RFulUtONCOL!' --smb-timeout 2 --shares
SMB         10.129.48.21    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:darkzero.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.48.21    445    DC01             [+] darkzero.htb\john.w:RFulUtONCOL! 
SMB         10.129.48.21    445    DC01             [*] Enumerated shares
SMB         10.129.48.21    445    DC01             Share           Permissions     Remark
SMB         10.129.48.21    445    DC01             -----           -----------     ------
SMB         10.129.48.21    445    DC01             ADMIN$                          Remote Admin
SMB         10.129.48.21    445    DC01             C$                              Default share
SMB         10.129.48.21    445    DC01             IPC$            READ            Remote IPC
SMB         10.129.48.21    445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.48.21    445    DC01             SYSVOL          READ            Logon server share 
```

Limited access on shares

```python
nxc smb dc01.darkzero.htb -u 'john.w' -p 'RFulUtONCOL!' --smb-timeout 2 --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
darkzero-ext$
john.w
```

Ill grab the user list!

There are no kerberoastable accounts

There is no access over WINRM

No password in user descriptions

No password reuse

# Bloodhound

```python
bloodyAD --host dc01.darkzero.htb -d darkzero.htb -u john.w -p 'RFulUtONCOL!' get bloodhound --transitive
[+] Connecting to LDAP server
[+] Connected to LDAP serrver
Dumping schema: 2it [00:00, 30.51it/s]
Generating lookuptable: 81it [00:00, 439.83it/s]
Dumping SDs:  99%|███████████████████████████████████████████████████████████▎| 84/85 [00:01<00:00, 58.76it/s]
Dumping domains: 100%|██████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 19.75it/s]
Dumping users: 100%|███████████████████████████████████████████████████████████| 4/4 [00:00<00:00, 189.24it/s]
Dumping computers: 100%|████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 54.44it/s]
Dumping groups: 100%|███████████████████████████████████████████████████████| 52/52 [00:00<00:00, 1317.04it/s]
Dumping GPOs: 100%|████████████████████████████████████████████████████████████| 2/2 [00:00<00:00, 111.73it/s]
Dumping OUs: 100%|██████████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 56.63it/s]
Dumping Containers: 100%|████████████████████████████████████████████████████| 19/19 [00:00<00:00, 380.53it/s]
[+] Bloodhound data saved to 20260820T164235_Bloodhound.zip
[+] Found 1 trusts
[+] Found trust darkzero.ext (S-1-5-21-1969715525-31638512-2552845157)
[+] Connecting to darkzero.ext (follow_trusts)
[-] Failed to connect to darkzero.ext ([Errno -2] Name or service not known)
[-] If it's a connectivity issue (not auth issue), make sure the hostname resolves to an IP address (eg. modify your /etc/hosts)
```

I always like to run `--transitive` with bloodyAD to detect any trusts and this time it did, but it failed likely because `darkzero.ext` does not exist in `/etc/hosts`

So ill add it and re run this

```python
bloodyAD --host dc01.darkzero.htb -d darkzero.htb -u john.w -p 'RFulUtONCOL!' get bloodhound --transitive
[+] Connecting to LDAP server
[+] Connected to LDAP serrver
Dumping schema: 2it [00:00, 29.59it/s]
Generating lookuptable: 81it [00:00, 448.77it/s]
Dumping SDs:  99%|███████████████████████████████████████████████████████████▎| 84/85 [00:01<00:00, 58.40it/s]
Dumping domains: 100%|██████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 19.52it/s]
Dumping users: 100%|███████████████████████████████████████████████████████████| 4/4 [00:00<00:00, 188.80it/s]
Dumping computers: 100%|████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 55.80it/s]
Dumping groups: 100%|███████████████████████████████████████████████████████| 52/52 [00:00<00:00, 1239.35it/s]
Dumping GPOs: 100%|████████████████████████████████████████████████████████████| 2/2 [00:00<00:00, 108.28it/s]
Dumping OUs: 100%|██████████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 54.03it/s]
Dumping Containers: 100%|████████████████████████████████████████████████████| 19/19 [00:00<00:00, 394.93it/s]
[+] Bloodhound data saved to 20260820T164556_Bloodhound.zip
[+] Found 1 trusts
[+] Found trust darkzero.ext (S-1-5-21-1969715525-31638512-2552845157)
[+] Connecting to darkzero.ext (follow_trusts)
[+] Connected to darkzero.ext (follow_trusts)
[+] Connecting to LDAP server
[+] Connected to LDAP serrver
Dumping schema: 2it [00:00, 30.71it/s]
Generating lookuptable: 81it [00:00, 436.47it/s]
Dumping SDs:  99%|███████████████████████████████████████████████████████████▎| 84/85 [00:01<00:00, 58.51it/s]
Dumping domains: 100%|██████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 19.02it/s]
Dumping users: 100%|███████████████████████████████████████████████████████████| 4/4 [00:00<00:00, 193.81it/s]
Dumping computers: 100%|████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 55.92it/s]
Dumping groups: 100%|███████████████████████████████████████████████████████| 52/52 [00:00<00:00, 1309.52it/s]
Dumping GPOs: 100%|████████████████████████████████████████████████████████████| 2/2 [00:00<00:00, 112.83it/s]
Dumping OUs: 100%|██████████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 57.12it/s]
Dumping Containers: 100%|████████████████████████████████████████████████████| 19/19 [00:00<00:00, 393.81it/s]
[+] Bloodhound data saved to 20260820T164619_Bloodhound.zip
[+] Found 1 trusts
```

Now after adding the other domain i can collect trust data to ingest

But after ingesting this into bloodhound i get an error, after some research its becuase the trust array in the json expects a string and its a number in this instance so ill modify it

https://github.com/FalconForceTeam/SOAPHound/issues/15

This gave me a guide to changing the data, and after ingesting it works

![](Pasted%20image%2020260820180755.png)

There is a bidirectional trust

There is nothing interesting on my current user!

# MSSQL

```python
mssqlclient.py darkzero.htb/'john.w':'RFulUtONCOL!'@dc01.darkzero.htb -windows-auth
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC01): Line 1: Changed database context to 'master'.
[*] INFO(DC01): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2022 RTM (16.0.1000)
[!] Press help for extra shell commands
SQL (darkzero\john.w  guest@master)>
```

I get access to the MSSQL instance

```python
SQL (darkzero\john.w  guest@master)> enum_links
SRV_NAME            SRV_PROVIDERNAME   SRV_PRODUCT   SRV_DATASOURCE      SRV_PROVIDERSTRING   SRV_LOCATION   SRV_CAT   
-----------------   ----------------   -----------   -----------------   ------------------   ------------   -------   
DC01                SQLNCLI            SQL Server    DC01                NULL                 NULL           NULL      
DC02.darkzero.ext   SQLNCLI            SQL Server    DC02.darkzero.ext   NULL                 NULL           NULL      
Linked Server       Local Login       Is Self Mapping   Remote Login   
-----------------   ---------------   ---------------   ------------   
DC02.darkzero.ext   darkzero\john.w             False   dc01_sql_svc
```


