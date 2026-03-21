# Hostname setup
```python
sudo nxc smb 10.129.234.42 --generate-hosts-file /etc/hosts      
[sudo] password for kali: 
SMB         10.129.234.42   445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:None) (Null Auth:True)
```
This has created me an entry in `/etc/hosts`

# Enuumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.234.42
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-21 17:13 +0000
Nmap scan report for 10.129.234.42
Host is up (0.014s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
50168/tcp open  unknown
50182/tcp open  unknown
50200/tcp open  unknown
50888/tcp open  unknown
51970/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.34 seconds
```
Whats interesting here is that there is SSH (22) open
## Nmap
```python

```