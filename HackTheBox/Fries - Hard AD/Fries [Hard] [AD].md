# Machine Information
Please allow up to 7 minutes for services to load. As is common in real life Windows penetration tests, you will start the Fries box with credentials for the following account : d.cooper@fries.htb / D4LE11maan!!

```python
d.cooper:D4LE11maan!!
```

# Host file setup
```python
sudo nxc smb 10.129.244.72 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.244.72   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:fries.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.fries.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-17 18:40 +0100
Nmap scan report for dc01.fries.htb (10.129.244.72)
Host is up (0.015s latency).
rDNS record for 10.129.244.72: DC01.fries.htb
Not shown: 65510 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
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
2179/tcp  open  vmrdp
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49667/tcp open  unknown
49681/tcp open  unknown
49682/tcp open  unknown
49684/tcp open  unknown
49693/tcp open  unknown
49917/tcp open  unknown
49934/tcp open  unknown
49975/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.32 seconds
```

## Nmap
```python

```