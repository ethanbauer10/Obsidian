# Host file setup
```python
sudo nxc smb 10.129.242.166 --generate-hosts-file /etc/hosts               
[sudo] password for kali: 
SMB         10.129.242.166  445    LUS2DC           [*]  x64 (name:LUS2DC) (domain:Lustrous2.vl) (signing:True) (SMBv1:None) (NTLM:False)
```

NTLM auth is disabled, which means ill have to work with kerberos

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT lus2dc.lustrous2.vl                                              
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-12 16:15 +0100
Nmap scan report for lus2dc.lustrous2.vl (10.129.242.166)
Host is up (0.013s latency).
rDNS record for 10.129.242.166: LUS2DC.Lustrous2.vl
Not shown: 65511 filtered tcp ports (no-response)
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
49664/tcp open  unknown
49667/tcp open  unknown
49779/tcp open  unknown
60285/tcp open  unknown
60287/tcp open  unknown
60304/tcp open  unknown
61989/tcp open  unknown
61990/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.29 seconds
```

## Nmap
```python

```
