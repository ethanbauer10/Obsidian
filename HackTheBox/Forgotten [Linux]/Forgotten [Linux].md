# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.234.81                                                                          
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-29 18:27 +0100
Nmap scan report for 10.129.234.81
Host is up (0.032s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 7.61 seconds
```

## Nmap
```python
nmap -p 22,80 -A --min-rate=2000 -sT 10.129.234.81                                                                  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-29 18:28 +0100
Nmap scan report for 10.129.234.81
Host is up (0.014s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 28:c7:f1:96:f9:53:64:11:f8:70:55:68:0b:e5:3c:22 (ECDSA)
|_  256 02:43:d2:ba:4e:87:de:77:72:ce:5a:fa:86:5c:0d:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.56
|_http-server-header: Apache/2.4.56 (Debian)
|_http-title: 403 Forbidden
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14
Network Distance: 2 hops
Service Info: Host: 172.17.0.2; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# SSH (22)
## Version info
```python
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 28:c7:f1:96:f9:53:64:11:f8:70:55:68:0b:e5:3c:22 (ECDSA)
|_  256 02:43:d2:ba:4e:87:de:77:72:ce:5a:fa:86:5c:0d:f4 (ED25519)
```
## Auth method
```python
ssh root@10.129.234.81     
The authenticity of host '10.129.234.81 (10.129.234.81)' can't be established.
ED25519 key fingerprint is: SHA256:Dgu3MKOg2XUbjInyeAgQbBsHXIBePlc4jLIgssUKTt0
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.234.81' (ED25519) to the list of known hosts.
(root@10.129.234.81) Password:
```
Password based authentication

# HTTP (80)

![](Pasted%20image%2020260429183134.png)

Landing page is forbidden!

## Feroxbuster
```python
feroxbuster -u http://forgotten.htb/ -C 404 --dont-filter 

403      GET        9l       28w      278c http://forgotten.htb/
301      GET        9l       28w      315c http://forgotten.htb/survey => http://forgotten.htb/survey/
302      GET        0l        0w        0c http://forgotten.htb/survey/images => http://forgotten.htb/survey/index.php?r=installer
302      GET        0l        0w        0c http://forgotten.htb/survey/password => http://forgotten.htb/survey/index.php?r=installer
302      GET        0l        0w        0c http://forgotten.htb/survey/libraries => http://forgotten.htb/survey/index.php?r=installer
302      GET        0l        0w        0c http://forgotten.htb/survey/img => http://forgotten.htb/survey/index.php?r=installer
302      GET        0l        0w        0c http://forgotten.htb/survey/includes => http://forgotten.htb/survey/index.php?r=installer
302      GET        0l        0w        0c http://forgotten.htb/survey/wp-includes => http://forgotten.htb/survey/index.php?r=installer
302      GET        0l        0w        0c http://forgotten.htb/survey/dyn => http://forgotten.htb/survey/index.php?r=installer
```

What interesting is it seems to be finding a lot of endpoints but they all redirect to the same place

