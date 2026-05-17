# Host file setup

```python
sudo nxc smb 10.129.230.109 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.230.109  445    hathor           [*]  x64 (name:hathor) (domain:windcorp.htb) (signing:True) (SMBv1:None) (NTLM:False)
```
- NTLM auth is disabled!
- SMB signing is enabled

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT hathor.windcorp.htb  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-17 17:46 +0100
Nmap scan report for hathor.windcorp.htb (10.129.230.109)
Host is up (0.012s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49667/tcp open  unknown
54175/tcp open  unknown
61498/tcp open  unknown
61517/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.77 seconds
```

## Nmap
```python

```