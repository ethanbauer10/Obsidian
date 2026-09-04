# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.129.232.7
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-04 17:07 +0100
Nmap scan report for 10.129.232.7
Host is up (0.014s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 66.20 seconds
```

## Nmap
```python

```

# SSH (22)
## Auth method
```python
ssh root@10.129.232.7                                     
The authenticity of host '10.129.232.7 (10.129.232.7)' can't be established.
ED25519 key fingerprint is: SHA256:JNw/rUlpDzlUEzvKKKFQ/M4prRH35ZhHammHWv47SkY
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.232.7' (ED25519) to the list of known hosts.
root@10.129.232.7's password:
```

Password based auth