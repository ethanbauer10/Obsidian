# Host file setup
```python
sudo nxc smb 10.129.96.155 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.96.155   445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True) (Null Auth:True)
```
- SMB signing enabled
- Null auth enabled
- Vulnerable windows version to priv esc attacks like `noPac` and `ZeroLogon`

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT resolute.megabank.local                                                                 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-15 17:41 +0100
Nmap scan report for resolute.megabank.local (10.129.96.155)
Host is up (0.016s latency).
rDNS record for 10.129.96.155: RESOLUTE.megabank.local
Not shown: 65511 closed tcp ports (conn-refused)
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
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49668/tcp open  unknown
49671/tcp open  unknown
49676/tcp open  unknown
49677/tcp open  unknown
49686/tcp open  unknown
49710/tcp open  unknown
49845/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 12.51 seconds
```
## Nmap
```python

```
