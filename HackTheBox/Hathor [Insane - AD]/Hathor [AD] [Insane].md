# Host file setup

```python
sudo nxc smb 10.129.230.109 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.230.109  445    hathor           [*]  x64 (name:hathor) (domain:windcorp.htb) (signing:True) (SMBv1:None) (NTLM:False)
```
- NTLM auth is disabled!
- SMB signing is enabled

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT hathor.windcorp.htb  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-17 17:46 +0100
Nmap scan report for hathor.windcorp.htb (10.129.230.109)
Host is up (0.012s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49667/tcp open  unknown
54175/tcp open  unknown
61498/tcp open  unknown
61517/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.77 seconds
```

## Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=2000 -sT hathor.windcorp.htb  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-17 17:49 +0100
Nmap scan report for hathor.windcorp.htb (10.129.230.109)
Host is up (0.013s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: Home - mojoPortal
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
| http-robots.txt: 29 disallowed entries (15 shown)
| /CaptchaImage.ashx* /Admin/ /App_Browsers/ /App_Code/ 
| /App_Data/ /App_Themes/ /bin/ /Blog/ViewCategory.aspx$ 
| /Blog/ViewArchive.aspx$ /Data/SiteImages/emoticons /MyPage.aspx 
|_/MyPage.aspx$ /MyPage.aspx* /NeatHtml/ /NeatUpload/
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-05-17 16:49:10Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: windcorp.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=hathor.windcorp.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:hathor.windcorp.htb
| Not valid before: 2026-05-17T16:35:46
|_Not valid after:  2027-05-17T16:35:46
|_ssl-date: 2026-05-17T16:50:34+00:00; +4s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: windcorp.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-05-17T16:50:34+00:00; +4s from scanner time.
| ssl-cert: Subject: commonName=hathor.windcorp.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:hathor.windcorp.htb
| Not valid before: 2026-05-17T16:35:46
|_Not valid after:  2027-05-17T16:35:46
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: windcorp.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-05-17T16:50:34+00:00; +4s from scanner time.
| ssl-cert: Subject: commonName=hathor.windcorp.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:hathor.windcorp.htb
| Not valid before: 2026-05-17T16:35:46
|_Not valid after:  2027-05-17T16:35:46
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: windcorp.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=hathor.windcorp.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:hathor.windcorp.htb
| Not valid before: 2026-05-17T16:35:46
|_Not valid after:  2027-05-17T16:35:46
|_ssl-date: 2026-05-17T16:50:34+00:00; +4s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: HATHOR; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)

Since NTLM auth is disabled, i cannot use null authenticaton

The guest account also appears to be disabled!

# HTTP (80)

## Nuclei

```python
nuclei -u http://windcorp.htb/                                                                             

[cookies-without-secure] [javascript] [info] windcorp.htb ["ASP","NET_SessionId"]
[waf-detect:aspgeneric] [http] [info] http://windcorp.htb/
[iis-shortname-detect] [http] [info] http://windcorp.htb/*~1*/a.aspx
[wordpress-xmlrpc-listmethods] [http] [info] http://windcorp.htb/xmlrpc.php
[ldap-metadata] [javascript] [info] windcorp.htb:389 ["DomainFunctionality: 7","ForestFunctionality: 7","DomainControllerFunctionality: 7","BaseDN: dc=389","DnsHostName: hathor.windcorp.htb","DefaultNamingContext: DC=windcorp,DC=htb"]
[ldap-anonymous-login-detect] [javascript] [medium] windcorp.htb
[form-detection] [http] [info] http://windcorp.htb/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://windcorp.htb/
[http-missing-security-headers:missing-content-type] [http] [info] http://windcorp.htb/
[http-missing-security-headers:strict-transport-security] [http] [info] http://windcorp.htb/
[http-missing-security-headers:x-frame-options] [http] [info] http://windcorp.htb/
[http-missing-security-headers:referrer-policy] [http] [info] http://windcorp.htb/
[http-missing-security-headers:clear-site-data] [http] [info] http://windcorp.htb/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://windcorp.htb/
[http-missing-security-headers:content-security-policy] [http] [info] http://windcorp.htb/
[http-missing-security-headers:permissions-policy] [http] [info] http://windcorp.htb/
[http-missing-security-headers:x-content-type-options] [http] [info] http://windcorp.htb/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://windcorp.htb/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://windcorp.htb/
[robots-txt-endpoint:endpoints] [http] [info] http://windcorp.htb/robots.txt ["/Data/SiteImages/emoticons","/MyPage.aspx","/NeatHtml/","/Secure/","/App_Browsers/","/NeatUpload/","/App_Themes/","/bin/","/Blog/ViewArchive.aspx","/nf/","/Services/TinyMCETemplates.ashx","/SiteMap.aspx","/Setup/","/WebStore/CartAdd.aspx","/CaptchaImage.ashx","/Admin/","/Blog/ViewCategory.aspx","/nofollow/","/SearchResults.aspx","/SiteOffice/","/WebStore/Cart.aspx","/Error.htm","/App_Code/","/App_Data/"]
[microsoft-iis-version] [http] [info] http://windcorp.htb/ ["Microsoft-IIS/10.0"]
[missing-sri] [http] [info] http://windcorp.htb/ ["//ajax.aspnetcdn.com/ajax/4.5/6/WebFormsBundle.js","//ajax.aspnetcdn.com/ajax/4.5/6/MsAjaxBundle.js","//maxcdn.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css","//code.jquery.com/ui/1.12.1/themes/smoothness/jquery-ui.css"]
[CVE-2025-28367] [http] [medium] http://windcorp.htb/api/BetterImageGallery/imagehandler?path=../../../Web.Config
[options-method] [http] [info] http://windcorp.htb/ ["GET, HEAD, OPTIONS, TRACE"]
[CVE-2023-24322] [http] [medium] http://windcorp.htb/Dialog/FileDialog.aspx?ed=foooooooooooooo%27);});});javascript:alert('document.domain');//g
[aspnet-version-detect] [http] [info] http://windcorp.htb/ ["4.0.30319"]
[missing-cookie-samesite-strict] [http] [info] http://windcorp.htb/ ["ASP.NET_SessionId=3bfxoxj0lc4cr30xyzmkpzq4; path=/; HttpOnly; SameSite=Lax"]
[mojoportal-detect] [http] [info] http://windcorp.htb/
[tech-detect:microsoft-asp.net] [http] [info] http://windcorp.htb/
[tech-detect:font-awesome] [http] [info] http://windcorp.htb/
[tech-detect:ms-iis] [http] [info] http://windcorp.htb/
[host-header-injection] [http] [info] http://windcorp.htb/
[robots-txt] [http] [info] http://windcorp.htb/robots.txt
[caa-fingerprint] [dns] [info] windcorp.htb
```
- CVE-2025-28367?
- CVE-2023-24322?

# Access to the web portal

The CMS that is running is MojoPortal, i can do some research into the default credentials to get access!

![](Pasted%20image%2020260517184754.png)

![](Pasted%20image%2020260517184808.png)

I now have admin access to the web portal!

![](Pasted%20image%2020260517185234.png)

There is also a file upload functionality but it appears to block php uploads, ill do some testing to figure out what extensions it does allow

I can also try renaming the file in the file manager, but this also fails

Ill try other extenstions too, like .aspx

# Remote code execution on the web server

https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/asp/cmd.aspx

Ill use this aspx file and save it my machine as a txt file then upload it as a txt file, once uploaded i can try renaming the file to make it a aspx file

So renaming after file upload failed, but i can try copying the file to another location with a different file name

![](Pasted%20image%2020260517194733.png)

Ill plant it as a txt file then proxy the request and see what i can do with it

![](Pasted%20image%2020260517195002.png)

After proxying the request and sending it, it doesnt give an error, so this might have worked

I know its in the same path as the image on the homepage so going back to the homeoage and viewing the image will give me the path i need and all i need to do is change the filename

![](Pasted%20image%2020260517195215.png)

I now have a web shell!

![](Pasted%20image%2020260517195307.png)

It is running as the web user!

![](Pasted%20image%2020260517195443.png)

Ive tried to upload a revshell from revshells.com but i get an error, which tells me the firewall is blocking this!

![](Pasted%20image%2020260517195752.png)

Found an interesting directory in the root, also `share`

> **Get-bADpasswords** is a PowerShell script designed to audit Active Directory environments for weak, leaked, or shared user passwords. It allows administrators to compare active users' passwords against custom wordlists (like breached credential databases) before attackers can exploit them.


![](Pasted%20image%2020260517195956.png)

```pythonVolume in drive C has no label.
 Volume Serial Number is BE61-D5E0

 Directory of C:\Get-bADpasswords\Accessible\CSVs

10/13/2022  09:09 PM    <DIR>          .
09/29/2021  08:18 PM    <DIR>          ..
10/03/2021  05:35 PM               248 exported_windcorp-03102021-173510.csv
10/03/2021  06:07 PM               248 exported_windcorp-03102021-180635.csv
10/03/2021  06:21 PM               112 exported_windcorp-03102021-182114.csv
10/03/2021  06:22 PM               112 exported_windcorp-03102021-182259.csv
10/03/2021  06:28 PM               248 exported_windcorp-03102021-182627.csv
10/03/2021  06:52 PM               248 exported_windcorp-03102021-185058.csv
10/04/2021  11:37 AM               248 exported_windcorp-04102021-113140.csv
10/05/2021  06:40 PM               248 exported_windcorp-05102021-183949.csv
10/13/2022  09:13 PM               248 exported_windcorp-13102022-210856.csv
10/13/2022  09:13 PM               248 exported_windcorp-13102022-210946.csv
03/17/2022  05:40 AM               112 exported_windcorp-17032022-044053.csv
03/18/2022  05:40 AM               112 exported_windcorp-18032022-044046.csv
              12 File(s)          2,432 bytes
               2 Dir(s)   9,346,019,328 bytes free
```
Found some csv files, ill have a look whats inside them.

Checking the most recent entry:

![](Pasted%20image%2020260517201716.png)

`exported_windcorp-13102022-210946.csv` Held what looks like a password hash!

```python

```