# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.129.96.147 -vv

Nmap scan report for 10.129.96.147
Host is up, received user-set (0.014s latency).
Scanned at 2026-09-25 16:32:08 BST for 66s
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE    REASON
80/tcp   open  http       syn-ack
5985/tcp open  wsman      syn-ack
8080/tcp open  http-proxy syn-ack
```

## Nmap
```python
nmap -p 80,5985,8080 -A --min-rate=2000 -sT -Pn 10.129.96.147
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-25 16:33 +0100
Nmap scan report for 10.129.96.147
Host is up (0.014s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Mega Engines
| http-methods: 
|_  Potentially risky methods: TRACE
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
8080/tcp open  http    Jetty 9.4.43.v20210629
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
|_http-server-header: Jetty(9.4.43.v20210629)
| http-robots.txt: 1 disallowed entry 
|_/
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1903 - 22H2 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

# HTTP (80)

![](Pasted%20image%2020260925163525.png)

 IIS

Also a link to a automation server?

Also a potential user `ideas`

# HTTP (8080)

![](Pasted%20image%2020260925163701.png)

It goes to `object.htb`

![](Pasted%20image%2020260925163820.png)

Jenkins install

It also looks to accept user registration

