# Host file setup
```python

```
# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.7.67 -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-20 15:32 GMT
Nmap scan report for 10.129.7.67
Host is up (0.013s latency).
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
1433/tcp  open  ms-sql-s
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49669/tcp open  unknown
49691/tcp open  unknown
49692/tcp open  unknown
49704/tcp open  unknown
49714/tcp open  unknown
49742/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.71 seconds
```
## Nmap
```python

```

