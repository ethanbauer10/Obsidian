# Host file setup
```python
sudo nxc smb 10.129.20.49 --generate-hosts-file /etc/hosts 
[sudo] password for kali: 
SMB         10.129.20.49    445    DC1              [*]  x64 (name:DC1) (domain:scrm.local) (signing:True) (SMBv1:None) (NTLM:False)
```
- NTLM auth is disabled
- SMB signing is enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc1.scrm.local                                                                 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-16 16:18 +0100
Nmap scan report for dc1.scrm.local (10.129.20.49)
Host is up (0.013s latency).
rDNS record for 10.129.20.49: DC1.scrm.local
Not shown: 65514 filtered tcp ports (no-response)
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
1433/tcp  open  ms-sql-s
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
4411/tcp  open  found
5985/tcp  open  wsman
9389/tcp  open  adws
49667/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49701/tcp open  unknown
49712/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.79 seconds
```
MS-SQL is open

## Nmap
```python

```