# Objective and scope
You have been authorized to perform an external penetration test against a target organization. During the initial reconnaissance phase, you identified a web application that allows unrestricted public user registration.

1. **Enumerate:** Map the application's attack surface and functionality.
2. **Identify:** Locate exploitable vulnerabilities within the application logic or configuration.
3. **Exploit & Escalate:** Leverage identified flaws to compromise the system, with the final goal of securing root access to the host server to demonstrate maximum impact.

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.1.68.30          
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-03 12:38 -0400
Nmap scan report for 10.1.68.30
Host is up (0.090s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 34.07 seconds
```

## Nmap
```python

```
