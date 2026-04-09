# Host file setup
```python
sudo nxc smb 10.129.16.113 --generate-hosts-file /etc/hosts                                                 
SMB         10.129.16.113   445    REEL             [*] Windows Server 2012 R2 Standard 9600 x64 (name:REEL) (domain:HTB.LOCAL) (signing:True) (SMBv1:True) (Null Auth:True)
```
- Null auth is enabled
- SMBv1
- SMB signing enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn reel.htb.local         
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-09 11:18 -0400
Nmap scan report for reel.htb.local (10.129.16.113)
Host is up (0.014s latency).
rDNS record for 10.129.16.113: REEL.HTB.LOCAL
Not shown: 65527 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
25/tcp    open  smtp
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
593/tcp   open  http-rpc-epmap
49159/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.70 seconds
```
Interesting port setup for a windows machine
## Nmap
```python

```