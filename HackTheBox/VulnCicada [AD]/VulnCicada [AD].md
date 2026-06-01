
# Host file setup

```python
sudo nxc smb 10.129.234.48 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.234.48   445    DC-JPQ225        [*]  x64 (name:DC-JPQ225) (domain:cicada.vl) (signing:True) (SMBv1:None) (NTLM:False)
```
- NTLM auth disabled
- SMB signing is enabled

# Enumeration

## Open ports

```python
nmap -p- --min-rate=2000 -sT DC-JPQ225.cicada.vl
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-01 18:22 +0100
Nmap scan report for DC-JPQ225.cicada.vl (10.129.234.48)
Host is up (0.013s latency).
Not shown: 65512 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
111/tcp   open  rpcbind
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
2049/tcp  open  nfs
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
9389/tcp  open  adws
49217/tcp open  unknown
49219/tcp open  unknown
49236/tcp open  unknown
49310/tcp open  unknown
49664/tcp open  unknown
49668/tcp open  unknown
49773/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.77 seconds
```

## Nmap

```python

```