# Host file setup
```python
sudo nxc smb 10.129.242.196 --generate-hosts-file /etc/hosts                                         
[sudo] password for kali: 
SMB         10.129.242.196  445    dc               [*]  x64 (name:dc) (domain:hercules.htb) (signing:True) (SMBv1:None) (NTLM:False)
```

NTLM auth is disabled 

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn dc.hercules.htb -vv

Nmap scan report for dc.hercules.htb (10.129.242.196)
Host is up, received user-set (0.014s latency).
Scanned at 2026-09-11 17:52:34 BST for 66s
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
80/tcp    open  http             syn-ack
88/tcp    open  kerberos-sec     syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
389/tcp   open  ldap             syn-ack
443/tcp   open  https            syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5986/tcp  open  wsmans           syn-ack
9389/tcp  open  adws             syn-ack
49664/tcp open  unknown          syn-ack
49668/tcp open  unknown          syn-ack
51753/tcp open  unknown          syn-ack
56289/tcp open  unknown          syn-ack
56313/tcp open  unknown          syn-ack
57888/tcp open  unknown          syn-ack
57897/tcp open  unknown          syn-ack
```

## Nmap
```python

```