# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.245.51 --generate-hosts-file /etc/hosts 
SMB         10.129.245.51   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:certificate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumerationm
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc01.certificate.htb -vv

Nmap scan report for dc01.certificate.htb (10.129.245.51)
Host is up, received user-set (0.014s latency).
rDNS record for 10.129.245.51: DC01.certificate.htb
Scanned at 2026-11-03 18:01:27 GMT for 66s
Not shown: 65515 filtered tcp ports (no-response)
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
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
49666/tcp open  unknown          syn-ack
49693/tcp open  unknown          syn-ack
49695/tcp open  unknown          syn-ack
49697/tcp open  unknown          syn-ack
49716/tcp open  unknown          syn-ack
56747/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 66.20 seconds
```

## Nmap
```python

```

# SMB (445)

Null auth enabled by default with DCs however cannot use it to enumerate

Th