# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.245.51 --generate-hosts-file /etc/hosts 
SMB         10.129.245.51   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:certificate.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumerationm
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc01.certificate.htb -vv


```

## Nmap
```python

```