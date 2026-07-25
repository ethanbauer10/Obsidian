
# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.238.32 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-25 17:23 +0100
Nmap scan report for 10.129.238.32
Host is up (0.034s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp
5001/tcp open  commplex-link
5002/tcp open  rfe

Nmap done: 1 IP address (1 host up) scanned in 9.98 seconds
```

## Nmap
```python
nmap -p 22,5000,5001,5002 -A --min-rate=2000 -sT 10.129.238.32
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-25 17:24 +0100
Nmap scan report for 10.129.238.32
Host is up (0.013s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 30:68:b8:a8:f5:47:ca:bf:1a:23:97:d5:4c:77:97:da (ECDSA)
|_  256 3f:83:9f:53:0a:49:db:00:d5:18:85:e9:2f:05:76:dd (ED25519)
5000/tcp open  http    Node.js (Express middleware)
|_http-title: Secure Encrypted Storage - 01001101 01101001 01101100 01101001...
5001/tcp open  http    Node.js (Express middleware)
|_http-title: Secure Encrypted Storage - 01001101 01101001 01101100 01101001...
5002/tcp open  http    Node.js (Express middleware)
|_http-title: Secure Encrypted Storage - 01001101 01101001 01101100 01101001...
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

All three web servers show the same site

# SSH (22)
## Version
```python
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 30:68:b8:a8:f5:47:ca:bf:1a:23:97:d5:4c:77:97:da (ECDSA)
|_  256 3f:83:9f:53:0a:49:db:00:d5:18:85:e9:2f:05:76:dd (ED25519)
```

## Auth method
```python
ssh root@10.129.238.32                 
The authenticity of host '10.129.238.32 (10.129.238.32)' can't be established.
ED25519 key fingerprint is: SHA256:MwKYyiZT4gAZ35VHTeZ0760cn1Fe7QFCFllCxBST4rA
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.238.32' (ED25519) to the list of known hosts.
(root@10.129.238.32) Password:
```

Password based auth, less secure

# HTTP (5000)

At first glance it appears that all the web servers are the same

Wappalyzer detects Express

No vhosts

Feroxbuster found `/tmp/` 

But it returns an error so no directory listing

So it appears after every upload it saves an encrypted copy in `/tmp/` and the plaintext version in `/file/`

So when you request the file in `/file/` is pulls from `/tmp/` then decrypts it.

![](Pasted%20image%2020260725183249.png)

But now the question is can i pull other files from `/tmp/` like `/etc/passwd`

![](Pasted%20image%2020260725184044.png)

Now testing this theory i see if i try with `/etc/passwd` i can pull from `/tmp/` in the `/file/` directory, now the only problem is the encrypted output since everything that gets pulled from `/tmp/` is encrypted

So i can start by making a guess that the start of `/etc/passwd` is:

```python
root:x:0:0:root:/root:/bin/bash
```

