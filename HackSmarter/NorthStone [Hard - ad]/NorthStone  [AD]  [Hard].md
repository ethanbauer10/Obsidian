# Objective and Scope
North Stone, a premier luxury real estate marketplace, has engaged Hack Smarter for a full-scope penetration test. Your objective: identify critical vulnerabilities and demonstrate real-world business impact by achieving complete domain compromise.

The client has provided you with VPN access to their network, but no credentials.

I have also been provided a wordlist to use!

# Host file setup

```python
sudo nxc smb 10.1.209.181 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.1.209.181    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:northstone.local) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT dc.northstone.local                                        
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-29 17:11 +0100
Nmap scan report for dc.northstone.local (10.1.209.181)
Host is up (0.097s latency).
rDNS record for 10.1.209.181: DC.northstone.local
Not shown: 65519 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
47001/tcp open  winrm
49669/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
49679/tcp open  unknown
55203/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.37 seconds
```

After another check, port `88` and port `5985` are also open for some reason nmap missed it!
## Nmap
```python

```