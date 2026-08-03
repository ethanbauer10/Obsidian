
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
nmap -p 53,88,111,135,139,389,445,464,593,636,2049,3389,5985 -A --min-rate=1000 -sT dc01.fragments.local  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-03 16:46 +0100
Nmap scan report for dc01.fragments.local (10.0.29.162)
Host is up (0.098s latency).
rDNS record for 10.0.29.162: DC01.fragments.local

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-03 15:46:42Z)
111/tcp  open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: fragments.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
2049/tcp open  nlockmgr      1-4 (RPC #100021)
3389/tcp open  ms-wbt-server
| ssl-cert: Subject: commonName=DC01.fragments.local
| Not valid before: 2026-07-13T01:02:46
|_Not valid after:  2027-01-12T01:02:46
| rdp-ntlm-info: 
|   Target_Name: FRAGMENTS
|   NetBIOS_Domain_Name: FRAGMENTS
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: fragments.local
|   DNS_Computer_Name: DC01.fragments.local
|   Product_Version: 10.0.26100
|_  System_Time: 2026-08-03T15:47:36+00:00
|_ssl-date: TLS randomness does not represent time
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.99%I=7%D=8/3%Time=6A70B7E7%P=x86_64-pc-linux-gnu%r(Ter
SF:minalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02\0
SF:\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```


