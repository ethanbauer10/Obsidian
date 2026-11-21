# Machine info
As is common in real life pentests, you will start the PingPong box with credentials for the following account c.roberts / AssumedBreach123

```python
c.roberts:AssumedBreach123
```

# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.245.56 --generate-hosts-file /etc/hosts    

SMB         10.129.245.56   445    dc1              [*]  x64 (name:dc1) (domain:ping.htb) (signing:True) (SMBv1:None) (NTLM:False)
```

Since NTLM is disabled ill also generate a realm 

# System time
```python
❯❯❯ ntpdate dc1.ping.htb  
2026-11-22 02:29:19.158508 (+0000) +28799.610798 +/- 0.006356 dc1.ping.htb 10.129.245.56 s1 no-leap
CLOCK: step_systime: Operation not permitted
```

Its running at +8h

# Enumeration
## Open ports
```python

```

## Nmap
```python

```