
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

