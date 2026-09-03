# Objective and scope
You have been authorized to perform an external penetration test against a target organization. During the initial reconnaissance phase, you identified a web application that allows unrestricted public user registration.

1. **Enumerate:** Map the application's attack surface and functionality.
2. **Identify:** Locate exploitable vulnerabilities within the application logic or configuration.
3. **Exploit & Escalate:** Leverage identified flaws to compromise the system, with the final goal of securing root access to the host server to demonstrate maximum impact.

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.1.68.30          
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-03 12:38 -0400
Nmap scan report for 10.1.68.30
Host is up (0.090s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 34.07 seconds
```

## Nmap
```python
nmap -p 22,80 -A --min-rate=2000 -sT verbose.hsm
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-03 12:40 -0400
Nmap scan report for verbose.hsm (10.1.68.30)
Host is up (0.090s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 7d:b8:dd:2a:63:d2:54:01:2d:ad:ba:24:9b:8f:95:54 (ECDSA)
|_  256 28:68:86:e0:98:47:3d:21:49:d0:51:3a:a3:6a:f9:11 (ED25519)
80/tcp open  http    Werkzeug httpd 3.1.5 (Python 3.12.3)
|_http-server-header: Werkzeug/3.1.5 Python/3.12.3
| http-title: Hack Smarter Portal
|_Requested resource was /login
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Its running a python flask web application

# SSH (22)

```python
ssh root@verbose.hsm             
The authenticity of host 'verbose.hsm (10.1.68.30)' can't be established.
ED25519 key fingerprint is: SHA256:Jl7742WucHUy7u4IdLyM87KPNDCk7yXh59zOWMvR/bE
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'verbose.hsm' (ED25519) to the list of known hosts.
root@verbose.hsm: Permission denied (publickey).
```

It is using key based authentication, this is more secure and would go down as a strength

# HTTP (80)

![](Pasted%20image%2020260903124159.png)

The landing page redirects to `/login`

![](Pasted%20image%2020260903124251.png)

Using the creds `admin:admin` i get incorrect password, this tells me the admin user does exist

Not vulnerable to SQLi

The forgot password function takes a username and send the password reset link

Ill register an account

![](Pasted%20image%2020260903124537.png)

Once registering and logging in i see this screen, this screen immediately looks vulnerable to things like HTML injection, XSS and even SSTI since the user input is being reflected on the screen

Since this is a python web application, and its saying `welcome back, <username>` i think its likely using something like jinja2 as a templating engine to render this, i might be able to abuse this with SSTI

## HTML injection

![](Pasted%20image%2020260903125848.png)

In the messages endpoint i can send a message to a user and embed the values inside flags like `<h1></h1>` to render headers, this confirms HTML injection

## Cross-Site Scripting (XSS)

![](Pasted%20image%2020260903130002.png)

![](Pasted%20image%2020260903130014.png)

In the same field i can also get XSS, and since this is a messaging feature i can get javascript to execute in another users browser and exfil session cookies to compromise user accounts

## Username enumeration

![](Pasted%20image%2020260903130313.png)

The messages endpoint shows me other users

## Server-Side Template Injection

![](Pasted%20image%2020260903130423.png)

![](Pasted%20image%2020260903130438.png)

I also have confirmed SSTI, in this endpoint, which should allow remote code execution

Since this payload worked, i can confirm this is running the jinja2 templating engine which makes sense since this is a python web application

```python
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

Ill try this again, but ill use the above payload and see if i can get RCE

![](Pasted%20image%2020260903131307.png)

I now have RCE

## More SSTI in `/profile`

On the landing page i had a message for `welcome back, <user>` this is likely a templating engine rendering this

![](Pasted%20image%2020260903131703.png)

If i can mody my username to a payload like `{{7*7}}` i shoud be able to get SSTI once again

But as of this point this field is locked, i can unlock it using the developer tools

![](Pasted%20image%2020260903131849.png)

I can simply remove the disabled value and change the username

![](Pasted%20image%2020260903131919.png)

Now i can change it

![](Pasted%20image%2020260903131935.png)

Ill change it to the payload mentioned

![](Pasted%20image%2020260903131956.png)

It renders my payload, this can be turned into another RCE vector

## Password reset function leads to full account takeover

![](Pasted%20image%2020260903132615.png)

When this is chained with something like XSS it can be used to exfil session IDs bypass MFA and reset accounts, leading to full account takeover

