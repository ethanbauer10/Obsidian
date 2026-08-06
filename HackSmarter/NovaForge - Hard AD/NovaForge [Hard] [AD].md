
# Objective and scope
NovaForge delivers enterprise software, intelligent automation, and internal management platforms targeted towards mid-sized companies. They have hired Hack Smarter to perform an internal network penetration test, with a special focus on Active Directory.

Your task is to enumerate the network and see if you can fully compromise the domain.

The client has provided you with VPN access to their network, but no credentials.

# Host file setup
```python
sudo nxc smb 10.0.0.100 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.0.0.100      445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:novaforge.local) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## DC
### Open ports
```python
nmap -p- --min-rate=2000 -sT dc.novaforge.local                                                   
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-06 16:26 +0100
Nmap scan report for dc.novaforge.local (10.0.0.100)
Host is up (0.094s latency).
rDNS record for 10.0.0.100: DC.novaforge.local
Not shown: 65504 closed tcp ports (conn-refused)
PORT      STATE SERVICE
25/tcp    open  smtp
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
110/tcp   open  pop3
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
143/tcp   open  imap
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
587/tcp   open  submission
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49672/tcp open  unknown
49674/tcp open  unknown
49693/tcp open  unknown
49697/tcp open  unknown
49709/tcp open  unknown
49714/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 33.62 seconds
```

### Nmap
```python

```

## Storage server
### Open ports
```python

```

### Nmap
```python

```