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


