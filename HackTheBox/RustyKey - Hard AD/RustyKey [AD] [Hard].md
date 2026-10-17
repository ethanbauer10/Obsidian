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

# Timeroasting leads to user compromise

```python

```