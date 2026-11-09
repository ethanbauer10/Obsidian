# Machine info
As is common in real life pentests, you will start the Garfield box with credentials for the following account j.arbuckle / Th1sD4mnC4t!@1978

```python
j.arbuckle:Th1sD4mnC4t!@1978
```

# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.86.27 --generate-hosts-file /etc/hosts

SMB         10.129.86.27    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:garfield.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc01.garfield.htb -vv

Nmap scan report for dc01.garfield.htb (10.129.86.27)
Host is up, received user-set (0.013s latency).
rDNS record for 10.129.86.27: DC01.garfield.htb
Scanned at 2026-11-09 17:30:55 GMT for 66s
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
88/tcp    open  kerberos-sec     syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
389/tcp   open  ldap             syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
2179/tcp  open  vmrdp            syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
3389/tcp  open  ms-wbt-server    syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
49667/tcp open  unknown          syn-ack
49674/tcp open  unknown          syn-ack
49675/tcp open  unknown          syn-ack
49677/tcp open  unknown          syn-ack
49678/tcp open  unknown          syn-ack
49889/tcp open  unknown          syn-ack
49920/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 66.21 seconds
```

## Nmap
```python

```