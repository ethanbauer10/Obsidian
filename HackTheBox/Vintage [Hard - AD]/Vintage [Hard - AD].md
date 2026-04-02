# Initial credentials
```python
P.Rosa:Rosaisbest123
```

# Host file setup
```python
sudo nxc smb 10.129.231.205 --generate-hosts-file /etc/hosts        
[sudo] password for kali: 
SMB         10.129.231.205  445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
```
So some things to note:
- NTLM auth disabled
- SMB signing is enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.vintage.htb      
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 16:28 +0100
Nmap scan report for dc01.vintage.htb (10.129.231.205)
Host is up (0.014s latency).
Not shown: 65516 filtered tcp ports (no-response)
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
49664/tcp open  unknown
49668/tcp open  unknown
49676/tcp open  unknown
49687/tcp open  unknown
52792/tcp open  unknown
62928/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.80 seconds
```

## Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT dc01.vintage.htb
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 16:30 +0100
Nmap scan report for dc01.vintage.htb (10.129.231.205)
Host is up (0.017s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-02 15:30:36Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: vintage.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: vintage.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```
DC is running at -1h

# SMB (445)
Since NTLM is disabled i basically need to get a tgt

```python
nxc smb dc01.vintage.htb -u 'p.rosa' -p 'Rosaisbest123' -k            
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [+] vintage.htb\p.rosa:Rosaisbest123
```
The creds work to make this simple though ill just generate a tgt for this save entering credentials

```python
nxc smb dc01.vintage.htb -u 'p.rosa' -p 'Rosaisbest123' -k --generate-tgt p_rosa.tgt
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [+] vintage.htb\p.rosa:Rosaisbest123 
SMB         dc01.vintage.htb 445    dc01             [+] TGT saved to: p_rosa.tgt.ccache
SMB         dc01.vintage.htb 445    dc01             [+] Run the following command to use the TGT: export KRB5CCNAME=p_rosa.tgt.ccache
```
Now i can export it!

```python
export KRB5CCNAME=p_rosa.tgt.ccache
```
## Shares
```python
nxc smb dc01.vintage.htb --use-kcache --shares                                      
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [+] VINTAGE.HTB\p.rosa from ccache 
SMB         dc01.vintage.htb 445    dc01             [*] Enumerated shares
SMB         dc01.vintage.htb 445    dc01             Share           Permissions     Remark
SMB         dc01.vintage.htb 445    dc01             -----           -----------     ------
SMB         dc01.vintage.htb 445    dc01             ADMIN$                          Remote Admin
SMB         dc01.vintage.htb 445    dc01             C$                              Default share
SMB         dc01.vintage.htb 445    dc01             IPC$            READ            Remote IPC
SMB         dc01.vintage.htb 445    dc01             NETLOGON        READ            Logon server share 
SMB         dc01.vintage.htb 445    dc01             SYSVOL          READ            Logon server share
```
Read access on default shares
## Users
```python
nxc smb dc01.vintage.htb --use-kcache --users-export users.txt
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [+] VINTAGE.HTB\p.rosa from ccache 
SMB         dc01.vintage.htb 445    dc01             -Username-                    -Last PW Set-       -BadPW- -Description-
SMB         dc01.vintage.htb 445    dc01             Administrator                 2024-06-08 11:34:54 0       Built-in account for administering the computer/domain
SMB         dc01.vintage.htb 445    dc01             Guest                         2024-11-13 14:16:53 0       Built-in account for guest access to the computer/domain
SMB         dc01.vintage.htb 445    dc01             krbtgt                        2024-06-05 10:27:35 0       Key Distribution Center Service Account
SMB         dc01.vintage.htb 445    dc01             M.Rossi                       2024-06-05 13:31:08 0        
SMB         dc01.vintage.htb 445    dc01             R.Verdi                       2024-06-05 13:31:08 0        
SMB         dc01.vintage.htb 445    dc01             L.Bianchi                     2024-06-05 13:31:08 0        
SMB         dc01.vintage.htb 445    dc01             G.Viola                       2024-06-05 13:31:08 0        
SMB         dc01.vintage.htb 445    dc01             C.Neri                        2024-06-05 21:08:13 0        
SMB         dc01.vintage.htb 445    dc01             P.Rosa                        2024-11-06 12:27:16 0        
SMB         dc01.vintage.htb 445    dc01             svc_sql                       2026-04-02 15:32:15 0        
SMB         dc01.vintage.htb 445    dc01             svc_ldap                      2024-06-06 13:45:27 0        
SMB         dc01.vintage.htb 445    dc01             svc_ark                       2024-06-06 13:45:27 0        
SMB         dc01.vintage.htb 445    dc01             C.Neri_adm                    2024-06-07 10:54:14 0        
SMB         dc01.vintage.htb 445    dc01             L.Bianchi_adm                 2024-11-26 11:40:30 0        
SMB         dc01.vintage.htb 445    dc01             [*] Enumerated 14 local users: VINTAGE
SMB         dc01.vintage.htb 445    dc01             [*] Writing 14 local users to users.txt
```
Dumped all the users to a text file

AS-REP roasting and kerberoasting both failed

No usernames as passwords and no password reuse

Not a lot else i can do with SMB

# Pre-created computer account
```python

```