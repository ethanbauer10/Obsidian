
# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.13.146
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-10 15:08 +0100
Nmap scan report for 10.129.13.146
Host is up (0.034s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 8.53 seconds
```

## Nmap
```python
nmap -p 22,80 -A --min-rate=2000 -sT 10.129.13.146    
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-10 15:08 +0100
Nmap scan report for 10.129.13.146
Host is up (0.013s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4b:c1:eb:48:87:4a:08:54:89:70:93:b7:c7:a9:ea:79 (ECDSA)
|_  256 46:da:a5:65:91:c9:08:99:b2:96:1d:46:0b:fc:df:63 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://snapped.htb/
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
So it looks like its given me the domain name `snapped.htb`

Ill add this to `/etc/hosts`

# SSH (22)

## Testing auth method
```python
ssh root@snapped.htb                         
The authenticity of host 'snapped.htb (10.129.13.146)' can't be established.
ED25519 key fingerprint is: SHA256:n0XlQQqHGczclhalpCeoOZDYQGr7rl3WlJytHLWPkr8
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'snapped.htb' (ED25519) to the list of known hosts.
root@snapped.htb's password:
```
Password based authentication, less secure than key based

## Version info
```python
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4b:c1:eb:48:87:4a:08:54:89:70:93:b7:c7:a9:ea:79 (ECDSA)
|_  256 46:da:a5:65:91:c9:08:99:b2:96:1d:46:0b:fc:df:63 (ED25519)
```
This version of SSH is not vulnerable

# HTTP (80)

## Ffuf for subdomains
```python
ffuf -u http://snapped.htb/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.snapped.htb' -t 40 -fs 154

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://snapped.htb/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.snapped.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 154
________________________________________________

admin                   [Status: 200, Size: 1407, Words: 164, Lines: 50, Duration: 20ms]
:: Progress: [114442/114442] :: Job [1/1] :: 2531 req/sec :: Duration: [0:00:39] :: Errors: 0 ::
```
Using host header injection i have found a subdomain `admin`

Ill add this to `/etc/hosts` as well

Nothing found on a vulnerability scan

No interesting directories on `http://snapped.htb`

Also nothing looks to be vulnerable at a glance on the site

## `admin` subdomain

