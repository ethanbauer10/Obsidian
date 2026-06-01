
# Host file setup

```python
sudo nxc smb 10.129.234.48 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.234.48   445    DC-JPQ225        [*]  x64 (name:DC-JPQ225) (domain:cicada.vl) (signing:True) (SMBv1:None) (NTLM:False)
```
- NTLM auth disabled
- SMB signing is enabled

# Enumeration

## 