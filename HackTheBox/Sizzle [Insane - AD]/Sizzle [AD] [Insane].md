# Host file setup
```python
sudo nxc smb 10.129.19.191 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.19.191   445    SIZZLE           [*] Windows 10 / Server 2016 Build 14393 x64 (name:SIZZLE) (domain:HTB.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
```
- SMB signing is enabled
- Null auth 

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.129.19.191                                                       
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-15 16:09 +0100
Nmap scan report for 10.129.19.191
Host is up (0.014s latency).
Not shown: 65507 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
53/tcp    open  domain
80/tcp    open  http
135/tcp   open  msrpc
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
5986/tcp  open  wsmans
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49668/tcp open  unknown
49673/tcp open  unknown
49688/tcp open  unknown
49689/tcp open  unknown
49692/tcp open  unknown
49695/tcp open  unknown
49710/tcp open  unknown
49720/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.19 seconds
```
