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


# SSH (22)

```python
ssh root@verbose.hsm             
The authenticity of host 'verbose.hsm (10.1.68.30)' can't be established.
ED25519 key fingerprint is: SHA256:Jl7742WucHUy7u4IdLyM87KPNDCk7yXh59zOWMvR/bE
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'verbose.hsm' (ED25519) to the list of known hosts.
root@verbose.hsm: Permission denied (publickey).
```

It is using key based authentication, this is more secure and would go 