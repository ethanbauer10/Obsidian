# Host file setup
```python
sudo nxc smb 10.129.12.5 --generate-hosts-file /etc/hosts 
[sudo] password for kali: 
SMB         10.129.12.5     445    BLN01            [*] Windows Server 2008 R2 Datacenter 7601 Service Pack 1 x64 (name:BLN01) (domain:retro2.vl) (signing:True) (SMBv1:True) (Null Auth:True)
```
Added an entry into `/etc/hosts`

SMB signing is enabled

SMBv1 

Null auth enabled