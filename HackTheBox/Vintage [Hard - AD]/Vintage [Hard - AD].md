# Host file setup
```python
sudo nxc smb 10.129.231.205 --generate-hosts-file /etc/hosts        
[sudo] password for kali: 
SMB         10.129.231.205  445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
```
So some things to note:
- NTLM auth disabled
- SMB signing is enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.vintage.htb      
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 16:28 +0100
Nmap scan report for dc01.vintage.htb (10.129.231.205)
Host is up (0.014s latency).
Not shown: 65516 filtered tcp ports (no-response)
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
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
49676/tcp open  unknown
49687/tcp open  unknown
52792/tcp open  unknown
62928/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.80 seconds
```
## Nmap
```python

```