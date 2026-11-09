# Machine info
As is common in real life pentests, you will start the Garfield box with credentials for the following account j.arbuckle / Th1sD4mnC4t!@1978

```python
j.arbuckle:Th1sD4mnC4t!@1978
```

# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.86.27 --generate-hosts-file /etc/hosts

SMB         10.129.86.27    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:garfield.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Opn