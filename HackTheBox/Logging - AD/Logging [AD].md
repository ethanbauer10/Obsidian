# Machine info
As is common in real life pentests, you will start the Logging box with credentials for the following account wallace.everette / Welcome2026@

```python
wallace.everette:Welcome2026@
```

# Host file setup
```python
sudo nxc smb 10.129.245.130 --generate-hosts-file /etc/hosts
SMB         10.129.245.130  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:logging.htb) (signing:True) (SMBv1:None) (Null Auth:True
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn dc01.logging.htb -vv

Nmap scan report for dc01.logging.htb (10.129.245.130)
Host is up, received user-set (0.014s latency).
rDNS record for 10.129.245.130: DC01.logging.htb
Scanned at 2026-10-06 17:06:25 BST for 10s
Not shown: 65505 closed tcp ports (conn-refused)
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
8530/tcp  open  unknown          syn-ack
8531/tcp  open  unknown          syn-ack
9389/tcp  open  adws             syn-ack
47001/tcp open  winrm            syn-ack
49664/tcp open  unknown          syn-ack
49665/tcp open  unknown          syn-ack
49666/tcp open  unknown          syn-ack
49667/tcp open  unknown          syn-ack
49673/tcp open  unknown          syn-ack
49696/tcp open  unknown          syn-ack
49697/tcp open  unknown          syn-ack
49698/tcp open  unknown          syn-ack
49705/tcp open  unknown          syn-ack
49742/tcp open  unknown          syn-ack
49752/tcp open  unknown          syn-ack
49802/tcp open  unknown          syn-ack
49832/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 10.79 seconds
```

## Nmap
```python

```