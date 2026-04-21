# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.96.147
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-21 15:08 +0100
Nmap scan report for 10.129.96.147
Host is up (0.012s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
5985/tcp open  wsman
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 66.29 seconds
```

## Nmap
```python
nmap -p 80,5985,8080 -A --min-rate=2000 -sT 10.129.96.147      
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-21 15:10 +0100
Nmap scan report for 10.129.96.147
Host is up (0.013s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Mega Engines
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
8080/tcp open  http    Jetty 9.4.43.v20210629
|_http-server-header: Jetty(9.4.43.v20210629)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1903 - 22H2 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
Two webservers and also winrm

Cannot really do anything with winrm until i get credentials

# HTTP ()