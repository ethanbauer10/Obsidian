# Machine info
As is common in real life pentests, you will start the Checkpoint box with credentials for the following account alex.turner / Checkpoint2024!

```python
alex.turner:Checkpoint2024!
```

# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.80.26 --generate-hosts-file /etc/hosts    
SMB         10.129.80.26    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:checkpoint.htb) (signing:True) (SMBv1:None)
```

# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc01.checkpoint.htb -vv


```

## Nmap
```python

```

