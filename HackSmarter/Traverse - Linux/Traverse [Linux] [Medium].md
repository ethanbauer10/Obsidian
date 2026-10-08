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
ftp> ls
229 Entering Extended Passive Mode (|||30044|)
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Sep 22 07:49 devops
-rw-rw-r--    1 ftp      ftp       3507597 Jun 20  2022 mountains-wallpaper-photo.jpg
226 Directory send OK.
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

