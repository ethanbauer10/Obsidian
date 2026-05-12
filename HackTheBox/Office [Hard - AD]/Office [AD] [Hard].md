# Host file setup

```python
sudo nxc smb 10.129.230.226 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.230.226  445    DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:office.htb) (signing:True) (SMBv1:None)
```

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT dc.office.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-12 16:59 +0100
Nmap scan report for dc.office.htb (10.129.230.226)
Host is up (0.014s latency).
rDNS record for 10.129.230.226: DC.office.htb
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
139/tcp   open  netbios-ssn
389/tcp   open  ldap
443/tcp   open  https
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
59909/tcp open  unknown
59917/tcp open  unknown
59944/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.35 seconds
```

## Nmap
```python

```