# Objective and scope

ShadowGate provides cybersecurity solutions for global enterprises. They are in the process of getting SOC 2 certified, and have hired Hack Smarter to perform an internal network penetration test. Find all vulnerabilities and, if possible, elevate your privileges to Domain Admin.

You have been provided with VPN access to their network, but no other information.

# Host file setup

```python
sudo nxc smb 10.1.232.232 --generate-hosts-file /etc/hosts                                    
[sudo] password for kali: 
SMB         10.1.232.232    445    SG-DC01          [*] Windows 10 / Server 2019 Build 17763 x64 (name:SG-DC01) (domain:shadowgate.local) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration

## Open ports
```python
nmap -p- --min-rate=1000 -sT sg-dc01.shadowgate.local
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-31 17:09 +0100
Stats: 0:02:06 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 97.77% done; ETC: 17:12 (0:00:03 remaining)
Nmap scan report for sg-dc01.shadowgate.local (10.1.232.232)
Host is up (0.095s latency).
rDNS record for 10.1.232.232: SG-DC01.shadowgate.local
Not shown: 65521 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
593/tcp   open  http-rpc-epmap
1433/tcp  open  ms-sql-s
3268/tcp  open  globalcatLDAP
3389/tcp  open  ms-wbt-server
9389/tcp  open  adws
49665/tcp open  unknown
49669/tcp open  unknown
49694/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 129.76 seconds
```

Also port 389 and 5985 are open and nmap missed them!

## Nmap
```python

```