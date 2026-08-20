
# Machine info
As is common in real life pentests, you will start the DarkZero box with credentials for the following account john.w / RFulUtONCOL!

```python
john.w:RFulUtONCOL!
```

# Host file setup
```python
sudo nxc smb 10.129.48.21 --generate-hosts-file /etc/hosts 
[sudo] password for kali: 
SMB         10.129.48.21    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:darkzero.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.darkzero.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-20 17:09 +0100
Nmap scan report for dc01.darkzero.htb (10.129.48.21)
Host is up (0.014s latency).
rDNS record for 10.129.48.21: DC01.darkzero.htb
Not shown: 65513 filtered tcp ports (no-response)
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
2179/tcp  open  vmrdp
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49666/tcp open  unknown
49676/tcp open  unknown
49677/tcp open  unknown
49897/tcp open  unknown
49926/tcp open  unknown
53177/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.26 seconds
```

## Nmap
```python

```