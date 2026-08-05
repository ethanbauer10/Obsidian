# Objective and scope
You have been engaged to conduct an internal penetration test targeting a domain-joined Linux server and its underlying Domain Controller. The client wants to evaluate the true impact and potential blast radius if an unauthenticated attacker breaches the internal network.

The client has provided you with VPN access to their environment, but no other information.

# Host file setup
```python
sudo nxc smb 10.0.31.82 --generate-hosts-file /etc/hosts 
[sudo] password for kali: 
SMB         10.0.31.82      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:casper.hsm) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumration
## Domain Controller
### Open ports
```python
nmap -p- -sT dc01.casper.hsm

Nmap scan report for dc01.casper.hsm (10.0.31.82)
Host is up (0.096s latency).
rDNS record for 10.0.31.82: DC01.casper.hsm
Not shown: 65512 filtered tcp ports (no-response)
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
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49666/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
49691/tcp open  unknown
49702/tcp open  unknown
49744/tcp open  unknown
54868/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 929.00 seconds
```

### Nmap
```python

```

## Linux Server
### Open ports
```python

```

### Nmap
```python

```