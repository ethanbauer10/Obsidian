# Initial credentials
```python
P.Rosa:Rosaisbest123
```

# Host file setup
```python
sudo nxc smb 10.129.231.205 --generate-hosts-file /etc/hosts        
[sudo] password for kali: 
SMB         10.129.231.205  445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
```
So some things to note:
- NTLM auth disabled
- SMB signing is enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.vintage.htb      
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 16:28 +0100
Nmap scan report for dc01.vintage.htb (10.129.231.205)
Host is up (0.014s latency).
Not shown: 65516 filtered tcp ports (no-response)
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
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
49676/tcp open  unknown
49687/tcp open  unknown
52792/tcp open  unknown
62928/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.80 seconds
```

## Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT dc01.vintage.htb
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 16:30 +0100
Nmap scan report for dc01.vintage.htb (10.129.231.205)
Host is up (0.017s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-02 15:30:36Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: vintage.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: vintage.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```
DC is running at -1h

# SMB (445)
Since NTLM is disabled i basically need to get a tgt

```python
nxc smb dc01.vintage.htb -u 'p.rosa' -p 'Rosaisbest123' -k            
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [+] vintage.htb\p.rosa:Rosaisbest123
```
The creds work to make this simple though ill just generate a tgt for this save entering credentials

```python
nxc smb dc01.vintage.htb -u 'p.rosa' -p 'Rosaisbest123' -k --generate-tgt p_rosa.tgt
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [+] vintage.htb\p.rosa:Rosaisbest123 
SMB         dc01.vintage.htb 445    dc01             [+] TGT saved to: p_rosa.tgt.ccache
SMB         dc01.vintage.htb 445    dc01             [+] Run the following command to use the TGT: export KRB5CCNAME=p_rosa.tgt.ccache
```
Now i can export it!

```python

```