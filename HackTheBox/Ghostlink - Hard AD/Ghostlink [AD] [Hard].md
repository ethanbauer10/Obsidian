# Host file setup
```python
sudo nxc smb 10.129.64.193 --generate-hosts-file /etc/hosts                                          
SMB         10.129.64.193   445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:ghostlink.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.129.64.193 -vv

Nmap scan report for 10.129.64.193
Host is up, received user-set (0.013s latency).
Scanned at 2026-09-30 15:36:00 BST for 98s
Not shown: 65511 filtered tcp ports (no-response)
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
1883/tcp  open  mqtt             syn-ack
2179/tcp  open  vmrdp            syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
49664/tcp open  unknown          syn-ack
49676/tcp open  unknown          syn-ack
49677/tcp open  unknown          syn-ack
49680/tcp open  unknown          syn-ack
49682/tcp open  unknown          syn-ack
49904/tcp open  unknown          syn-ack
49916/tcp open  unknown          syn-ack
57507/tcp open  unknown          syn-ack
```

## Nmap
```python

```

# SMB (445)

