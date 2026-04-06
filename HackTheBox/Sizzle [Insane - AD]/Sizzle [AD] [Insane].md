# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT sizzle.htb.local
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-06 11:06 -0400
Nmap scan report for sizzle.htb.local (10.129.15.12)
Host is up (0.014s latency).
rDNS record for 10.129.15.12: SIZZLE.HTB.LOCAL
Not shown: 65507 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
53/tcp    open  domain
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
443/tcp   open  https
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
5986/tcp  open  wsmans
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49668/tcp open  unknown
49671/tcp open  unknown
49687/tcp open  unknown
49688/tcp open  unknown
49692/tcp open  unknown
49695/tcp open  unknown
49710/tcp open  unknown
49730/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.79 seconds
```
Something interesting is kerberos is filtered
## Nmap
```python

```