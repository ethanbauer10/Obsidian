# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.85.14 --generate-hosts-file /etc/hosts 
 
SMB         10.129.85.14    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:streamIO.htb) (signing:True) (SMBv1:None)
```

# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc.streamio.htb -vv
```

## Nmap
```python

```