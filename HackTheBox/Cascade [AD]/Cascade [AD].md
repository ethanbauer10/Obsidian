# Host file setup
```python
sudo nxc smb 10.129.18.9 --generate-hosts-file /etc/hosts  
[sudo] password for kali: 
SMB         10.129.18.9     445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
```
- SMB signing is enabled
- Null auth enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn casc-dc1.cascade.local
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-12 17:11 +0100
Nmap scan report for casc-dc1.cascade.local (10.129.18.9)
Host is up (0.013s latency).
rDNS record for 10.129.18.9: CASC-DC1.cascade.local
Not shown: 65520 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49168/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.70 seconds
```

## Nmap
```python

```