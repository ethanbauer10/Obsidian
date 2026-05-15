# Objective and scope

An adult beverage company "Martini Bars" recently had a corporate breach and the compliance and risk team dictates they perform a penetration test at one of their branch offices. The Hack Smarter team has been authorized to perform an internal black box pentest.

The client has provided you with VPN access to their internal network, but no credentials.

# Host file setup

```python
sudo nxc smb 10.1.33.56 --generate-hosts-file /etc/hosts
[sudo] password for kali:  
SMB         10.1.33.56      445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:DRY.MARTINI.BARS) (signing:False) (SMBv1:None)
```

- SMB Signing disabled
- Windows 11 server 2025

