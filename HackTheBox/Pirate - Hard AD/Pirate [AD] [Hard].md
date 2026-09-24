# Machine Description
As is common in real life pentests, you will start the Pirate box with credentials for the following account pentest / p3nt3st2025!&

```python
pentest:p3nt3st2025!&
```

# Host file setup
```python
sudo nxc smb 10.129.244.95 --generate-hosts-file /etc/hosts  
SMB         10.129.244.95   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:pirate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.129.244.95 -vv

Nmap scan report for 10.129.244.95
Host is up, received user-set (0.014s latency).
Scanned at 2026-09-24 16:27:34 BST for 66s
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
2179/tcp  open  vmrdp            syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
49667/tcp open  unknown          syn-ack
49683/tcp open  unknown          syn-ack
49684/tcp open  unknown          syn-ack
49686/tcp open  unknown          syn-ack
49689/tcp open  unknown          syn-ack
49913/tcp open  unknown          syn-ack
49936/tcp open  unknown          syn-ack
```

## Nmap
```python

```