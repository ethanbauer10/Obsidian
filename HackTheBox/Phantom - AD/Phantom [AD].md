
# Machine info
Should you need to crack a hash, use a short custom wordlist based on company name and simple mutation rules commonly seen in real life passwords (e.g. year and a special character).

# Host file setup
```python
sudo nxc smb 10.129.234.63 --generate-hosts-file /etc/hosts
SMB         10.129.234.63   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:phantom.vl) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc.phantom.vl                                                                    
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-15 15:54 +0100
Nmap scan report for dc.phantom.vl (10.129.234.63)
Host is up (0.013s latency).
rDNS record for 10.129.234.63: DC.phantom.vl
Not shown: 65514 filtered tcp ports (no-response)
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
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49667/tcp open  unknown
49675/tcp open  unknown
49676/tcp open  unknown
49683/tcp open  unknown
56328/tcp open  unknown
56353/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.28 seconds
```

## Nmap
```python

```