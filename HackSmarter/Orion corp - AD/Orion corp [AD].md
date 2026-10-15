# Objective and scope
Orion Corp has hired you to perform an internal penetration test against two machines within their network. Your task is to identify all vulnerabilities and (if possible) elevate your privileges to Domain Admin.

The client has provided you with VPN access to their network, but no credentials.

![806](Pasted%20image%2020261015174115.png)

There are two hosts in scope

```python
❯❯❯ ping 10.0.0.6
PING 10.0.0.6 (10.0.0.6) 56(84) bytes of data.
64 bytes from 10.0.0.6: icmp_seq=1 ttl=127 time=94.1 ms

❯❯❯ ping 10.0.0.5     
PING 10.0.0.5 (10.0.0.5) 56(84) bytes of data.
64 bytes from 10.0.0.5: icmp_seq=1 ttl=127 time=93.9 ms
```

I can reach both hosts, so likely no tunneling in this instance

# Enumeration
## WS-IT (10.0.0.6)

### Open ports
```python
❯❯❯ nmap -p- --min-rate=1000 -sT -Pn 10.0.0.6 -vv 

Nmap scan report for 10.0.0.6
Host is up, received user-set (0.093s latency).
Scanned at 2026-10-15 17:45:44 BST for 128s
Not shown: 65529 filtered tcp ports (no-response)
PORT      STATE SERVICE       REASON
135/tcp   open  msrpc         syn-ack
139/tcp   open  netbios-ssn   syn-ack
445/tcp   open  microsoft-ds  syn-ack
3389/tcp  open  ms-wbt-server syn-ack
5985/tcp  open  wsman         syn-ack
49674/tcp open  unknown       syn-ack
```

### Nmap
```python

```

## DC01 (10.0.0.5)
### Open ports
```python

```

### Nmap
```python

```

# Host file setup on both machines

```python

```