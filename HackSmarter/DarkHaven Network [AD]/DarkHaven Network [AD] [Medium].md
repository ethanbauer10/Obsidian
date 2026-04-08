![](Pasted%20image%2020260408101620.png)
This is the layout for the network

# Objective and scope
Darkhaven Technologies is a networking organization based throughout the world with locations in NY, CA, Japan, and more. They have segregated their network and would like to do a Red Team engagement to see if a user is able to move throughout the different networks.

A Close Access Team has infiltrated Darkhaven Technologies and dropped a machine for you on the internal network that you can connect to through OpenVPN. This machine should allow you to see the entire global network, as it was dropped on a port that is within the global VLAN. The Close Access Team relayed information that they overheard about the Web Portal being worked on at this time.

# Target Hosts
**dc.darkhaven.tech** - 10.10.10.4
**dc02.corp.darkhaven.tech** - 10.10.10.5
**web.ext.darkhaven.local** - 10.10.10.132
**ca.ext.darkhaven.local** - 10.10.10.134
**share.ext.darhaven.local** - 10.10.10.135
**sql.ext.darkhaven.local** - 10.10.10.133
**dc.ext.darkhaven.local** - 10.10.10.136
**VPN** - 10.10.10.137

# Enumeration
From the objective and scope its giving me a hint that the web server may be my initial access vector

## web.ext.darkhaven.local

### Open ports
```python
nmap -p- --min-rate=2000 -sT web.ext.darkhaven.local -Pn
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-08 10:28 -0400
Nmap scan report for web.ext.darkhaven.local (10.10.10.132)
Host is up (0.094s latency).
Not shown: 65528 filtered tcp ports (no-response)
PORT      STATE SERVICE
80/tcp    open  http
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
2107/tcp  open  msmq-mgmt
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
49680/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.71 seconds
```
### Nmap
```python
nmap -p 80,139,445,2107,3389,5985 -A --min-rate=2000 -sT -Pn web.ext.darkhaven.local                          
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-08 10:31 -0400
Nmap scan report for web.ext.darkhaven.local (10.10.10.132)
Host is up (0.094s latency).

PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Darkhaven Technologies \xE2\x80\x93 Secure Network Solutions
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
2107/tcp open  msrpc         Microsoft Windows RPC
3389/tcp open  ms-wbt-server
| ssl-cert: Subject: commonName=EC2AMAZ-IKFPL26.ext.darkhaven.local
| Not valid before: 2026-02-26T01:17:22
|_Not valid after:  2026-08-28T01:17:22
|_ssl-date: TLS randomness does not represent time
| rdp-ntlm-info: 
|   Target_Name: DARKHAVEN
|   NetBIOS_Domain_Name: DARKHAVEN
|   NetBIOS_Computer_Name: EC2AMAZ-IKFPL26
|   DNS_Domain_Name: ext.darkhaven.local
|   DNS_Computer_Name: EC2AMAZ-IKFPL26.ext.darkhaven.local
|   DNS_Tree_Name: ext.darkhaven.local
|   Product_Version: 10.0.26100
|_  System_Time: 2026-04-08T14:32:31+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3389-TCP:V=7.98%I=7%D=4/8%Time=69D666C6%P=x86_64-pc-linux-gnu%r(Ter
SF:minalServerCookie,13,"\x03\0\0\x13\x0e\xd0\0\0\x124\0\x02\?\x08\0\x02\0
SF:\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

# HTTP (80) - web.ext.darkhaven.local
## Feroxbuster
```python
feroxbuster -u http://web.ext.darkhaven.local/ -C 404

301      GET        2l       10w      161c http://web.ext.darkhaven.local/images => http://web.ext.darkhaven.local/images/
301      GET        2l       10w      158c http://web.ext.darkhaven.local/css => http://web.ext.darkhaven.local/css/
200      GET      931l     2345w    23449c http://web.ext.darkhaven.local/css/site.css
200      GET      363l     1058w    16821c http://web.ext.darkhaven.local/default.aspx
301      GET        2l       10w      161c http://web.ext.darkhaven.local/Images => http://web.ext.darkhaven.local/Images/
200      GET       83l      200w     3443c http://web.ext.darkhaven.local/login.aspx
200      GET      363l     1058w    16821c http://web.ext.darkhaven.local/
301      GET        2l       10w      158c http://web.ext.darkhaven.local/CSS => http://web.ext.darkhaven.local/CSS/
301      GET        2l       10w      158c http://web.ext.darkhaven.local/Css => http://web.ext.darkhaven.local/Css/
301      GET        2l       10w      161c http://web.ext.darkhaven.local/IMAGES => http://web.ext.darkhaven.local/IMAGES/
```
## Ffuf for vhosts
```python
nothing found
```
## Website functionality
Ill start by testing the `login.aspx` page

![](Pasted%20image%2020260408104021.png)
It looks like the portal is using domain credentials rather than creds that are specific to the website

There is also a `continue as guest` option

Found a user `it-helpdesk`

![1090](Pasted%20image%2020260408104246.png)
Found a user `sql_svc`

![](Pasted%20image%2020260408104502.png)
Using the search `Look up user sql_svc` i got an initial password

```python
sql_svc:SqLS3rvic3!
```