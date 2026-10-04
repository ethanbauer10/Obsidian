# Host file setup
```python
sudo nxc smb 10.129.232.99 --generate-hosts-file /etc/hosts 
SMB         10.129.232.99   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:infiltrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn dc01.infiltrator.htb -vv

Nmap scan report for dc01.infiltrator.htb (10.129.232.99)
Host is up, received user-set (0.015s latency).
rDNS record for 10.129.232.99: DC01.infiltrator.htb
Scanned at 2026-10-04 16:17:05 BST for 65s
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
80/tcp    open  http             syn-ack
88/tcp    open  kerberos-sec     syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
389/tcp   open  ldap             syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
3389/tcp  open  ms-wbt-server    syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
15220/tcp open  unknown          syn-ack
49667/tcp open  unknown          syn-ack
49694/tcp open  unknown          syn-ack
49695/tcp open  unknown          syn-ack
49700/tcp open  unknown          syn-ack
49726/tcp open  unknown          syn-ack
49749/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 66.20 seconds
```

## Nmap
```python

```