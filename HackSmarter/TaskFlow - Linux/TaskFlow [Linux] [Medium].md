# Lab description
You have been hired to perform a penetration test against the client's development infrastructure. The dev team relies heavily on a project management application, which they have provided you access to.

Your task is to start as an unauthenticated attacker, identify all vulnerabilities, and demonstrate full impact by compromising the underlying host (if possible).

The client has provided you with VPN access to their environment, but no other information.

# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn 10.0.22.103 -vv

Nmap scan report for 10.0.22.103
Host is up, received user-set (0.11s latency).
Scanned at 2026-10-08 17:42:19 BST for 66s
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack
```

## Nmap
```python

```

# HTTP (80)
