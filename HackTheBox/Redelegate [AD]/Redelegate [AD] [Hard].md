# Host file setup
```python
sudo nxc smb 10.129.234.50 --generate-hosts-file /etc/hosts                                  
[sudo] password for kali: 
SMB         10.129.234.50   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:redelegate.vl) (signing:True) (SMBv1:None) (Null Auth:True)
```
This has added an entry into /etc/hosts

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.234.50                                                            
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-22 15:37 +0000
Nmap scan report for 10.129.234.50
Host is up (0.016s latency).
Not shown: 65507 closed tcp ports (conn-refused)
PORT      STATE SERVICE
21/tcp    open  ftp
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
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49669/tcp open  unknown
61025/tcp open  unknown
64959/tcp open  unknown
64960/tcp open  unknown
64966/tcp open  unknown
64976/tcp open  unknown
64978/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 11.48 seconds
```

## Nmap
```python

```