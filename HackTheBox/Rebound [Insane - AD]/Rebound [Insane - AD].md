# Host file setup
```python
sudo nxc smb 10.129.14.17 --generate-hosts-file /etc/hosts                                     
[sudo] password for kali: 
SMB         10.129.14.17    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```
Few things note:
- Null auth is true
- SMB signing is enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.rebound.htb                                                        
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-04 14:53 +0100
Nmap scan report for dc01.rebound.htb (10.129.14.17)
Host is up (0.015s latency).
rDNS record for 10.129.14.17: DC01.rebound.htb
Not shown: 65509 closed tcp ports (conn-refused)
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
49667/tcp open  unknown
49673/tcp open  unknown
49690/tcp open  unknown
49691/tcp open  unknown
49696/tcp open  unknown
49709/tcp open  unknown
49724/tcp open  unknown
49743/tcp open  unknown
59390/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 13.68 seconds
```
## Nmap
```python

```