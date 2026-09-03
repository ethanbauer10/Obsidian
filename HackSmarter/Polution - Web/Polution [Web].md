
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
nmap -p 22,3000 -A --min-rate=2000 -sT 10.1.1.213
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-03 14:02 -0400
Nmap scan report for 10.1.1.213
Host is up (0.091s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 84:54:fd:05:a7:15:3e:b7:95:96:b9:f1:d1:13:ea:07 (ECDSA)
|_  256 ee:7a:63:67:d9:4f:f4:f2:65:bc:37:38:08:a0:81:16 (ED25519)
3000/tcp open  http    Node.js Express framework
|_http-title: Hacksmarter | Login
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
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

Key absed auth, more secure

# HTTP (3000)

![](Pasted%20image%2020260903140436.png)


