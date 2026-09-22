# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.129.60.255 -vv

PORT     STATE SERVICE REASON
3000/tcp open  ppp     syn-ack
```

## Nmap
```python
nmap -p 3000 -A --min-rate=2000 -sT -Pn 10.129.60.255    
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-22 17:14 +0100
Nmap scan report for 10.129.60.255
Host is up (0.014s latency).

PORT     STATE SERVICE VERSION
3000/tcp open  http    Node.js Express framework
|_http-title: Did not follow redirect to http://aegis.korvia.htb:3000/
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|11|2012 (87%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2
Aggressive OS guesses: Microsoft Windows Server 2022 (87%), Microsoft Windows 11 24H2 (85%), Microsoft Windows Server 2012 R2 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using proto 1/icmp)
HOP RTT      ADDRESS
1   13.85 ms 10.10.14.1
2   14.02 ms 10.129.60.255

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.95 seconds
```

# HTTP (3000)

![](Pasted%20image%2020260922171633.png)

Browsing to the IP address gives me this domain!

![](Pasted%20image%2020260922171835.png)

The landing page is a logon portal

