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