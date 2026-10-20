# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn 10.129.136.40 -vv

Nmap scan report for 10.129.136.40
Host is up, received user-set (0.020s latency).
Scanned at 2026-10-20 16:09:47 BST for 66s
Not shown: 65534 filtered tcp ports (no-response)
PORT    STATE SERVICE REASON
443/tcp open  https   syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 66.25 seconds
```

## Nmap
```python
❯❯❯ nmap -p 443 -A --min-rate=2000 -sT -Pn 10.129.136.40
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-20 16:11 +0100
Nmap scan report for 10.129.136.40
Host is up (0.013s latency).

PORT    STATE SERVICE    VERSION
443/tcp open  ssl/https?
|_ssl-date: 2026-10-20T15:13:17+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=atsserver.acute.local
| Subject Alternative Name: DNS:atsserver.acute.local, DNS:atsserver
| Not valid before: 2022-01-06T06:34:58
|_Not valid after:  2030-01-04T06:34:58
| tls-alpn: 
|   h2
|_  http/1.1
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1903 - 22H2 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
```

Only a web service open

`atsserver.acute.local` ill add to `/etc/hosts`

# HTTPS (443)

![](Pasted%20image%2020261020161640.png)

Im running this through my proxy and im seeing wordpress endpoints

There is a docx i can download

![](Pasted%20image%2020261020162900.png)

This docx file reveals a potential user `lois`

![](Pasted%20image%2020261020163036.png)


