# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.230.179 --generate-hosts-file /etc/hosts        
 
SMB         10.129.230.179  445    DC-ANALYSIS      [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC-ANALYSIS) (domain:analysis.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc-analysis.analysis.htb -vv 

Host is up, received user-set (0.014s latency).
rDNS record for 10.129.230.179: DC-ANALYSIS.analysis.htb
Scanned at 2026-11-11 17:42:36 GMT for 12s
Not shown: 65506 closed tcp ports (conn-refused)
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
3306/tcp  open  mysql            syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
33060/tcp open  mysqlx           syn-ack
47001/tcp open  winrm            syn-ack
49664/tcp open  unknown          syn-ack
49665/tcp open  unknown          syn-ack
49666/tcp open  unknown          syn-ack
49667/tcp open  unknown          syn-ack
49671/tcp open  unknown          syn-ack
49678/tcp open  unknown          syn-ack
49679/tcp open  unknown          syn-ack
49682/tcp open  unknown          syn-ack
49691/tcp open  unknown          syn-ack
49702/tcp open  unknown          syn-ack
49715/tcp open  unknown          syn-ack
49717/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 13.13 seconds
```

## Nmap
```python

```