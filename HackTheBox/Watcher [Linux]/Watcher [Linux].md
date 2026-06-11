
# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.234.163         
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-11 17:59 +0100
Nmap scan report for 10.129.234.163
Host is up (0.020s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
10050/tcp open  zabbix-agent
10051/tcp open  zabbix-trapper

Nmap done: 1 IP address (1 host up) scanned in 21.07 seconds
```

## Nmap
```python
nmap -p 22,80,10050,10051 -A --min-rate=2000 -sT 10.129.234.163
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-11 18:01 +0100
Nmap scan report for 10.129.234.163
Host is up (0.037s latency).

PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f0:e4:e7:ae:27:22:14:09:0c:fe:1a:aa:85:a8:c3:a5 (ECDSA)
|_  256 fd:a3:b9:36:17:39:25:1d:40:6d:5a:07:97:b3:42:13 (ED25519)
80/tcp    open  http       Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Did not follow redirect to http://watcher.vl/
|_http-server-header: Apache/2.4.52 (Ubuntu)
10050/tcp open  tcpwrapped
10051/tcp open  tcpwrapped
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Website is redirecting to `http://watcher.vl`

Ill add this to `/etc/hosts`

# SSH (22)

## Version info
```python
22/tcp    open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 f0:e4:e7:ae:27:22:14:09:0c:fe:1a:aa:85:a8:c3:a5 (ECDSA)
|_  256 fd:a3:b9:36:17:39:25:1d:40:6d:5a:07:97:b3:42:13 (ED25519)
```

## Auth method
```python
ssh root@watcher.vl     
The authenticity of host 'watcher.vl (10.129.234.163)' can't be established.
ED25519 key fingerprint is: SHA256:JDOGxd+Q6ONAdpL+ofsWbYXtuRCe30lTgg7EiA3nMMg
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'watcher.vl' (ED25519) to the list of known hosts.
(root@watcher.vl) Password:
```
SSH allows password based authentication, this is less secure

# HTTP (80)

## Ffuf for subdomians
```python
ffuf -u http://watcher.vl/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.watcher.vl' -t 30 -fs 4991

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://watcher.vl/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.watcher.vl
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 30
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 4991
________________________________________________

zabbix                  [Status: 200, Size: 3946, Words: 199, Lines: 33, Duration: 177ms]
:: Progress: [114442/114442] :: Job [1/1] :: 1807 req/sec :: Duration: [0:01:23] :: Errors: 0 ::
```
Found a subdomain `zabbiz`