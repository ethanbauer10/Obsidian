# Objective and scope
You have been assigned a penetration test of a client's internal network. There are three different servers in-scope. Your task is to identify all vulnerabilities and demonstrate impact to the client by elevating your access to Domain Admin. The client has provided you VPN access and Active Directory credentials (below) for the engagement.

Please note - there are 2 intended paths to compromise this machine. Once you solve it one way, we encourage you to try and solve it a second way.

# Hosts
DC1 - **10.0.16.4**
DC2 - **10.0.18.13**
Nexus - **10.0.27.179**

# Initial Access

The client has provided you with Active Directory credentials and VPN access. `xiao.ge:AmBZATVjnH4qo8H4`

# Host file setup
```python
sudo nxc smb 10.0.16.4 --generate-hosts-file /etc/hosts    
[sudo] password for kali: 
SMB         10.0.16.4       445    DC1              [*] Windows Server 2022 Build 20348 x64 (name:DC1) (domain:dismay.hsm) (signing:True) (SMBv1:None) (Null Auth:True)


sudo nxc smb 10.0.18.13 --generate-hosts-file /etc/hosts
SMB         10.0.18.13      445    DC2              [*] Windows Server 2022 Build 20348 x64 (name:DC2) (domain:dismay.hsm) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
