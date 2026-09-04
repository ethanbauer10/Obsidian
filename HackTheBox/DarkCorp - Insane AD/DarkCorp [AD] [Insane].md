# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.129.232.7
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-04 17:07 +0100
Nmap scan report for 10.129.232.7
Host is up (0.014s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 66.20 seconds
```

## Nmap
```python
nmap -p 22,80 -A --min-rate=2000 -sT 10.129.232.7    
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-04 17:09 +0100
Nmap scan report for 10.129.232.7
Host is up (0.014s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 33:41:ed:0a:a5:1a:86:d0:cc:2a:a6:2b:8d:8d:b2:ad (ECDSA)
|_  256 04:ad:7e:ba:11:0e:e0:fb:d0:80:d3:24:c2:3e:2c:c5 (ED25519)
80/tcp open  http    nginx 1.22.1
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: nginx/1.22.1
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# SSH (22)
## Auth method
```python
ssh root@10.129.232.7                                     
The authenticity of host '10.129.232.7 (10.129.232.7)' can't be established.
ED25519 key fingerprint is: SHA256:JNw/rUlpDzlUEzvKKKFQ/M4prRH35ZhHammHWv47SkY
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.232.7' (ED25519) to the list of known hosts.
root@10.129.232.7's password:
```

Password based auth

# HTTP (80)

The website points at the domain `drip.htb`

![](Pasted%20image%2020260904171628.png)

The sign in link takes me to `mail.drip.htb`

![](Pasted%20image%2020260904172715.png)

DripMail instance

![1049](Pasted%20image%2020260904172910.png)

There is also a register function on the page

Just as i thought registering an accout