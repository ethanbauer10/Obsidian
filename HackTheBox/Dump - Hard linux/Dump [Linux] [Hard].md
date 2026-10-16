# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn 10.129.234.97 -vv

Nmap scan report for 10.129.234.97
Host is up, received user-set (0.031s latency).
Scanned at 2026-10-16 16:20:14 BST for 7s
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack
```

## Nmap
```python
❯❯❯ nmap -p 22,80 -A --min-rate=2000 -sT -Pn 10.129.234.97           
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-16 16:21 +0100
Nmap scan report for 10.129.234.97
Host is up (0.014s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u5 (protocol 2.0)
| ssh-hostkey: 
|   3072 fb:31:61:8d:2f:86:e5:60:f9:e6:24:a3:1c:62:0c:ae (RSA)
|   256 0c:b7:c4:fb:4a:fc:31:1b:e9:4b:0b:d1:19:56:2f:ce (ECDSA)
|_  256 3c:c6:e8:71:4d:9a:d5:1d:86:dd:dd:6c:82:ee:7e:4d (ED25519)
80/tcp open  http    Apache httpd 2.4.65 ((Debian))
|_http-server-header: Apache/2.4.65 (Debian)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: hdmpll?
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# SSH (22)
## Version info
```python
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u5 (protocol 2.0)
| ssh-hostkey: 
|   3072 fb:31:61:8d:2f:86:e5:60:f9:e6:24:a3:1c:62:0c:ae (RSA)
|   256 0c:b7:c4:fb:4a:fc:31:1b:e9:4b:0b:d1:19:56:2f:ce (ECDSA)
|_  256 3c:c6:e8:71:4d:9a:d5:1d:86:dd:dd:6c:82:ee:7e:4d (ED25519)
```

## Auth method

![](Pasted%20image%2020261016162413.png)

It accepts password based auth, this is less secure

# HTTP (80)

![](Pasted%20image%2020261016162703.png)

The landing page is a login form

Ill attempt some logons using default credentials, no overly verbose error messages telling me if a user exists

Also does not appear to be vulnerable to SQL injection

Ill create a user and login 

![](Pasted%20image%2020261016163026.png)

So there are three main options, capture, download and upload

Clicking on download gives me a 404 not found, this is likely because there isnt any stored

If i click on capture it runs a capture on port 31330 for 10s

![1055](Pasted%20image%2020261016163525.png)

If i then return to the home page....

![](Pasted%20image%2020261016163547.png)

I can see there are captures

![](Pasted%20image%2020261016163633.png)

Clicking into one, shows the output on the screen

Its also using the URL parameter `?fn=`

It will be worth testing for LFI and RFI


