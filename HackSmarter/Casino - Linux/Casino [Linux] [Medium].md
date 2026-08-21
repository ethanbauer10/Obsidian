# Objective and scope
Las Vegas is gearing up for a massive cybersecurity conference, and you've been hired to conduct a penetration test against one of the casinos. The client - Hack Smarter World - is a luxury resort where many of the attendees will be staying. Your objective is to identify all vulnerabilities and elevate your privileges to root (if possible).

You have been provided the IP of the Wifi Captive Portal... but no other information.

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.0.16.132      
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-21 16:29 +0100
Nmap scan report for 10.0.16.132
Host is up (0.095s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
2222/tcp open  EtherNetIP-1

Nmap done: 1 IP address (1 host up) scanned in 35.04 seconds
```

## Nmap
```python

```
