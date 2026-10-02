# Host file setup
```
sudo nxc smb 10.129.66.33 --generate-hosts-file /etc/hosts  
SMB         10.129.66.33    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:freelancer.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```
nmap -p- --min-rate=2000 -sT -Pn dc.freelancer.htb -vv

Nmap scan report for dc.freelancer.htb (10.129.66.33)
Host is up, received user-set (0.014s latency).
rDNS record for 10.129.66.33: DC.freelancer.htb
Scanned at 2026-10-02 16:54:50 BST for 11s
Not shown: 65509 closed tcp ports (conn-refused)
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
47001/tcp open  winrm            syn-ack
49664/tcp open  unknown          syn-ack
49665/tcp open  unknown          syn-ack
49666/tcp open  unknown          syn-ack
49667/tcp open  unknown          syn-ack
49671/tcp open  unknown          syn-ack
49680/tcp open  unknown          syn-ack
49681/tcp open  unknown          syn-ack
49684/tcp open  unknown          syn-ack
49689/tcp open  unknown          syn-ack
49711/tcp open  unknown          syn-ack
55297/tcp open  unknown          syn-ack
```

## Nmap
```

```

