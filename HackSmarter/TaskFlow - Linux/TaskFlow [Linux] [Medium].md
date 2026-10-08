# Lab description
You have been hired to perform a penetration test against the client's development infrastructure. The dev team relies heavily on a project management application, which they have provided you access to.

Your task is to start as an unauthenticated attacker, identify all vulnerabilities, and demonstrate full impact by compromising the underlying host (if possible).

The client has provided you with VPN access to their environment, but no other information.

# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn 10.0.22.103 -vv

Nmap scan report for 10.0.22.103
Host is up, received user-set (0.11s latency).
Scanned at 2026-10-08 17:42:19 BST for 66s
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack
```

## Nmap
```python
❯❯❯ nmap -p 22,80 -A --min-rate=2000 -sT -Pn 10.0.22.103
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-08 17:47 +0100
Nmap scan report for 10.0.22.103
Host is up (0.12s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.5 (protocol 2.0)
80/tcp open  http    nginx 1.30.4
|_http-server-header: nginx/1.30.4
|_http-title: Did not follow redirect to http://taskflow.hsm/
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Linux 4.X|5.X (87%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
Aggressive OS guesses: Linux 4.15 - 5.19 (87%)
No exact OS matches for host (test conditions non-ideal).
```

# SSH (22)
## Version info
```python
22/tcp open  ssh     OpenSSH 10.5 (protocol 2.0)
```

This version is not vulnerable

## Auth mehotd

# HTTP (80)
