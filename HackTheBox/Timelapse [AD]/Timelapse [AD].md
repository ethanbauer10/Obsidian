
# Host file setup
```python
[Jun 13, 2026 - 17:56:48 (BST)] exegol-CTF timelapse # nxc smb 10.129.227.113 --generate-hosts-file /etc/hosts 
SMB         10.129.227.113  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:timelapse.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
# nmap -p- --min-rate=2000 -sT dc01.timelapse.htb                                                                     
Starting Nmap 7.93 ( https://nmap.org ) at 2026-06-13 17:58 BST
Nmap scan report for dc01.timelapse.htb (10.129.227.113)
Host is up (0.013s latency).
rDNS record for 10.129.227.113: DC01.timelapse.htb
Not shown: 65517 filtered tcp ports (no-response)
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
5986/tcp  open  wsmans
9389/tcp  open  adws
49667/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49693/tcp open  unknown
49727/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.77 seconds
```

## Nmap
```python
# nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5986,9389 -A --min-rate=2000 -sT dc01.timelapse.htb             
Starting Nmap 7.93 ( https://nmap.org ) at 2026-06-13 18:01 BST
Nmap scan report for dc01.timelapse.htb (10.129.227.113)
Host is up (0.013s latency).
rDNS record for 10.129.227.113: DC01.timelapse.htb

PORT     STATE SERVICE           VERSION
53/tcp   open  domain            Simple DNS Plus
88/tcp   open  kerberos-sec      Microsoft Windows Kerberos (server time: 2026-06-14 01:01:25Z)
135/tcp  open  msrpc             Microsoft Windows RPC
139/tcp  open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ldapssl?
3268/tcp open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
3269/tcp open  globalcatLDAPssl?
5986/tcp open  ssl/http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_ssl-date: 2026-06-14T01:02:50+00:00; +8h00m00s from scanner time.
| ssl-cert: Subject: commonName=dc01.timelapse.htb
| Not valid before: 2021-10-25T14:05:29
|_Not valid after:  2022-10-25T14:25:29
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
| tls-alpn: 
|_  http/1.1
9389/tcp open  mc-nmf            .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Null authentication is enabled, but cannot use it to access shares or list users

## Guest access

The guest account is enabled

```python
# nxc smb dc01.timelapse.htb -u 'Guest' -p ''             
SMB         10.129.227.113  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:timelapse.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.227.113  445    DC01             [+] timelapse.htb\Guest:
```


