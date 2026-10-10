# Host file setup
```python
❯❯❯ sudo nxc smb 10.129.232.163 --generate-hosts-file /etc/hosts     

SMB         10.129.232.163  445    dc01             [*]  x64 (name:dc01) (domain:mirage.htb) (signing:True) (SMBv1:None) (NTLM:False)
```

# Setting up kerberos realm
```python
❯❯❯ sudo nxc smb 10.129.232.163 --generate-krb5-file /etc/krb5.conf 
SMB         10.129.232.163  445    dc01             [*]  x64 (name:dc01) (domain:mirage.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         10.129.232.163  445    dc01             [+] krb5 conf saved to: /etc/krb5.conf
SMB         10.129.232.163  445    dc01             [+] Run the following command to use the conf file: export KRB5_CONFIG=/etc/krb5.conf
```

Since NTLM is disabled, ill be interacting with kerberos so ill set this now

# Enumeration
## Open ports
```python

```

## Nmap
``