# Starting credentials
```python
j.fleischman:J0elTHEM4n1990!
```

# Host file setup
```python
sudo nxc smb 10.129.232.88 --generate-hosts-file /etc/hosts                 
[sudo] password for kali: 
SMB         10.129.232.88   445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.232.88                                 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-24 18:05 +0000
Nmap scan report for 10.129.232.88
Host is up (0.014s latency).
Not shown: 65517 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
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
49667/tcp open  unknown
49689/tcp open  unknown
49690/tcp open  unknown
49697/tcp open  unknown
49707/tcp open  unknown
49720/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.31 seconds
```

## Nmap
```python

```