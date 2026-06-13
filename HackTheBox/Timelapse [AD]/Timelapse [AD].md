
# Host file setup
```python
[Jun 13, 2026 - 17:56:48 (BST)] exegol-CTF timelapse # nxc smb 10.129.227.113 --generate-hosts-file /etc/hosts 
SMB         10.129.227.113  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:timelapse.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
# nmap -p- --min-rate=2000 -sT dc01.timelapse.htb                                                                     
Starting Nmap 7.93 ( https://nmap.org ) at 2026-06-13 17:58 BST
Nmap scan report for dc01.timelapse.htb (10.129.227.113)
Host is up (0.013s latency).
rDNS record for 10.129.227.113: DC01.timelapse.htb
Not shown: 65517 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
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
5986/tcp  open  wsmans
9389/tcp  open  adws
49667/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49693/tcp open  unknown
49727/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.77 seconds
```

## Nmap
```python

```
