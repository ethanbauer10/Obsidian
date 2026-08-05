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
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,3389,5985,9389 -A --min-rate=1000 -sT dc01.casper.hsm
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-05 16:27 +0100
Nmap scan report for dc01.casper.hsm (10.0.31.82)
Host is up (0.095s latency).
rDNS record for 10.0.31.82: DC01.casper.hsm

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-05 15:27:24Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: casper.hsm, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.casper.hsm
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.casper.hsm
| Not valid before: 2026-07-26T08:04:06
|_Not valid after:  2027-07-26T08:04:06
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: casper.hsm, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.casper.hsm
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.casper.hsm
| Not valid before: 2026-07-26T08:04:06
|_Not valid after:  2027-07-26T08:04:06
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: casper.hsm, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.casper.hsm
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.casper.hsm
| Not valid before: 2026-07-26T08:04:06
|_Not valid after:  2027-07-26T08:04:06
|_ssl-date: TLS randomness does not represent time
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: casper.hsm, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.casper.hsm
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.casper.hsm
| Not valid before: 2026-07-26T08:04:06
|_Not valid after:  2027-07-26T08:04:06
|_ssl-date: TLS randomness does not represent time
3389/tcp open  ms-wbt-server
|_ssl-date: TLS randomness does not represent time
| rdp-ntlm-info: 
|   Target_Name: CASPER
|   NetBIOS_Domain_Name: CASPER
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: casper.hsm
|   DNS_Computer_Name: DC01.casper.hsm
|   DNS_Tree_Name: casper.hsm
|   Product_Version: 10.0.26100
|_  System_Time: 2026-08-05T15:28:16+00:00
| ssl-cert: Subject: commonName=DC01.casper.hsm
| Not valid before: 2026-07-25T07:13:00
|_Not valid after:  2027-01-24T07:13:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf        .NET Message Framing
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.99%I=7%D=8/5%Time=6A735661%P=x86_64-pc-linux-gnu%r(Ter
SF:minalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02\0
SF:\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

## Linux Server
### Open ports
```python
rustscan -a nix01.casper.hsm

Open 10.0.20.54:22
Open 10.0.20.54:80
Open 10.0.20.54:8060
Open 10.0.20.54:9094
```

Nmap was taking too long!

### Nmap
```python

```