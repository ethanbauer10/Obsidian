
# Objective and Scope

You have been hired by Hack Smarter to perform a Penetration Test on Past Systems Inc. During your call with the client, they stated they are currently adding new machines to the network.

The client has provided you with VPN access to their internal network, but no credentials.

# Host file setup

```python
sudo nxc smb 10.0.16.121 --generate-hosts-file /etc/hosts                             
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [*] Windows Server 2016 Datacenter 14393 x64 (name:EC2AMAZ-A5O4OL8) (domain:past.local) (signing:True) (SMBv1:True) (Null Auth:True)
```

Looks as if its running an old version of Windows

And its also running SMB v1?

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT EC2AMAZ-A5O4OL8.past.local       
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-27 16:43 +0100
Nmap scan report for EC2AMAZ-A5O4OL8.past.local (10.0.16.121)
Host is up (0.095s latency).
Not shown: 65520 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
593/tcp   open  http-rpc-epmap
3389/tcp  open  ms-wbt-server
47001/tcp open  winrm
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49700/tcp open  unknown
49705/tcp open  unknown
57538/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.93 seconds
```

## Nmap
```python
nmap -p 53,88,135,139,389,445,593,3389 -A --min-rate=2000 -sT EC2AMAZ-A5O4OL8.past.local                
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-27 16:46 +0100
Nmap scan report for EC2AMAZ-A5O4OL8.past.local (10.0.16.121)
Host is up (0.095s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-27 15:46:07Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: past.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds  Windows Server 2016 Datacenter 14393 microsoft-ds (workgroup: PAST)
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-07-27T15:46:56+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=EC2AMAZ-A5O4OL8.past.local
| Not valid before: 2026-07-26T15:35:57
|_Not valid after:  2027-01-25T15:35:57
| rdp-ntlm-info: 
|   Target_Name: PAST
|   NetBIOS_Domain_Name: PAST
|   NetBIOS_Computer_Name: EC2AMAZ-A5O4OL8
|   DNS_Domain_Name: past.local
|   DNS_Computer_Name: EC2AMAZ-A5O4OL8.past.local
|   Product_Version: 10.0.14393
|_  System_Time: 2026-07-27T15:46:16+00:00
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2016|2012 (87%)
OS CPE: cpe:/o:microsoft:windows_server_2016 cpe:/o:microsoft:windows_server_2012:r2
Aggressive OS guesses: Microsoft Windows Server 2016 (87%), Microsoft Windows Server 2012 R2 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: Host: EC2AMAZ-A5O4OL8; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (139)

So since this is running SMB v1, it will be running on port 139 as opposed to 445 for SMB v2 and v3

So null auth is enabled as with all DCs by default, but cant use it to dump users or list shares

## Guest access

The guest account is enabled!

### Users
```python
nxc smb EC2AMAZ-A5O4OL8.past.local -u 'Guest' -p '' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 > users.txt
```

```python
cat users.txt 
Administrator
Guest
krbtgt
DefaultAccount
tyler
EC2AMAZ-A5O4OL8$
APPDEV01$
WEBDEV01$
DEV01$
ryan
```
### Shares
```python

```