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
nmap -p- --min-rate=2000 -sT -Pn dc.hsm-defense.local -vv

PORT      STATE SERVICE          REASON
25/tcp    open  smtp             syn-ack
53/tcp    open  domain           syn-ack
80/tcp    open  http             syn-ack
88/tcp    open  kerberos-sec     syn-ack
110/tcp   open  pop3             syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
143/tcp   open  imap             syn-ack
389/tcp   open  ldap             syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
587/tcp   open  submission       syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
3389/tcp  open  ms-wbt-server    syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
47001/tcp open  winrm            syn-ack
49664/tcp open  unknown          syn-ack
49665/tcp open  unknown          syn-ack
49666/tcp open  unknown          syn-ack
49668/tcp open  unknown          syn-ack
49669/tcp open  unknown          syn-ack
49670/tcp open  unknown          syn-ack
49671/tcp open  unknown          syn-ack
49672/tcp open  unknown          syn-ack
49694/tcp open  unknown          syn-ack
49725/tcp open  unknown          syn-ack
```

## Nmap
```python

```