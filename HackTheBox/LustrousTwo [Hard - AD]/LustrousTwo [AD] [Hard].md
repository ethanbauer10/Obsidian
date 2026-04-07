# Host file setup
```python
sudo nxc smb 10.129.242.166 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.242.166  445    LUS2DC           [*]  x64 (name:LUS2DC) (domain:Lustrous2.vl) (signing:True) (SMBv1:None) (NTLM:False)
```
- NTLM auth disabled
- SMB signing is enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT lus2dc.lustrous2.vl                                                       
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-07 10:24 -0400
Nmap scan report for lus2dc.lustrous2.vl (10.129.242.166)
Host is up (0.014s latency).
rDNS record for 10.129.242.166: LUS2DC.Lustrous2.vl
Not shown: 65512 filtered tcp ports (no-response)
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
9389/tcp  open  adws
49664/tcp open  unknown
49667/tcp open  unknown
49670/tcp open  unknown
49673/tcp open  unknown
49675/tcp open  unknown
49676/tcp open  unknown
49690/tcp open  unknown
49695/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.77 seconds
```

## Nmap
```python

```