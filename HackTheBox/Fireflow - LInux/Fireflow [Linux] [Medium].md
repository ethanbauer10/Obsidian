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


