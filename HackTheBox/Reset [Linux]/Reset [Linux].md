# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.234.130                                       
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-20 16:26 +0100
Nmap scan report for 10.129.234.130
Host is up (0.033s latency).
Not shown: 65530 closed tcp ports (conn-refused)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
512/tcp open  exec
513/tcp open  login
514/tcp open  shell

Nmap done: 1 IP address (1 host up) scanned in 7.57 seconds
```
Standard SSH and HTTP but there is also some Rsync ports open
## Nmap
```python
nmap -p 22,80,512,513,514 -A --min-rate=2000 -sT 10.129.234.130
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-20 16:28 +0100
Nmap scan report for 10.129.234.130
Host is up (0.014s latency).

PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 6a:16:1f:c8:fe:fd:e3:98:a6:85:cf:fe:7b:0e:60:aa (ECDSA)
|_  256 e4:08:cc:5f:8e:56:25:8f:38:c3:ec:df:b8:86:0c:69 (ED25519)
80/tcp  open  http    Apache httpd 2.4.52 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Admin Login
|_http-server-header: Apache/2.4.52 (Ubuntu)
512/tcp open  exec    netkit-rsh rexecd
513/tcp open  login?
514/tcp open  shell   Netkit rshd
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# SSH (22)
## Version info
```python
22/tcp  open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 6a:16:1f:c8:fe:fd:e3:98:a6:85:cf:fe:7b:0e:60:aa (ECDSA)
|_  256 e4:08:cc:5f:8e:56:25:8f:38:c3:ec:df:b8:86:0c:69 (ED25519)
```
## Auth method
```python
ssh root@10.129.234.130 
The authenticity of host '10.129.234.130 (10.129.234.130)' can't be established.
ED25519 key fingerprint is: SHA256:HFXQA4H5wiZsaBpe1Ymxvzg72KiVi0uEjib0GxsYF30
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.234.130' (ED25519) to the list of known hosts.
root@10.129.234.130's password:
```
Password based authentication!

