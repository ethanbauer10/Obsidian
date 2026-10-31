# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn 10.129.238.35 -vv

Host is up, received user-set (0.014s latency).
Scanned at 2026-10-31 19:29:56 GMT for 197s
Not shown: 65508 filtered tcp ports (no-response)
PORT      STATE SERVICE       REASON
22/tcp    open  ssh           syn-ack
25/tcp    open  smtp          syn-ack
80/tcp    open  http          syn-ack
111/tcp   open  rpcbind       syn-ack
135/tcp   open  msrpc         syn-ack
139/tcp   open  netbios-ssn   syn-ack
443/tcp   open  https         syn-ack
445/tcp   open  microsoft-ds  syn-ack
1063/tcp  open  kyoceranetdev syn-ack
2049/tcp  open  nfs           syn-ack
3389/tcp  open  ms-wbt-server syn-ack
5985/tcp  open  wsman         syn-ack
6160/tcp  open  ecmp          syn-ack
6161/tcp  open  patrol-ism    syn-ack
6162/tcp  open  patrol-coll   syn-ack
6169/tcp  open  unknown       syn-ack
6170/tcp  open  unknown       syn-ack
6190/tcp  open  unknown       syn-ack
6210/tcp  open  unknown       syn-ack
6290/tcp  open  unknown       syn-ack
10001/tcp open  scp-config    syn-ack
10002/tcp open  documentum    syn-ack
10003/tcp open  documentum_s  syn-ack
10005/tcp open  stel          syn-ack
10006/tcp open  netapp-sync   syn-ack
11731/tcp open  unknown       syn-ack
49669/tcp open  unknown       syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 197.42 seconds
```

## Nmap
```python
❯❯❯ nmap -p 22,25,80,111,135,139,443,445,1063,2049,3389,5985,6160,6161,6162,6169,6170,6190,6210,6290,10001,10002,10003,10005,10006,11731 -A --min-rate=2000 -sT -Pn 10.129.238.35 
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-31 19:36 +0000
Nmap scan report for 10.129.238.35
Host is up (0.014s latency).

PORT      STATE SERVICE              VERSION
22/tcp    open  ssh                  OpenSSH for_Windows_9.5 (protocol 2.0)
25/tcp    open  smtp                 hMailServer smtpd
| smtp-commands: JOB2, SIZE 20480000, AUTH LOGIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp    open  http                 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
111/tcp   open  rpcbind
135/tcp   open  msrpc                Microsoft Windows RPC
139/tcp   open  netbios-ssn          Microsoft Windows netbios-ssn
443/tcp   open  ssl/https?
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=www.job2.vl
| Subject Alternative Name: DNS:job2.vl, DNS:www.job2.vl
| Not valid before: 2023-05-09T13:31:40
|_Not valid after:  2122-05-09T13:41:37
| tls-alpn: 
|   h2
|_  http/1.1
445/tcp   open  microsoft-ds?
1063/tcp  open  rpcbind
2049/tcp  open  rpcbind
3389/tcp  open  ms-wbt-server        Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: JOB2
|   NetBIOS_Domain_Name: JOB2
|   NetBIOS_Computer_Name: JOB2
|   DNS_Domain_Name: JOB2
|   DNS_Computer_Name: JOB2
|   Product_Version: 10.0.20348
|_  System_Time: 2026-10-31T20:39:07+00:00
| ssl-cert: Subject: commonName=JOB2
| Not valid before: 2026-10-30T20:28:01
|_Not valid after:  2027-05-01T20:28:01
|_ssl-date: 2026-10-31T20:39:38+00:00; +59m53s from scanner time.
5985/tcp  open  http                 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
6160/tcp  open  msrpc                Microsoft Windows RPC
6161/tcp  open  msrpc                Microsoft Windows RPC
6162/tcp  open  msrpc                Microsoft Windows RPC
6169/tcp  open  tcpwrapped
6170/tcp  open  storagecraft-image   StorageCraft Image Manager
6190/tcp  open  msrpc                Microsoft Windows RPC
6210/tcp  open  msrpc                Microsoft Windows RPC
6290/tcp  open  unknown
10001/tcp open  msexchange-logcopier Microsoft Exchange 2010 log copier
10002/tcp open  msexchange-logcopier Microsoft Exchange 2010 log copier
10003/tcp open  storagecraft-image   StorageCraft Image Manager
10005/tcp open  stel?
10006/tcp open  netapp-sync?
11731/tcp open  msrpc                Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: JOB2; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SSH (22)

![](Pasted%20image%2020261031193222.png)

Password based auth

# SMTP (25)

```python
❯❯❯ telnet 10.129.238.35 25                                
Trying 10.129.238.35...
Connected to 10.129.238.35.
Escape character is '^]'.
220 JOB2 ESMTP
HELO 10.129.238.35  
250 Hello.
AUTH NTLM 334
504 Authentication mechanism not supported.
```

It doesnt support NTLM

Not a whole lot i can do with SMTP at the moment

# HTTP (80)

It looks like both port 80 and 443 are running the same service

![](Pasted%20image%2020261031194828.png)

![](Pasted%20image%2020261031194901.png)

Looks like i might have a user i might be able send an email to?



