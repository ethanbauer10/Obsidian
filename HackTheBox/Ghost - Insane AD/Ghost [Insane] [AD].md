# Host file setup
```python
sudo nxc smb 10.129.231.105 --generate-hosts-file /etc/hosts
 
SMB         10.129.231.105  445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:ghost.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.ghost.htb        
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-25 16:40 +0100
Nmap scan report for dc01.ghost.htb (10.129.231.105)
Host is up (0.013s latency).
rDNS record for 10.129.231.105: DC01.ghost.htb
Not shown: 65508 filtered tcp ports (no-response)
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
1433/tcp  open  ms-sql-s
2179/tcp  open  vmrdp
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
8008/tcp  open  http
8443/tcp  open  https-alt
9389/tcp  open  adws
49443/tcp open  unknown
49664/tcp open  unknown
49672/tcp open  unknown
49681/tcp open  unknown
63211/tcp open  unknown
63351/tcp open  unknown
64506/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.27 seconds
```

## Nmap
```python

```