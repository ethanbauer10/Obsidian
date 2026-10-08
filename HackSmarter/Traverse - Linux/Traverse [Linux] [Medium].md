# Lab description
You’ve been brought in to conduct an internal penetration test targeting a highly sensitive Linux server. To demonstrate the real-world business impact of a breach, the client has hidden two flags on the system. Your objective: compromise the server and extract both flags.

The client has provided you with VPN access to their environment, but no other information.

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.0.20.19 -vv

Nmap scan report for 10.0.20.19
Host is up, received user-set (0.096s latency).
Scanned at 2026-10-08 18:05:45 BST for 65s
Not shown: 65486 filtered tcp ports (no-response)
PORT      STATE  SERVICE REASON
21/tcp    open   ftp     syn-ack
22/tcp    open   ssh     syn-ack
80/tcp    open   http    syn-ack
```

## Nmap
```python

```

# SSH (22)
## Version info
```python

```

## Auth method
```python

```

