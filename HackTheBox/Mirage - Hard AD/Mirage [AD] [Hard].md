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
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn 10.129.232.163 -vv

Nmap scan report for 10.129.232.163
Host is up, received user-set (0.013s latency).
Scanned at 2026-10-10 16:04:09 BST for 11s
Not shown: 65506 closed tcp ports (conn-refused)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
88/tcp    open  kerberos-sec     syn-ack
111/tcp   open  rpcbind          syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
389/tcp   open  ldap             syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
2049/tcp  open  nfs              syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
4222/tcp  open  vrml-multi-use   syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
47001/tcp open  winrm            syn-ack
49664/tcp open  unknown          syn-ack
49665/tcp open  unknown          syn-ack
49666/tcp open  unknown          syn-ack
49667/tcp open  unknown          syn-ack
49668/tcp open  unknown          syn-ack
51462/tcp open  unknown          syn-ack
51463/tcp open  unknown          syn-ack
51478/tcp open  unknown          syn-ack
51484/tcp open  unknown          syn-ack
51508/tcp open  unknown          syn-ack
58086/tcp open  unknown          syn-ack
62970/tcp open  unknown          syn-ack
```

## Nmap
```python

```