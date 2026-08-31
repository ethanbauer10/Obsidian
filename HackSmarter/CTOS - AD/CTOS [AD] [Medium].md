# Objective
CTOS Corporation delivers cutting-edge managed services, cloud solutions, and cybersecurity expertise to clients of all sizes. You have been hired to perform their annual penetration test against 3 high-value targets in the Active Directory environment. Your task is to identify all vulnerabilties and (if possible) elevate your privileges to Domain Admin.

You have been provided VPN access to their internal network, but no other information.

![](Pasted%20image%2020260831140535.png)

# Host file setup
```python
sudo nxc smb 10.1.24.233 --generate-hosts-file /etc/hosts
SMB         10.1.24.233     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:CTOS.CORP) (signing:True) (SMBv1:None) (Null Auth:True)
```




