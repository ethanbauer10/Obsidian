
# Initial Credentials

```python
rose:KxEPkKe6R8su
```

# Host file setup

```python
sudo nxc smb 10.129.232.128 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.232.128  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.232.128         
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-23 19:16 +0100
Nmap scan report for DC01.sequel.htb (10.129.232.128)
Host is up (0.013s latency).
Not shown: 65510 filtered tcp ports (no-response)
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
1433/tcp  open  ms-sql-s
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49668/tcp open  unknown
49693/tcp open  unknown
49694/tcp open  unknown
49695/tcp open  unknown
49700/tcp open  unknown
49724/tcp open  unknown
49733/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.80 seconds
```

## Nmap
```python

```

# SMB (445)

