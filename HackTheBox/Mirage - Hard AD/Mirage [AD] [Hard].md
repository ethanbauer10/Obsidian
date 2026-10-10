# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.232.163 --generate-hosts-file /etc/hosts     

SMB         10.129.232.163  445    dc01             [*]  x64 (name:dc01) (domain:mirage.htb) (signing:True) (SMBv1:None) (NTLM:False)
```

# Setting up kerberos realm
```python
❯❯❯ sudo nxc smb 10.129.232.163 --generate-krb5-file /etc/krb5.conf 
SMB         10.129.232.163  445    dc01             [*]  x64 (name:dc01) (domain:mirage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         10.129.232.163  445    dc01             [+] krb5 conf saved to: /etc/krb5.conf
SMB         10.129.232.163  445    dc01             [+] Run the following command to use the conf file: export KRB5_CONFIG=/etc/krb5.conf
```

Since NTLM is disabled, ill be interacting with kerberos so ill set this now

# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn 10.129.232.163 -vv

Nmap scan report for 10.129.232.163
Host is up, received user-set (0.013s latency).
Scanned at 2026-10-10 16:04:09 BST for 11s
Not shown: 65506 closed tcp ports (conn-refused)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
88/tcp    open  kerberos-sec     syn-ack
111/tcp   open  rpcbind          syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
389/tcp   open  ldap             syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
2049/tcp  open  nfs              syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
4222/tcp  open  vrml-multi-use   syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
47001/tcp open  winrm            syn-ack
49664/tcp open  unknown          syn-ack
49665/tcp open  unknown          syn-ack
49666/tcp open  unknown          syn-ack
49667/tcp open  unknown          syn-ack
49668/tcp open  unknown          syn-ack
51462/tcp open  unknown          syn-ack
51463/tcp open  unknown          syn-ack
51478/tcp open  unknown          syn-ack
51484/tcp open  unknown          syn-ack
51508/tcp open  unknown          syn-ack
58086/tcp open  unknown          syn-ack
62970/tcp open  unknown          syn-ack
```

## Nmap
```python
❯❯❯ nmap -p 53,88,111,135,139,389,445,464,593,636,2049,3268,3269,4222,5985 -A --min-rate=2000 -sT -Pn dc01.mirage.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-10 16:08 +0100
Nmap scan report for dc01.mirage.htb (10.129.232.163)
Host is up (0.013s latency).

PORT     STATE SERVICE         VERSION
53/tcp   open  domain          Simple DNS Plus
88/tcp   open  kerberos-sec    Microsoft Windows Kerberos (server time: 2026-10-10 22:08:49Z)
111/tcp  open  rpcbind         2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp  open  msrpc           Microsoft Windows RPC
139/tcp  open  netbios-ssn     Microsoft Windows netbios-ssn
389/tcp  open  ldap            Microsoft Windows Active Directory LDAP (Domain: mirage.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.mirage.htb, DNS:mirage.htb, DNS:MIRAGE
| Not valid before: 2025-07-04T19:58:41
|_Not valid after:  2105-07-04T19:58:41
|_ssl-date: TLS randomness does not represent time
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http      Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap        Microsoft Windows Active Directory LDAP (Domain: mirage.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.mirage.htb, DNS:mirage.htb, DNS:MIRAGE
| Not valid before: 2025-07-04T19:58:41
|_Not valid after:  2105-07-04T19:58:41
2049/tcp open  nlockmgr        1-4 (RPC #100021)
3268/tcp open  ldap            Microsoft Windows Active Directory LDAP (Domain: mirage.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.mirage.htb, DNS:mirage.htb, DNS:MIRAGE
| Not valid before: 2025-07-04T19:58:41
|_Not valid after:  2105-07-04T19:58:41
3269/tcp open  ssl/ldap        Microsoft Windows Active Directory LDAP (Domain: mirage.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.mirage.htb, DNS:mirage.htb, DNS:MIRAGE
| Not valid before: 2025-07-04T19:58:41
|_Not valid after:  2105-07-04T19:58:41
|_ssl-date: TLS randomness does not represent time
4222/tcp open  vrml-multi-use?
| fingerprint-strings: 
|   GenericLines: 
|     INFO {"server_id":"NCN74AT3XD6OBJIKFUN3TNNWT6Q3JNI4RUDLWYJPBMSNG4AVWU53MQSV","server_name":"NCN74AT3XD6OBJIKFUN3TNNWT6Q3JNI4RUDLWYJPBMSNG4AVWU53MQSV","version":"2.11.3","proto":1,"git_commit":"a82cfda","go":"go1.24.2","host":"0.0.0.0","port":4222,"headers":true,"auth_required":true,"max_payload":1048576,"jetstream":true,"client_id":17,"client_ip":"10.10.14.61","xkey":"XBXOTVN3PTCXANU3EO4N7VV6NSBAYCRRIEQZTS4PBMIFEP6KJDXWSFM6"} 
|     -ERR 'Authorization Violation'
|   GetRequest: 
|     INFO {"server_id":"NCN74AT3XD6OBJIKFUN3TNNWT6Q3JNI4RUDLWYJPBMSNG4AVWU53MQSV","server_name":"NCN74AT3XD6OBJIKFUN3TNNWT6Q3JNI4RUDLWYJPBMSNG4AVWU53MQSV","version":"2.11.3","proto":1,"git_commit":"a82cfda","go":"go1.24.2","host":"0.0.0.0","port":4222,"headers":true,"auth_required":true,"max_payload":1048576,"jetstream":true,"client_id":18,"client_ip":"10.10.14.61","xkey":"XBXOTVN3PTCXANU3EO4N7VV6NSBAYCRRIEQZTS4PBMIFEP6KJDXWSFM6"} 
|     -ERR 'Authorization Violation'
|   HTTPOptions: 
|     INFO {"server_id":"NCN74AT3XD6OBJIKFUN3TNNWT6Q3JNI4RUDLWYJPBMSNG4AVWU53MQSV","server_name":"NCN74AT3XD6OBJIKFUN3TNNWT6Q3JNI4RUDLWYJPBMSNG4AVWU53MQSV","version":"2.11.3","proto":1,"git_commit":"a82cfda","go":"go1.24.2","host":"0.0.0.0","port":4222,"headers":true,"auth_required":true,"max_payload":1048576,"jetstream":true,"client_id":19,"client_ip":"10.10.14.61","xkey":"XBXOTVN3PTCXANU3EO4N7VV6NSBAYCRRIEQZTS4PBMIFEP6KJDXWSFM6"} 
|     -ERR 'Authorization Violation'
|   NULL: 
|     INFO {"server_id":"NCN74AT3XD6OBJIKFUN3TNNWT6Q3JNI4RUDLWYJPBMSNG4AVWU53MQSV","server_name":"NCN74AT3XD6OBJIKFUN3TNNWT6Q3JNI4RUDLWYJPBMSNG4AVWU53MQSV","version":"2.11.3","proto":1,"git_commit":"a82cfda","go":"go1.24.2","host":"0.0.0.0","port":4222,"headers":true,"auth_required":true,"max_payload":1048576,"jetstream":true,"client_id":16,"client_ip":"10.10.14.61","xkey":"XBXOTVN3PTCXANU3EO4N7VV6NSBAYCRRIEQZTS4PBMIFEP6KJDXWSFM6"} 
|_    -ERR 'Authentication Timeout'
5985/tcp open  http            Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (96%), Microsoft Windows Server 2016 (96%), Microsoft Windows 11 24H2 (96%), Microsoft Windows Server 2022 (95%), Microsoft Windows 11 24H2 - 25H2 (95%), Microsoft Windows Server 2012 R2 (93%), Microsoft Windows Server 2019 (92%), Microsoft Windows Server 2016 or Server 2019 (91%), Microsoft Windows Server 2012 (91%), Microsoft Windows 10 1703 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# System time
```python
❯❯❯ ntpdate dc01.mirage.htb                    
2026-10-10 23:11:35.843886 (+0100) +25199.759920 +/- 0.006475 dc01.mirage.htb 10.129.232.163 s1 no-leap
CLOCK: step_systime: Operation not permitted
```

So the DC is running at +7h

# SMB (445)

Null auth not really possible when only kerberos is enabled

Guest account is also disabled

# NFS (2049)

Looks like root escape is not possible here

```python
❯❯❯ nxc nfs dc01.mirage.htb --shares
NFS         10.129.232.163  2049   dc01.mirage.htb  [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.129.232.163  2049   dc01.mirage.htb  [*] Enumerating NFS Shares
NFS         10.129.232.163  2049   dc01.mirage.htb  UID        Perms    Storage Usage    Share                          Access List    
NFS         10.129.232.163  2049   dc01.mirage.htb  ---        -----    -------------    -----                          -----------    
NFS         10.129.232.163  2049   dc01.mirage.htb  4294967294 r--      16.3GB/19.8GB    /MirageReports
```

```python
❯❯❯ nxc nfs dc01.mirage.htb --share '/MirageReports' --ls '/'
NFS         10.129.232.163  2049   dc01.mirage.htb  [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.129.232.163  2049   dc01.mirage.htb  UID        Perms  File Size     File Path
NFS         10.129.232.163  2049   dc01.mirage.htb  ---        -----  ---------     ---------
NFS         10.129.232.163  2049   dc01.mirage.htb  4294967294 dr--   64.0B         /MirageReports/.
NFS         10.129.232.163  2049   dc01.mirage.htb  4294967294 dr--   64.0B         /MirageReports/..
NFS         10.129.232.163  2049   dc01.mirage.htb  4294967294 -r-x   8.1MB         /MirageReports/Incident_Report_Missing_DNS_Record_nats-svc.pdf
NFS         10.129.232.163  2049   dc01.mirage.htb  4294967294 -r-x   8.9MB         /MirageReports/Mirage_Authentication_Hardening_Report.pdf
```

Looks like some pdf files in the NFS share

```python
❯❯❯ nxc nfs dc01.mirage.htb --share '/MirageReports' --get-file Incident_Report_Missing_DNS_Record_nats-svc.pdf Incident_Report_Missing_DNS_Record_nats-svc.pdf
NFS         10.129.232.163  2049   dc01.mirage.htb  [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.129.232.163  2049   dc01.mirage.htb  [*] Downloading Incident_Report_Missing_DNS_Record_nats-svc.pdf to Incident_Report_Missing_DNS_Record_nats-svc.pdf
NFS         10.129.232.163  2049   dc01.mirage.htb  File successfully downloaded from Incident_Report_Missing_DNS_Record_nats-svc.pdf to Incident_Report_Missing_DNS_Record_nats-svc.pdf

❯❯❯ nxc nfs dc01.mirage.htb --share '/MirageReports' --get-file Mirage_Authentication_Hardening_Report.pdf Mirage_Authentication_Hardening_Report.pdf
NFS         10.129.232.163  2049   dc01.mirage.htb  [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.129.232.163  2049   dc01.mirage.htb  [*] Downloading Mirage_Authentication_Hardening_Report.pdf to Mirage_Authentication_Hardening_Report.pdf
NFS         10.129.232.163  2049   dc01.mirage.htb  File successfully downloaded from Mirage_Authentication_Hardening_Report.pdf to Mirage_Authentication_Hardening_Report.pdf
```

Ill download both of these files and have a look

```python
❯❯❯ nxc nfs dc01.mirage.htb --share '/MirageReports' --put-file test.txt test.txt 
NFS         10.129.232.163  2049   dc01.mirage.htb  [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.129.232.163  2049   dc01.mirage.htb  [*] Uploading from test.txt to test.txt
NFS         10.129.232.163  2049   dc01.mirage.htb  [*] Trying to create test.txt
NFS         10.129.232.163  2049   dc01.mirage.htb  [-] Error writing file to share test.txt: NFS3ERR_ROFS
```

I dont have the ability to write to the share

![](Pasted%20image%2020261010162407.png)

There is a report of a missing DNS record for the NATS service, which is running on port 4222

![658](Pasted%20image%2020261010162706.png)

Image above could be helpful for when and if i need to interact with NATS

Also a user `dev_account_a`

![639](Pasted%20image%2020261010162844.png)

![613](Pasted%20image%2020261010162915.png)

Some more potentially helpful info

![](Pasted%20image%2020261010163227.png)

The other report shows a potential user, but other than that basically just explains the disabling of NTLM, however its mentioned it was a gradual process, some systems may still allow NTLM

# NATS (4222)



