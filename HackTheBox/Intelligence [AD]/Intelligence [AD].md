# Host file setup
```python
sudo nxc smb 10.129.95.154 --generate-hosts-file /etc/hosts                                                                  
[sudo] password for kali: 
SMB         10.129.95.154   445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```
- SMB signing is enabled
- Null auth enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc.intelligence.htb   
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-14 15:29 +0100
Nmap scan report for dc.intelligence.htb (10.129.95.154)
Host is up (0.015s latency).
rDNS record for 10.129.95.154: DC.intelligence.htb
Not shown: 65517 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
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
9389/tcp  open  adws
49667/tcp open  unknown
49691/tcp open  unknown
49692/tcp open  unknown
49711/tcp open  unknown
49717/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.79 seconds
```
## Nmap
```python

```