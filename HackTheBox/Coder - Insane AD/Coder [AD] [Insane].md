# Host file setup

```python
❯❯❯ sudo nxc smb 10.129.229.190 --generate-hosts-file /etc/hosts                     
[sudo] password for kali: 
SMB         10.129.229.190  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:coder.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration
## Open ports
```python
❯❯❯ nmap -p- --min-rate=2000 -sT -Pn dc01.coder.htb -vv

Nmap scan report for dc01.coder.htb (10.129.229.190)
Host is up, received user-set (0.013s latency).
rDNS record for 10.129.229.190: DC01.coder.htb
Scanned at 2026-11-13 17:09:44 GMT for 12s
Not shown: 65508 closed tcp ports (conn-refused)
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack
80/tcp    open  http             syn-ack
88/tcp    open  kerberos-sec     syn-ack
135/tcp   open  msrpc            syn-ack
139/tcp   open  netbios-ssn      syn-ack
389/tcp   open  ldap             syn-ack
443/tcp   open  https            syn-ack
445/tcp   open  microsoft-ds     syn-ack
464/tcp   open  kpasswd5         syn-ack
593/tcp   open  http-rpc-epmap   syn-ack
636/tcp   open  ldapssl          syn-ack
3268/tcp  open  globalcatLDAP    syn-ack
3269/tcp  open  globalcatLDAPssl syn-ack
5985/tcp  open  wsman            syn-ack
9389/tcp  open  adws             syn-ack
47001/tcp open  winrm            syn-ack
49664/tcp open  unknown          syn-ack
49665/tcp open  unknown          syn-ack
49666/tcp open  unknown          syn-ack
49667/tcp open  unknown          syn-ack
49669/tcp open  unknown          syn-ack
49670/tcp open  unknown          syn-ack
49671/tcp open  unknown          syn-ack
49677/tcp open  unknown          syn-ack
49680/tcp open  unknown          syn-ack
49688/tcp open  unknown          syn-ack
49692/tcp open  unknown          syn-ack

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 11.99 seconds
```

## Nmap
```python
❯❯❯ nmap -p 53,80,88,135,139,389,443,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT -Pn dc01.coder.htb                 
Starting Nmap 7.991 ( https://nmap.org ) at 2026-11-13 17:10 +0000
Nmap scan report for dc01.coder.htb (10.129.229.190)
Host is up (0.013s latency).
rDNS record for 10.129.229.190: DC01.coder.htb

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-11-14 01:10:24Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: coder.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.coder.htb, DNS:coder.htb, DNS:CODER
| Not valid before: 2023-11-21T23:06:46
|_Not valid after:  2033-11-21T23:16:46
|_ssl-date: 2026-11-14T01:11:12+00:00; +7h59m32s from scanner time.
443/tcp  open  ssl/https?
| ssl-cert: Subject: commonName=default-ssl/organizationName=HTB/stateOrProvinceName=CA/countryName=US
| Not valid before: 2022-11-04T17:25:43
|_Not valid after:  2032-11-01T17:25:43
| tls-alpn: 
|   h2
|_  http/1.1
|_ssl-date: 2026-11-14T01:11:12+00:00; +7h59m32s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: coder.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-11-14T01:11:12+00:00; +7h59m32s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.coder.htb, DNS:coder.htb, DNS:CODER
| Not valid before: 2023-11-21T23:06:46
|_Not valid after:  2033-11-21T23:16:46
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: coder.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-11-14T01:11:12+00:00; +7h59m32s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.coder.htb, DNS:coder.htb, DNS:CODER
| Not valid before: 2023-11-21T23:06:46
|_Not valid after:  2033-11-21T23:16:46
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: coder.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-11-14T01:11:12+00:00; +7h59m32s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.coder.htb, DNS:coder.htb, DNS:CODER
| Not valid before: 2023-11-21T23:06:46
|_Not valid after:  2033-11-21T23:16:46
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10|11|2012|2022|2016 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1909 - 2004 (96%), Microsoft Windows 10 1709 - 22H2 (94%), Microsoft Windows 10 1909 (92%), Microsoft Windows 11 24H2 - 25H2 (92%), Microsoft Windows Server 2012 R2 (92%), Microsoft Windows Server 2022 (92%), Microsoft Windows Server 2016 (90%), Microsoft Windows 10 21H2 (90%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)

Null auth is enabled but cannot use it to enumerate

The guest account is enabled

```python
❯❯❯ nxc smb dc01.coder.htb -u 'Guest' -p ''             
SMB         10.129.229.190  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:coder.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.229.190  445    DC01             [+] coder.htb\Guest:
```

```python
❯❯❯ nxc smb dc01.coder.htb -u 'Guest' -p '' --shares
SMB         10.129.229.190  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:coder.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.229.190  445    DC01             [+] coder.htb\Guest: 
SMB         10.129.229.190  445    DC01             [*] Enumerated shares
SMB         10.129.229.190  445    DC01             Share           Permissions     Remark
SMB         10.129.229.190  445    DC01             -----           -----------     ------
SMB         10.129.229.190  445    DC01             ADMIN$                          Remote Admin
SMB         10.129.229.190  445    DC01             C$                              Default share
SMB         10.129.229.190  445    DC01             Development     READ            
SMB         10.129.229.190  445    DC01             IPC$            READ            Remote IPC
SMB         10.129.229.190  445    DC01             NETLOGON                        Logon server share 
SMB         10.129.229.190  445    DC01             SYSVOL                          Logon server share 
SMB         10.129.229.190  445    DC01             Users           READ          
```

This user has read on `development` and `users`

```python
❯❯❯ nxc smb dc01.coder.htb -u 'Guest' -p '' --rid-brute 20000 | grep SidTypeUser | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
e.black
c.cage
j.briggs
l.kang
s.blade
svc_teamcity
```

Ill also grab the users

# Development share

```python
❯❯❯ smbclient //dc01.coder.htb/Development -U 'Guest'%''
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Thu Nov  3 15:16:25 2022
  ..                                  D        0  Thu Nov  3 15:16:25 2022
  Migrations                          D        0  Tue Nov  8 22:11:25 2022
  Temporary Projects                  D        0  Fri Nov 11 22:19:03 2022

		6232831 blocks of size 4096. 1013631 blocks available
smb: \>
```

This share looks more interesting

```python
smb: \Temporary Projects\> dir
  .                                   D        0  Fri Nov 11 22:19:03 2022
  ..                                  D        0  Fri Nov 11 22:19:03 2022
  Encrypter.exe                       A     5632  Fri Nov  4 16:51:59 2022
  s.blade.enc                         A     3808  Fri Nov 11 22:17:08 2022

		6232831 blocks of size 4096. 1015003 blocks available
smb: \Temporary Projects\> 
```

Ill switch to the impacket python tool since it has better download functionality

```python
❯❯❯ smbclient.py coder.htb/Guest:''@dc01.coder.htb
# shares
# use Development
# rget *
```

This downloads all the file here

The encrypter is .NET which means i can load it into ILspy

# Reverse engineering of .NET binary

```C#
// AES
using System;
using System.IO;
using System.Security.Cryptography;

internal class AES
{
	public static void Main(string[] args)
	{
		if (args.Length != 1)
		{
			Console.WriteLine("You must provide the name of a file to encrypt.");
			return;
		}
		FileInfo fileInfo = new FileInfo(args[0]);
		string destFile = Path.ChangeExtension(fileInfo.Name, ".enc");
		long value = DateTimeOffset.Now.ToUnixTimeSeconds();
		Random random = new Random(Convert.ToInt32(value));
		byte[] array = new byte[16];
		random.NextBytes(array);
		byte[] array2 = new byte[32];
		random.NextBytes(array2);
		byte[] array3 = EncryptFile(fileInfo.Name, destFile, array2, array);
	}

	private static byte[] EncryptFile(string sourceFile, string destFile, byte[] Key, byte[] IV)
	{
		using (RijndaelManaged rijndaelManaged = new RijndaelManaged())
		{
			using FileStream stream = new FileStream(destFile, FileMode.Create);
			using ICryptoTransform transform = rijndaelManaged.CreateEncryptor(Key, IV);
			using CryptoStream cryptoStream = new CryptoStream(stream, transform, CryptoStreamMode.Write);
			using FileStream fileStream = new FileStream(sourceFile, FileMode.Open);
			byte[] array = new byte[1024];
			int count;
			while ((count = fileStream.Read(array, 0, array.Length)) != 0)
			{
				cryptoStream.Write(array, 0, count);
			}
		}
		return null;
	}
}
```

This is pretty much all of the application

This is the code its using to encrypt the file

```python
long value = DateTimeOffset.Now.ToUnixTimeSeconds();
Random random = new Random(Convert.ToInt32(value));   
```

This bit of code is the most interesting

> System.Random is a linear congruential generator (LCG), not a CSPRNG. Seeding it with the current Unix timestamp means the 16-byte IV and 32-byte key are fully determined by the second the file was encrypted. If you can recover that timestamp (e.g., from file metadata, logs, or by brute-forcing a small window of nearby seconds), you can reproduce the exact key and IV and decrypt the .enc file.

This essentially means if i can find the exact timestamp it was encrypted i can use that timestamp to decrypt

```python
# ls
drw-rw-rw-          0  Fri Nov 11 22:19:03 2022 .
drw-rw-rw-          0  Fri Nov 11 22:19:03 2022 ..
-rw-rw-rw-       5632  Fri Nov  4 16:58:20 2022 Encrypter.exe
-rw-rw-rw-       3808  Fri Nov 11 22:17:08 2022 s.blade.enc
# 
```

Using the SMB client i can see the date used 

This is synced with my clock though, so ill have to convert it

```python
❯❯❯ date -d "2022-11-11 17:17:08 UTC" +"%s"
1668187028
```

Then ill create the seed


