# Objective and scope
Orion Corp has hired you to perform an internal penetration test against two machines within their network. Your task is to identify all vulnerabilities and (if possible) elevate your privileges to Domain Admin.

The client has provided you with VPN access to their network, but no credentials.

![806](Pasted%20image%2020261015174115.png)

There are two hosts in scope

```python
❯❯❯ ping 10.0.0.6
PING 10.0.0.6 (10.0.0.6) 56(84) bytes of data.
64 bytes from 10.0.0.6: icmp_seq=1 ttl=127 time=94.1 ms

❯❯❯ ping 10.0.0.5     
PING 10.0.0.5 (10.0.0.5) 56(84) bytes of data.
64 bytes from 10.0.0.5: icmp_seq=1 ttl=127 time=93.9 ms
```

I can reach both hosts, so likely no tunneling in this instance

# Enumeration
## WS-IT (10.0.0.6)

### Open ports
```python
❯❯❯ nmap -p- --min-rate=1000 -sT -Pn 10.0.0.6 -vv 

Nmap scan report for 10.0.0.6
Host is up, received user-set (0.093s latency).
Scanned at 2026-10-15 17:45:44 BST for 128s
Not shown: 65529 filtered tcp ports (no-response)
PORT      STATE SERVICE       REASON
135/tcp   open  msrpc         syn-ack
139/tcp   open  netbios-ssn   syn-ack
445/tcp   open  microsoft-ds  syn-ack
3389/tcp  open  ms-wbt-server syn-ack
5985/tcp  open  wsman         syn-ack
49674/tcp open  unknown       syn-ack
```

### Nmap
```python
❯❯❯ nmap -p 135,139,445,3389,5985 -A --min-rate=1000 -sT -Pn 10.0.0.6
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-15 17:48 +0100
Nmap scan report for 10.0.0.6
Host is up (0.094s latency).

PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server
| rdp-ntlm-info: 
|   Target_Name: ORION
|   NetBIOS_Domain_Name: ORION
|   NetBIOS_Computer_Name: WS-IT
|   DNS_Domain_Name: orion.local
|   DNS_Computer_Name: WS-IT.orion.local
|   DNS_Tree_Name: orion.local
|   Product_Version: 10.0.26100
|_  System_Time: 2026-10-15T16:49:01+00:00
| ssl-cert: Subject: commonName=WS-IT.orion.local
| Not valid before: 2026-10-02T15:45:35
|_Not valid after:  2027-04-03T15:45:35
|_ssl-date: TLS randomness does not represent time
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.991%I=7%D=10/15%Time=6AD103F4%P=x86_64-pc-linux-gnu%r(
SF:TerminalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x0
SF:2\0\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## DC01 (10.0.0.5)
### Open ports
```python
❯❯❯ nmap -p- --min-rate=1000 -sT -Pn 10.0.0.5 -vv

Host is up, received user-set (0.093s latency).
Scanned at 2026-10-15 17:53:13 BST for 1697s
Not shown: 65512 filtered tcp ports (no-response)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
88/tcp    open  kerberos-sec     syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
389/tcp   open  ldap             syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
3389/tcp  open  ms-wbt-server    syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
49664/tcp open  unknown          syn-ack
49667/tcp open  unknown          syn-ack
49668/tcp open  unknown          syn-ack
49670/tcp open  unknown          syn-ack
49671/tcp open  unknown          syn-ack
49693/tcp open  unknown          syn-ack
49715/tcp open  unknown          syn-ack
49733/tcp open  unknown          syn-ack
49807/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 1697.14 seconds
```

Nmap is being reallt

# Host file setup on both machines

```python
❯❯❯ sudo nxc smb 10.0.0.6 --generate-hosts-file /etc/hosts         
SMB         10.0.0.6        445    WS-IT            [*] Windows 11 / Server 2025 Build 26100 x64 (name:WS-IT) (domain:orion.local) (signing:True) (SMBv1:None)

❯❯❯ sudo nxc smb 10.0.0.5 --generate-hosts-file /etc/hosts
SMB         10.0.0.5        445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:orion.local) (signing:True) (SMBv1:None) (Null Auth:True)
```

# SMB (445 - DC01)

```python
❯❯❯ nxc smb dc01.orion.local -u '' -p ''                                             
SMB         10.0.0.5        445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:orion.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.0.5        445    DC01             [+] orion.local\:
```

There is null auth

However i cannot do anything with it

The guest account is also disabled

# SMB (445 - WS-IT)

```python
❯❯❯ nxc smb ws-it.orion.local -u 'Guest' -p '' --smb-timeout 10 --shares
SMB         10.0.0.6        445    WS-IT            [*] Windows 11 / Server 2025 Build 26100 x64 (name:WS-IT) (domain:orion.local) (signing:True) (SMBv1:None)
SMB         10.0.0.6        445    WS-IT            [+] orion.local\Guest: 
SMB         10.0.0.6        445    WS-IT            [*] Enumerated shares
SMB         10.0.0.6        445    WS-IT            Share           Permissions     Remark
SMB         10.0.0.6        445    WS-IT            -----           -----------     ------
SMB         10.0.0.6        445    WS-IT            ADMIN$                          Remote Admin
SMB         10.0.0.6        445    WS-IT            C$                              Default share
SMB         10.0.0.6        445    WS-IT            Company                         Company documents - all staff
SMB         10.0.0.6        445    WS-IT            IPC$            READ            Remote IPC
SMB         10.0.0.6        445    WS-IT            IT                              IT admins only
SMB         10.0.0.6        445    WS-IT            Uploads         READ,WRITE      Public Upload Share
```

The guest account is enabled on the `WS-IT` machine

Also read and write on uploads

```python
❯❯❯ smbclient.py orion.local/Guest:''@ws-it.orion.local                          
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

Password:
Type help for list of commands
# shares
Share Name                Type            Comment
----------------------------------------------------------------------
ADMIN$                    DISK (SPECIAL)  Remote Admin
C$                        DISK (SPECIAL)  Default share
Company                   DISK            Company documents - all staff
IPC$                      IPC (SPECIAL)   Remote IPC
IT                        DISK            IT admins only
Uploads                   DISK            Public Upload Share
# use Uploads
# ls
drw-rw-rw-          0  Thu Oct 15 18:10:22 2026 .
drw-rw-rw-          0  Sat Oct  3 20:59:18 2026 ..
-rw-rw-rw-        509  Sat Oct  3 17:24:40 2026 README.txt
# get README.txt
# 
```

There is a read me file

```python
# cat README.txt
Orion Corp - Public Upload Share
=================================

This share is provided for staff to drop files for the IT department
(scanned documents, logs, screenshots, driver files, etc.).

GUIDELINES
----------
- Files placed here are collected and processed by IT daily.
- Do NOT store sensitive or personal data in this folder.
- Files older than 30 days are automatically removed.
- For urgent requests, contact the helpdesk: helpdesk@orion.local

Thank you,
Orion Corp IT Operations
# 
```



