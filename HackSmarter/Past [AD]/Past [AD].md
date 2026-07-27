
# Objective and Scope

You have been hired by Hack Smarter to perform a Penetration Test on Past Systems Inc. During your call with the client, they stated they are currently adding new machines to the network.

The client has provided you with VPN access to their internal network, but no credentials.

# Host file setup

```python
sudo nxc smb 10.0.16.121 --generate-hosts-file /etc/hosts                             
SMB         10.0.16.121     445    EC2AMAZ-A5O4OL8  [*] Windows Server 2016 Datacenter 14393 x64 (name:EC2AMAZ-A5O4OL8) (domain:past.local) (signing:True) (SMBv1:True) (Null Auth:True)
```

Looks as if its running an old version of Windows

And its also running SMB v1?

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT EC2AMAZ-A5O4OL8.past.local       
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-27 16:43 +0100
Nmap scan report for EC2AMAZ-A5O4OL8.past.local (10.0.16.121)
Host is up (0.095s latency).
Not shown: 65520 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
593/tcp   open  http-rpc-epmap
3389/tcp  open  ms-wbt-server
47001/tcp open  winrm
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49700/tcp open  unknown
49705/tcp open  unknown
57538/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.93 seconds
```

## Nmap
```python

```