# Objective and scope
You have been hired to perform a red team engagement against Trask Industries. Trask is a forward-thinking technology firm, with a heavy focus on research and development (R&D).

Trask Industries is not aware of the engagement; instead, you have been hired as a new employee in the R&D department. Your task is to begin as a "new employee" and demonstrate impact by elevating your privileges to Domain Admin (if possible).

I have also been provided a onboarding document that i will use for initial access

Also given a wordlist for any hashcracking

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.0.0.5 -vv

Completed Connect Scan at 16:17, 459.70s elapsed (65535 total ports)
Nmap scan report for dc01 (10.0.0.5)
Host is up, received user-set (0.092s latency).
Scanned at 2026-09-27 16:09:38 BST for 460s
Not shown: 65516 filtered tcp ports (no-response)
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
45985/tcp open  unknown          syn-ack
49664/tcp open  unknown          syn-ack
52200/tcp open  unknown          syn-ack
52207/tcp open  unknown          syn-ack
52224/tcp open  unknown          syn-ack
52234/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 459.73 seconds
```

## NMap
```python

```

![](Pasted%20image%2020260927160217.png)

I have been given a username and a password, which i will use

```python
k.pryde:KP_TempPass_1988!
```

