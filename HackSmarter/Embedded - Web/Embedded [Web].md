# Objective and scope
Hack Smarter is releasing a new SaaS platform for cloud storage. You have been hired to perform a full penetration test on the platform. The web app is in-scope, the client has also placed a "flag" on the machine. If this flag is retrieved, that is an immediate early disclosure as it demonstrates full compromise of the application.

The client has provisioned you with a standard user account

```python
Username: brian
Password: HackSmarter123!
```

During OSINT, your team identified a leaked password but are unsure of who it belongs to:

```python
cJ2yxWMs3XEHbO
```

Finally, they also gathered a list of potential usernames

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.1.74.45              
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-21 18:17 +0100
Stats: 0:00:25 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 75.42% done; ETC: 18:17 (0:00:08 remaining)
Nmap scan report for 10.1.74.45
Host is up (0.096s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
2222/tcp open  EtherNetIP-1
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 33.72 seconds
```

# HTTP (8080)

![](Pasted%20image%2020260821181936.png)

Landing page is a login form

`admin:admin` returns `Invalid username or password`, so no username enum from the error

![](Pasted%20image%2020260821182059.png)

![](Pasted%20image%2020260821182159.png)

The provided creds for `brian` got me logged in

![](Pasted%20image%2020260821184305.png)

There is an option to update the users, i could spray this paramter with the username list, i would assume the app wont let me modify the username to be the same as an existing user

# Username enumeration via the `update_profile` endpoint

First ill proxy the request to caido and send the request to automate

```python

```