# Host file setup
```python
sudo nxc smb 10.129.229.56 --generate-hosts-file /etc/hosts                                             
[sudo] password for kali: 
SMB         10.129.229.56   445    AUTHORITY        [*] Windows 10 / Server 2019 Build 17763 x64 (name:AUTHORITY) (domain:authority.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT authority.authority.htb
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-23 20:13 +0000
Nmap scan report for authority.authority.htb (10.129.229.56)
Host is up (0.014s latency).
rDNS record for 10.129.229.56: AUTHORITY.authority.htb
Not shown: 65507 closed tcp ports (conn-refused)
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
8443/tcp  open  https-alt
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49671/tcp open  unknown
49674/tcp open  unknown
49675/tcp open  unknown
49679/tcp open  unknown
49682/tcp open  unknown
49691/tcp open  unknown
49702/tcp open  unknown
61399/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 19.91 seconds
```

## Nmap
```python

```