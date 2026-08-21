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
nmap -p 22,80,2222 -A --min-rate=2000 -sT 10.0.16.132          
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-21 16:31 +0100
Nmap scan report for 10.0.16.132
Host is up (0.095s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.18 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 50:b1:7c:7a:d5:3b:c0:83:a8:c7:37:7b:21:b3:b5:9e (ECDSA)
|_  256 cb:8e:b7:cc:bd:a8:2c:5a:e5:c4:cb:8c:60:d3:d3:d3 (ED25519)
80/tcp   open  http    Werkzeug httpd 3.1.8 (Python 3.10.18)
|_http-server-header: Werkzeug/3.1.8 Python/3.10.18
| http-title: Hack Smarter World - Guest WiFi & Portal
|_Requested resource was /login
2222/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u7 (protocol 2.0)
| ssh-hostkey: 
|   3072 7d:c5:f5:ba:03:3e:f0:76:5c:9d:47:b6:39:b5:c7:a4 (RSA)
|   256 ed:5d:fa:ea:74:a0:56:b1:39:59:fc:c5:22:1e:5e:bd (ECDSA)
|_  256 50:31:d9:54:80:42:b8:44:cb:40:66:ea:cf:8f:cf:37 (ED25519)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 (96%), HP P2000 G3 NAS device (93%), Linux 2.6.32 - 3.13 (93%), Linux 5.0 - 5.14 (93%), Linux 5.14 - 6.8 (93%), Linux 3.2 - 4.14 (93%), Linux 4.15 - 5.19 (93%), Linux 2.6.32 - 3.10 (92%), Linux 5.4 - 5.15 (92%), Linux 3.10 - 4.11 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Ill create an entry in `/etc/hosts` for `casino.hsm` since it doesnt look like there is one setup
# SSH (22)
## Auth method
```python
ssh root@10.0.16.132                  
The authenticity of host '10.0.16.132 (10.0.16.132)' can't be established.
ED25519 key fingerprint is: SHA256:xNA8ct5SST3a/v1BCwF46Mf9QnDdLMnEET/t37F26gU
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.16.132' (ED25519) to the list of known hosts.
root@10.0.16.132: Permission denied (publickey).
```

Key based authentication

# SSH (2222)
## Auth method
```python
ssh root@10.0.16.132 -p 2222
The authenticity of host '[10.0.16.132]:2222 ([10.0.16.132]:2222)' can't be established.
ED25519 key fingerprint is: SHA256:TF+GAFGmxDZ9jQQKJMaZoJ/D+UdsPgR9P2sa6YiJswM
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.0.16.132]:2222' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
root@10.0.16.132's password:
```

This i assume is a docker container, this does allow password based auth

# HTTP (80)

![487](Pasted%20image%2020260821164006.png)

So it looks like its taking a last name and a room number, could be vulnerable to a brute force

![](Pasted%20image%2020260821163940.png)

The application is running on flask

There are no subdomains 



