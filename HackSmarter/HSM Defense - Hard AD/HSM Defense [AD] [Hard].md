# Objective and initial access
HSM Defense is a defense contractor, and is required to have an in-depth penetration test done against their internal Domain Controller. You have been hired to perform a comprehensive penetration test and, if possible, elevate your privileges to Domain Admin to demonstrate impact.

You have been provided credentials for a low-privileged Active Directory user.

```python
Username: kelly.johnson
Password: Lordofwar
```

# Host file setup
```python
sudo nxc smb 10.1.22.99 --generate-hosts-file /etc/hosts                     
[sudo] password for kali: 
SMB         10.1.22.99      445    DC               [*]  x64 (name:DC) (domain:hsm-defense.local) (signing:True) (SMBv1:None) (NTLM:False)
```

NTLM is disabled

# Enumeration
## Open ports
```python

```

## Nmap
```python

```