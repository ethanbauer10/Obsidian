# Host file setup
```python
sudo nxc smb 10.129.12.5 --generate-hosts-file /etc/hosts 
[sudo] password for kali: 
SMB         10.129.12.5     445    BLN01            [*] Windows Server 2008 R2 Datacenter 7601 Service Pack 1 x64 (name:BLN01) (domain:retro2.vl) (signing:True) (SMBv1:True) (Null Auth:True)
```
Added an entry into `/etc/hosts`

SMB signing is enabled

SMBv1 

Null auth enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.12.5 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-30 15:47 +0100
Nmap scan report for 10.129.12.5
Host is up (0.018s latency).
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
3389/tcp  open  ms-wbt-server
5722/tcp  open  msdfsr
9389/tcp  open  adws
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49165/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.29 seconds
```
## Nmap
```python

```