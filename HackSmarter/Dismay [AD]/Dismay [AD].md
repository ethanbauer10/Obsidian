# Objective and scope
You have been assigned a penetration test of a client's internal network. There are three different servers in-scope. Your task is to identify all vulnerabilities and demonstrate impact to the client by elevating your access to Domain Admin. The client has provided you VPN access and Active Directory credentials (below) for the engagement.

Please note - there are 2 intended paths to compromise this machine. Once you solve it one way, we encourage you to try and solve it a second way.

# Hosts
DC1 - **10.0.16.4**
DC2 - **10.0.18.13**
Nexus - **10.0.27.179**

# Initial Access

The client has provided you with Active Directory credentials and VPN access. `xiao.ge:AmBZATVjnH4qo8H4`

# Host file setup
```python
sudo nxc smb 10.0.16.4 --generate-hosts-file /etc/hosts    
[sudo] password for kali: 
SMB         10.0.16.4       445    DC1              [*] Windows Server 2022 Build 20348 x64 (name:DC1) (domain:dismay.hsm) (signing:True) (SMBv1:None) (Null Auth:True)


sudo nxc smb 10.0.18.13 --generate-hosts-file /etc/hosts
SMB         10.0.18.13      445    DC2              [*] Windows Server 2022 Build 20348 x64 (name:DC2) (domain:dismay.hsm) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Nexus server - **10.0.27.179**
### Open ports
```python
nmap -p- --min-rate=2000 -sT 10.0.27.179                      
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-24 15:29 +0100
Nmap scan report for 10.0.27.179
Host is up (0.096s latency).
Not shown: 65527 filtered tcp ports (no-response)
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
8530/tcp  open  unknown
8531/tcp  open  unknown
49668/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.41 seconds
```

### Nmap
```python
nmap -p 80,135,139,445,3389,8530,8531 -A --min-rate=2000 -sT 10.0.27.179
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-24 15:33 +0100
Nmap scan report for 10.0.27.179
Host is up (0.097s latency).

PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-04-24T14:34:47+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: EC2AMAZ-GQCP864
|   NetBIOS_Domain_Name: EC2AMAZ-GQCP864
|   NetBIOS_Computer_Name: EC2AMAZ-GQCP864
|   DNS_Domain_Name: EC2AMAZ-GQCP864
|   DNS_Computer_Name: EC2AMAZ-GQCP864
|   Product_Version: 10.0.20348
|_  System_Time: 2026-04-24T14:34:08+00:00
| ssl-cert: Subject: commonName=EC2AMAZ-GQCP864
| Not valid before: 2026-02-06T01:27:31
|_Not valid after:  2026-08-08T01:27:31
8530/tcp open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title.
| http-methods: 
|_  Potentially risky methods: TRACE
8531/tcp open  unknown
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## DC2 - **10.0.18.13**
### Open ports
```python
nmap -p- --min-rate=2000 -sT 10.0.18.13 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-24 15:34 +0100
Nmap scan report for DC2.dismay.hsm (10.0.18.13)
Host is up (0.096s latency).
Not shown: 65523 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
443/tcp   open  https
445/tcp   open  microsoft-ds
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3389/tcp  open  ms-wbt-server
49678/tcp open  unknown
60485/tcp open  unknown
65507/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.88 seconds
```

### Nmap
```python
nmap -p 53,80,135,139,443,445,593,636,3389 -A --min-rate=2000 -sT 10.0.18.13 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-24 15:37 +0100
Nmap scan report for DC2.dismay.hsm (10.0.18.13)
Host is up (0.095s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp  open  https?
445/tcp  open  microsoft-ds?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: dismay.hsm, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC2.dismay.hsm, DNS:dismay.hsm, DNS:DISMAY
| Not valid before: 2025-12-12T13:39:44
|_Not valid after:  2026-12-12T13:39:44
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-04-24T14:38:56+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=DC2.dismay.hsm
| Not valid before: 2025-12-11T12:29:58
|_Not valid after:  2026-06-12T12:29:58
| rdp-ntlm-info: 
|   Target_Name: DISMAY
|   NetBIOS_Domain_Name: DISMAY
|   NetBIOS_Computer_Name: DC2
|   DNS_Domain_Name: dismay.hsm
|   DNS_Computer_Name: DC2.dismay.hsm
|   Product_Version: 10.0.20348
|_  System_Time: 2026-04-24T14:38:16+00:00
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: Host: DC2; OS: Windows; CPE: cpe:/o:microsoft:windows
```

## DC1 - **10.0.16.4**
### Open ports
```python
nmap -p- --min-rate=2000 -sT 10.0.16.4 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-24 15:38 +0100
Nmap scan report for DC1.dismay.hsm (10.0.16.4)
Host is up (0.096s latency).
Not shown: 65524 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
49669/tcp open  unknown
51106/tcp open  unknown
51116/tcp open  unknown
51144/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.89 seconds
```

### Nmap
```python
nmap -p 53,88,135,139,389,445,3389 -A --min-rate=2000 -sT 10.0.16.4 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-24 15:41 +0100
Nmap scan report for DC1.dismay.hsm (10.0.16.4)
Host is up (0.095s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-24 14:41:33Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: dismay.hsm, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC1.dismay.hsm, DNS:dismay.hsm, DNS:DISMAY
| Not valid before: 2025-12-12T13:42:13
|_Not valid after:  2026-12-12T13:42:13
|_ssl-date: TLS randomness does not represent time
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: DISMAY
|   NetBIOS_Domain_Name: DISMAY
|   NetBIOS_Computer_Name: DC1
|   DNS_Domain_Name: dismay.hsm
|   DNS_Computer_Name: DC1.dismay.hsm
|   DNS_Tree_Name: dismay.hsm
|   Product_Version: 10.0.20348
|_  System_Time: 2026-04-24T14:41:46+00:00
| ssl-cert: Subject: commonName=DC1.dismay.hsm
| Not valid before: 2025-12-11T11:50:46
|_Not valid after:  2026-06-12T11:50:46
|_ssl-date: 2026-04-24T14:42:26+00:00; 0s from scanner time.
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: Host: DC1; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# Nexus server (10.0.27.179)
## HTTP (80)
Landing page is a default IIS page!

Nothing interesting out of nuclei

No subdomains

