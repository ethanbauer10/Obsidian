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

## Feroxbuster
```python
feroxbuster -u http://casino.hsm/ -C 404 --dont-filter

302      GET        5l       22w      199c http://casino.hsm/ => http://casino.hsm/login
302      GET        5l       22w      199c http://casino.hsm/logout => http://casino.hsm/login
200      GET       39l       84w      788c http://casino.hsm/static/css/style.css
200      GET       99l      351w     4778c http://casino.hsm/login
200      GET        2l        9w      171c http://casino.hsm/static/js/app.min.js
302      GET        5l       22w      199c http://casino.hsm/profile => http://casino.hsm/login
302      GET        5l       22w      199c http://casino.hsm/dashboard => http://casino.hsm/login
```

Nothing too interesting

Ill have a look at the javascript file

![](Pasted%20image%2020260821164803.png)

Another reference to a another file ill have a look at

Browsing to `app.min.js.map` downloads a file

```python
cat app.min.js.map                        
{ 
	"version": 3, 
	"file": "app.min.js", 
	"sources": ["src/api/roomVerification.js"], 
	"sourcesContent": [ 
		"// Front-Desk Kiosk API verification helper\nasync function checkRoomStatus(roomNum) {\n const res = await fetch('/api/v1/rooms/status?status=occupied');\n return await res.json();\n}"
  ]
}
```

Looks like there is a request to an API

Ill try curling it and seeing the output

After curling it gives a large JSON response ill use jq to format it nicely

```python
curl http://casino.hsm/api/v1/rooms/status?status=occupied | jq .
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100  11732 100  11732   0      0  40330      0                              0
{
  "filter": "occupied",
  "rooms": [
    {
      "checkout": "2026-08-11",
      "guest_name": "Smith",
      "id": 1,
      "room_number": 105,
      "status": "occupied",
      "tier": "Standard Guest"
    },
    {
      "checkout": "2026-08-23",
      "guest_name": "Johnson",
      "id": 2,
      "room_number": 107,
      "status": "occupied",
      "tier": "Executive Suite"
    },
    {
      "checkout": "2026-08-20",
      "guest_name": "Williams",
      "id": 3,
      "room_number": 108,
      "status": "occupied",
      "tier": "Diamond Club"
    },
    {
      "checkout": "2026-08-26",
      "guest_name": "Brown",
      "id": 4,
      "room_number": 112,
      "status": "occupied",
      "tier": "Standard Guest"
    },
    {
      "checkout": "2026-08-22",
      "guest_name": "Jones",
      "id": 5,
      "room_number": 122,
      "status": "occupied",
      "tier": "VIP Premium"
    },
    {
      "checkout": "2026-08-18",
      "guest_name": "Garcia",
      "id": 6,
      "room_number": 127,
      "status": "occupied",
      "tier": "Standard Guest"
    },
    
...[SNIP]...
```

This endpoint returns all the guests and their room numbers, this info will allow me to login

# Logging into the portal

Im not sure if the login i choose matters, i would guess every guest has the same access

![1043](Pasted%20image%2020260821165503.png)

I am now logged in

![](Pasted%20image%2020260821165708.png)

The only functionality once logged in is in `/profile`

It is vulnerable to SSTI, specifically jinja2 which makes sense since its a python web app

> The reason i wanted to test for this, was simply becuase it was reflecting the preferred display name back to the

# Server Side Template Injection (SSTI)

