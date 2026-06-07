
# Host file setup
```python
sudo nxc smb 10.129.11.223 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.11.223   445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:haze.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.haze.htb                                                                                                                                   
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-07 16:31 +0100
Nmap scan report for dc01.haze.htb (10.129.11.223)
Host is up (0.013s latency).
rDNS record for 10.129.11.223: DC01.haze.htb
Not shown: 65505 closed tcp ports (conn-refused)
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
5985/tcp  open  wsman
8000/tcp  open  http-alt
8088/tcp  open  radan-http
8089/tcp  open  unknown
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49674/tcp open  unknown
49683/tcp open  unknown
49684/tcp open  unknown
64858/tcp open  unknown
64865/tcp open  unknown
64868/tcp open  unknown
64883/tcp open  unknown
64918/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 11.22 seconds
```

## Nmap
```python

```