
# Objective and scope
NovaForge delivers enterprise software, intelligent automation, and internal management platforms targeted towards mid-sized companies. They have hired Hack Smarter to perform an internal network penetration test, with a special focus on Active Directory.

Your task is to enumerate the network and see if you can fully compromise the domain.

The client has provided you with VPN access to their network, but no credentials.

# Host file setup
```python
sudo nxc smb 10.0.0.100 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.0.0.100      445    DC               [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC) (domain:novaforge.local) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## DC
### Open ports
```python

```

### Nmap
```python

```

## Storage server
### Open ports
```python

```

### Nmap
```python

```