# Machine info
As is common in real life pentests, you will start the PingPong box with credentials for the following account c.roberts / AssumedBreach123

```python
c.roberts:AssumedBreach123
```

# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.245.56 --generate-hosts-file /etc/hosts    

SMB         10.129.245.56   445    dc1              [*]  x64 (name:dc1) (domain:ping.htb) (signing:True) (SMBv1:None) (NTLM:False)
```

Since NTLM is disabled ill also generate a realm 

# System time
```python
❯❯❯ ntpdate dc1.ping.htb  
2026-11-22 02:29:19.158508 (+0000) +28799.610798 +/- 0.006356 dc1.ping.htb 10.129.245.56 s1 no-leap
CLOCK: step_systime: Operation not permitted
```

Its running at +8h

# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc1.ping.htb -vv

Nmap scan report for dc1.ping.htb (10.129.245.56)
Host is up, received user-set (0.012s latency).
Scanned at 2026-11-21 18:30:13 GMT for 65s
Not shown: 65515 filtered tcp ports (no-response)
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
2179/tcp  open  vmrdp            syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
49664/tcp open  unknown          syn-ack
49670/tcp open  unknown          syn-ack
51795/tcp open  unknown          syn-ack
64770/tcp open  unknown          syn-ack
64920/tcp open  unknown          syn-ack
64924/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 65.69 seconds
```

## Nmap
```python
❯❯❯ nmap -p 53,88,135,139,389,445,464,593,636,2179,3268,3269,5985,9389 -A --min-rate=2000 -sT -Pn dc1.ping.htb
Starting Nmap 7.991 ( https://nmap.org ) at 2026-11-21 18:33 +0000
Nmap scan report for dc1.ping.htb (10.129.245.56)
Host is up (0.013s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-11-22 02:33:10Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: ping.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc1.ping.htb, DNS:ping.htb, DNS:PING
| Not valid before: 2026-04-20T18:54:50
|_Not valid after:  2106-04-20T18:54:50
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: ping.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc1.ping.htb, DNS:ping.htb, DNS:PING
| Not valid before: 2026-04-20T18:54:50
|_Not valid after:  2106-04-20T18:54:50
|_ssl-date: TLS randomness does not represent time
2179/tcp open  vmrdp?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: ping.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc1.ping.htb, DNS:ping.htb, DNS:PING
| Not valid before: 2026-04-20T18:54:50
|_Not valid after:  2106-04-20T18:54:50
|_ssl-date: TLS randomness does not represent time
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: ping.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc1.ping.htb, DNS:ping.htb, DNS:PING
| Not valid before: 2026-04-20T18:54:50
|_Not valid after:  2106-04-20T18:54:50
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC1; OS: Windows; CPE: cpe:/o:microsoft:windows
```

2179 kinda tells me there is going to be some form of virtualisation, most likely an internal network

# SMB (445)

```python
❯❯❯ faketime -f +8h nxc smb dc1.ping.htb -u 'c.roberts' -p 'AssumedBreach123' -k
SMB         dc1.ping.htb    445    dc1              [*]  x64 (name:dc1) (domain:ping.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc1.ping.htb    445    dc1              [+] ping.htb\c.roberts:AssumedBreach123
```

The creds work on the domain!

```python
❯❯❯ faketime -f +8h nxc smb dc1.ping.htb -u 'c.roberts' -p 'AssumedBreach123' -k --users
SMB         dc1.ping.htb    445    dc1              [*]  x64 (name:dc1) (domain:ping.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc1.ping.htb    445    dc1              [+] ping.htb\c.roberts:AssumedBreach123 
SMB         dc1.ping.htb    445    dc1              -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         dc1.ping.htb    445    dc1              Administrator                 2026-02-03 09:42:36 0       Built-in account for administering the computer/domain 
SMB         dc1.ping.htb    445    dc1              Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         dc1.ping.htb    445    dc1              krbtgt                        2025-06-16 11:36:47 0       Key Distribution Center Service Account 
SMB         dc1.ping.htb    445    dc1              M.Russell                     2025-12-28 08:14:51 0       IT manager 
SMB         dc1.ping.htb    445    dc1              G.Lorents                     2025-12-28 08:14:51 0       IT Support Specialist 
SMB         dc1.ping.htb    445    dc1              Z.Iskam                       2025-12-28 08:14:51 0       IT Director 
SMB         dc1.ping.htb    445    dc1              M.Wallace                     2025-12-28 08:14:51 0       Assurance Tester 
SMB         dc1.ping.htb    445    dc1              P.Paul                        2025-12-28 08:14:52 0       Security Specialist 
SMB         dc1.ping.htb    445    dc1              A.Gordon                      2025-12-28 08:14:52 0       Network Engineer 
SMB         dc1.ping.htb    445    dc1              P.Klain                       2025-12-28 08:14:52 0       Senior IT Technician 
SMB         dc1.ping.htb    445    dc1              C.Roberts                     2025-12-28 08:14:52 0       Junior IT Technician 
SMB         dc1.ping.htb    445    dc1              S.Zhoski                      2026-01-17 14:42:20 0        
SMB         dc1.ping.htb    445    dc1              R.Miller                      2026-01-17 14:42:20 0        
SMB         dc1.ping.htb    445    dc1              A.Adam                        2026-01-17 14:42:20 0        
SMB         dc1.ping.htb    445    dc1              R.Robins                      2026-01-17 14:42:20 0        
SMB         dc1.ping.htb    445    dc1              P.Pollek                      2026-01-17 14:42:20 0        
SMB         dc1.ping.htb    445    dc1              O.Plinski                     2026-01-17 14:42:20 0        
SMB         dc1.ping.htb    445    dc1              V.Vodomir                     2026-01-17 14:42:21 0        
SMB         dc1.ping.htb    445    dc1              D.MacKenzie                   2026-01-17 14:42:21 0        
SMB         dc1.ping.htb    445    dc1              P.Vandeval                    2026-01-17 14:42:21 0        
SMB         dc1.ping.htb    445    dc1              T.Yang                        2026-01-17 14:42:21 0        
SMB         dc1.ping.htb    445    dc1              N.Neri                        2026-01-17 14:42:21 0        
SMB         dc1.ping.htb    445    dc1              W.Williams                    2026-01-17 14:42:21 0        
SMB         dc1.ping.htb    445    dc1              K.Podroski                    2026-01-17 14:42:21 0        
SMB         dc1.ping.htb    445    dc1              G.James                       2026-01-17 14:42:21 0        
SMB         dc1.ping.htb    445    dc1              [*] Enumerated 25 local users: PING
```

Ill get the users and the user account descriptions

```python
❯❯❯ faketime -f +8h nxc smb dc1.ping.htb -u 'c.roberts' -p 'AssumedBreach123' -k --rid-brute 20000 | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC1$
pong$
M.Russell
G.Lorents
Z.Iskam
M.Wallace
P.Paul
A.Gordon
P.Klain
C.Roberts
gMSA$
S.Zhoski
R.Miller
A.Adam
R.Robins
P.Pollek
O.Plinski
V.Vodomir
D.MacKenzie
P.Vandeval
T.Yang
N.Neri
W.Williams
K.Podroski
G.James
```

Ill use the rid brute flag since this also pulls machine accounts to get a user file

I also only have read on default shares

# Bloodhound collection

```python
❯❯❯ bloodyAD --host dc1.ping.htb -d ping.htb -k get bloodhound --transitive                  
Clock skew detected. Adjusting local time by 7:59:58.951478. Retrying operation.
[+] Connecting to LDAP server
Clock skew detected. Adjusting local time by 7:59:58.812825. Retrying operation.
[+] Connected to LDAP serrver
Dumping schema: 2it [00:00, 26.81it/s]
Generating lookuptable: 105it [00:00, 548.40it/s]
Dumping SDs:  99%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▍ | 110/111 [00:01<00:00, 61.98it/s]
Dumping domains: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 21.15it/s]
Dumping users: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 25/25 [00:00<00:00, 650.14it/s]
Dumping computers: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2/2 [00:00<00:00, 93.06it/s]
Dumping groups: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 53/53 [00:00<00:00, 1183.26it/s]
Dumping GPOs: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 3/3 [00:00<00:00, 168.87it/s]
Dumping OUs: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 66.65it/s]
Dumping Containers: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 19/19 [00:00<00:00, 353.97it/s]
[+] Bloodhound data saved to 20261121T184834_Bloodhound.zip
[+] Found 1 trusts
[+] Found trust pong.htb (S-1-5-21-2410575906-3092493790-2123333151)
[+] Connecting to pong.htb (follow_trusts)
[-] Failed to connect to pong.htb ([Errno -2] Name or service not known)
[-] If it's a connectivity issue (not auth issue), make sure the hostname resolves to an IP address (eg. modify your /etc/hosts)
```

From this output i can see bloodyAD detects a trust, but since the machine is likely internal i cannot reach it as of yet

