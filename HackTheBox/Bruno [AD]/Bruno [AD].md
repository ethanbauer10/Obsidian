# Host file setup
```python
sudo nxc smb 10.129.238.9 --generate-hosts-file /etc/hosts            
[sudo] password for kali: 
SMB         10.129.238.9    445    BRUNODC          [*] Windows Server 2022 Build 20348 x64 (name:BRUNODC) (domain:bruno.vl) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.238.9          
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-27 18:11 +0000
Nmap scan report for 10.129.238.9
Host is up (0.015s latency).
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
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
3389/tcp  open  ms-wbt-server
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
56106/tcp open  unknown
56109/tcp open  unknown
63099/tcp open  unknown
63104/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.33 seconds
```

## Nmap
```python

```