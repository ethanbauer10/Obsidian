
# Host file setup

```python
sudo nxc smb 10.129.229.57 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.229.57   445    RESEARCH         [*] Windows 10 / Server 2019 Build 17763 x64 (name:RESEARCH) (domain:search.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```
- Windows version tells me this is going to be vulnerable to ZeroLogon

# Enumeration

## Open ports

```python
nmap -p- --min-rate=2000 -sT research.search.htb                                                              
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-19 18:14 +0100
Nmap scan report for research.search.htb (10.129.229.57)
Host is up (0.013s latency).
rDNS record for 10.129.229.57: RESEARCH.search.htb
Not shown: 65514 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
443/tcp   open  https
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
8172/tcp  open  unknown
9389/tcp  open  adws
49667/tcp open  unknown
49693/tcp open  unknown
49694/tcp open  unknown
49710/tcp open  unknown
49719/tcp open  unknown
49745/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.28 seconds
```

## Nmap

```python

```