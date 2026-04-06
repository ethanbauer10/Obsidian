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

```