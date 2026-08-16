# Host file setup
```python
sudo nxc smb 10.129.232.60 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.232.60   445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc.absolute.htb                                                           
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-16 15:26 +0100
Nmap scan report for dc.absolute.htb (10.129.232.60)
Host is up (0.012s latency).
rDNS record for 10.129.232.60: DC.absolute.htb
Not shown: 65509 closed tcp ports (conn-refused)
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
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49677/tcp open  unknown
49690/tcp open  unknown
49691/tcp open  unknown
49695/tcp open  unknown
49712/tcp open  unknown
49715/tcp open  unknown
51753/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 11.52 seconds
```

## Nmap
```python

```