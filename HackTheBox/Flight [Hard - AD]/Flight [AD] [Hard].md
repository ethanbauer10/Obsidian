# Host file setup
```python
sudo nxc smb 10.129.228.120 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.228.120  445    G0               [*] Windows 10 / Server 2019 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```
SMB signing enabled

Null auth enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT g0.flight.htb -Pn
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-31 15:59 +0100
Nmap scan report for g0.flight.htb (10.129.228.120)
Host is up (0.014s latency).
rDNS record for 10.129.228.120: G0.flight.htb
Not shown: 65518 filtered tcp ports (no-response)
PORT      STATE SERVICE
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
9389/tcp  open  adws
49667/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49695/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.70 seconds
```

## Nmap
```python

```