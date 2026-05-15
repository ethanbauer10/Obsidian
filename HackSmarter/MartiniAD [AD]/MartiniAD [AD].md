# Objective and scope

An adult beverage company "Martini Bars" recently had a corporate breach and the compliance and risk team dictates they perform a penetration test at one of their branch offices. The Hack Smarter team has been authorized to perform an internal black box pentest.

The client has provided you with VPN access to their internal network, but no credentials.

# Host file setup

```python
sudo nxc smb 10.1.33.56 --generate-hosts-file /etc/hosts
[sudo] password for kali:  
SMB         10.1.33.56      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
```

- SMB Signing disabled
- Windows 11 server 2025

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT DC01.DRY.MARTINI.BARS                                                   
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-15 17:10 +0100
Nmap scan report for DC01.DRY.MARTINI.BARS (10.1.33.56)
Host is up (0.097s latency).
Not shown: 65522 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
3268/tcp  open  globalcatLDAP
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
49664/tcp open  unknown
49667/tcp open  unknown
49672/tcp open  unknown
49679/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.88 seconds
```
Ldap is also open but for some reason Nmap missed it!

## Nmap
```python

```