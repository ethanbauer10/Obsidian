# Machine info
As is common in real life pentests, you will start the Checkpoint box with credentials for the following account alex.turner / Checkpoint2024!

```python
alex.turner:Checkpoint2024!
```

# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.80.26 --generate-hosts-file /etc/hosts    
SMB         10.129.80.26    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:checkpoint.htb) (signing:True) (SMBv1:None)
```

# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc01.checkpoint.htb -vv

Nmap scan report for dc01.checkpoint.htb (10.129.80.26)
Host is up, received user-set (0.014s latency).
rDNS record for 10.129.80.26: DC01.checkpoint.htb
Scanned at 2026-10-26 16:48:48 GMT for 98s
Not shown: 65514 filtered tcp ports (no-response)
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
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
49664/tcp open  unknown          syn-ack
49672/tcp open  unknown          syn-ack
49674/tcp open  unknown          syn-ack
49679/tcp open  unknown          syn-ack
49680/tcp open  unknown          syn-ack
49683/tcp open  unknown          syn-ack
49710/tcp open  unknown          syn-ack
49718/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 98.98 seconds
```

## Nmap
```python

```

# SMB (445)

Null auth disabled

Guest account is disabled

But i can try the provided credentials

```python

```