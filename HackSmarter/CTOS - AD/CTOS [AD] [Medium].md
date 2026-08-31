# Objective
CTOS Corporation delivers cutting-edge managed services, cloud solutions, and cybersecurity expertise to clients of all sizes. You have been hired to perform their annual penetration test against 3 high-value targets in the Active Directory environment. Your task is to identify all vulnerabilties and (if possible) elevate your privileges to Domain Admin.

You have been provided VPN access to their internal network, but no other information.

![](Pasted%20image%2020260831140535.png)

# Host file setup
```python
sudo nxc smb 10.1.24.233 --generate-hosts-file /etc/hosts
SMB         10.1.24.233     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:CTOS.CORP) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration

Ill start by making a hosts.txt file with all IPs inside

## Open ports
```python
nmap -p- --min-rate=2000 -sT -iL hosts.txt -Pn
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-31 14:09 -0400
Warning: 10.1.61.204 giving up on port because retransmission cap hit (10).
Warning: 10.1.24.233 giving up on port because retransmission cap hit (10).
Nmap scan report for DC01.CTOS.CORP (10.1.24.233)
Host is up (0.14s latency).
Not shown: 64816 closed tcp ports (conn-refused), 691 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
49674/tcp open  unknown
49675/tcp open  unknown
49688/tcp open  unknown
49708/tcp open  unknown
49716/tcp open  unknown
49721/tcp open  unknown
49805/tcp open  unknown

Nmap scan report for 10.1.206.115
Host is up (0.097s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT     STATE SERVICE
135/tcp  open  msrpc
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
5357/tcp open  wsdapi
5985/tcp open  wsman

Nmap scan report for 10.1.61.204
Host is up (0.096s latency).
Not shown: 64638 closed tcp ports (conn-refused), 895 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 3 IP addresses (3 hosts up) scanned in 213.67 seconds
```

## Nmap
```python

```

# HTTP (80) WEB-01

![](Pasted%20image%2020260831142035.png)

The about page holds some users

![](Pasted%20image%2020260831142129.png)

```python
marcus chen
elena rodriguez
james wilson
sarah mitchell
david park
lisa conrad
```

I could potentially run these users against username anarchy to generate some combinations then user kerburte to vaidate





