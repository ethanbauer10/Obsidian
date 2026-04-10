# Host file setup
```python
sudo nxc smb 10.129.16.242 --generate-hosts-file /etc/hosts 
SMB         10.129.16.242   445    MANTIS           [*] Windows Server 2008 R2 Standard 7601 Service Pack 1 x64 (name:MANTIS) (domain:htb.local) (signing:True) (SMBv1:True) (Null Auth:True)
```
- Null auth is enabled
- SMB signing is on 
- SMBv1
- Windows server 2008

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn mantis.htb.local                               
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-10 12:03 -0400
Nmap scan report for mantis.htb.local (10.129.16.242)
Host is up (0.014s latency).
rDNS record for 10.129.16.242: MANTIS.htb.local
Not shown: 65508 closed tcp ports (conn-refused)
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
1337/tcp  open  waste
1433/tcp  open  ms-sql-s
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5722/tcp  open  msdfsr
8080/tcp  open  http-proxy
9389/tcp  open  adws
47001/tcp open  winrm
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49167/tcp open  unknown
49170/tcp open  unknown
49174/tcp open  unknown
50255/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 17.09 seconds
```
## Nmap
```python

```