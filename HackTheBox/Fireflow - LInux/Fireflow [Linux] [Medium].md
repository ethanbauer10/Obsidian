# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.48.171
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-22 17:02 +0100
Nmap scan report for 10.129.48.171
Host is up (0.018s latency).
Not shown: 62762 closed tcp ports (conn-refused), 2771 filtered tcp ports (no-response)
PORT    STATE SERVICE
22/tcp  open  ssh
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 14.11 seconds
```

## Nmap
```python
nmap -p 22,443 -A --min-rate=2000 -sT 10.129.48.171  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-22 17:04 +0100
Nmap scan report for fireflow.htb (10.129.48.171)
Host is up (0.013s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
443/tcp open  ssl/http nginx
|_http-title: FireFlow \xE2\x80\x94 Task Force Nightfall
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|   http/1.1
|   http/1.0
|_  http/0.9
| ssl-cert: Subject: commonName=fireflow.htb/organizationName=Task Force Nightfall/countryName=US
| Subject Alternative Name: DNS:fireflow.htb, DNS:*.fireflow.htb
| Not valid before: 2026-04-14T16:35:31
|_Not valid after:  2028-07-17T16:35:31
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Nmap detects the domain `fireflow.htb`

# SSH (22)
## Auth method
```python
ssh root@fireflow.htb                   
The authenticity of host 'fireflow.htb (10.129.48.171)' can't be established.
ED25519 key fingerprint is: SHA256:OZNUeTZ9jastNKKQ1tFXatbeOZzSFg5Dt7nhwhjorR0
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'fireflow.htb' (ED25519) to the list of known hosts.
root@fireflow.htb's password:
```

Password based authentication

# HTTP (443)

