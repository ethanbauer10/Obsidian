
# Objective and scope
You have been hired to perform an internal penetration test against the client's Active Directory environment. There is only one host in-scope (the Domain Controller). Your task is to get initial access and then perform lateral movement and privilege escalation until you have fully compromised the domain.

The client has provided you with VPN access to their environment, but no other information

# Host file setup
```python
sudo nxc smb 10.0.29.162 --generate-hosts-file /etc/hosts 
[sudo] password for kali: 
SMB         10.0.29.162     445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:fragments.local) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=1000 -sT dc01.fragments.local                                                            
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-03 16:42 +0100
Nmap scan report for dc01.fragments.local (10.0.29.162)
Host is up (0.097s latency).
rDNS record for 10.0.29.162: DC01.fragments.local
Not shown: 65521 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
111/tcp   open  rpcbind
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
2049/tcp  open  nfs
3389/tcp  open  ms-wbt-server
49684/tcp open  unknown
49688/tcp open  unknown
49719/tcp open  unknown
49753/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 131.83 seconds
```

Port 88, 389 and 5985 are also open
## Nmap
```python

```
