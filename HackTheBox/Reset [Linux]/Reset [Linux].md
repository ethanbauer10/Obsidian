# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.234.130                                       
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-20 16:26 +0100
Nmap scan report for 10.129.234.130
Host is up (0.033s latency).
Not shown: 65530 closed tcp ports (conn-refused)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
512/tcp open  exec
513/tcp open  login
514/tcp open  shell

Nmap done: 1 IP address (1 host up) scanned in 7.57 seconds
```
Standard SSH and HTTP but there is also some Rsync ports open
## Nmap
```python

```

# SSH (22)
## Version info
```python

```
## 