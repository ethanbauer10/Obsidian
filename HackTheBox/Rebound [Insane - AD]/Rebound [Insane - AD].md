# Host file setup
```python
sudo nxc smb 10.129.14.17 --generate-hosts-file /etc/hosts                                     
[sudo] password for kali: 
SMB         10.129.14.17    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```
Few things note:
- Null auth is true
- SMB signing is enabled

# Enumeration
## Open ports
```p
```