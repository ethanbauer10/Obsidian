
# Objective and scope
You are a member of the Hack Smarter Red Team and your organization is beginning to roll out a managed SOC service. You've been provided access to a staging version of the web app before it's pushed to production. > Note from Tyler: Is the typo intentional? Let's just pretend like it is

The credentials below mirror a customer. Are you able to elevate your privileges and become an Administrator?

```python
pentester:HackSmarter123
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.1.1.213
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-03 14:01 -0400
Nmap scan report for 10.1.1.213
Host is up (0.091s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
3000/tcp open  ppp

Nmap done: 1 IP address (1 host up) scanned in 33.87 seconds
```

## NMap
```python

```

# SSH (22)

```python
ssh root@10.1.1.213                   
The authenticity of host '10.1.1.213 (10.1.1.213)' can't be established.
ED25519 key fingerprint is: SHA256:kFBFA1OIn6bgQpPzHIxSFp4osWp/Ep7jERVpIXVkUzs
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.1.1.213' (ED25519) to the list of known hosts.
root@10.1.1.213: Permission denied (publickey).
```

Key absed autn