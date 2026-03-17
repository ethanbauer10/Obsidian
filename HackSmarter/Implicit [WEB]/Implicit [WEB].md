# Objectives
## 1
Our engineering team is currently rolling out "HackSmarter ID," a centralized Single Sign-On (SSO) solution designed to provide seamless access across all of our internal platforms. To support lightweight, browser-based applications, the developers have implemented an OAuth-style Implicit Grant flow.

You have been contracted to perform a targeted penetration test against this new integration before it goes live to production. Can you log in as the `administrator` user?
## 2
As the attending Application Security Engineer, you must immediately triage the vulnerable codebase, implement a hot-patch, and deploy the fix via our automated CI/CD pipeline.

You can access the internal Git server at `http://[lab-ip]:3000` and log in with these credentials: `student:HackSmarter2026!`

All changes you make to `app.py` will automatically be pushed to production (it can take 2 - 4 minutes for the action to run). Your task is to fix the vuln you discovered (without breaking the app).

Verify the fix by attempting to re-exploit the app.

# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT 10.1.182.248                                   
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-16 17:59 +0000
Nmap scan report for 10.1.182.248
Host is up (0.090s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
222/tcp  open  rsh-spx
3000/tcp open  ppp

Nmap done: 1 IP address (1 host up) scanned in 34.02 seconds
```
## Nmap
```python
nmap -p 22,80,222,3000 -A --min-rate=2000 -sT 10.1.182.248
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-16 18:02 +0000
Nmap scan report for 10.1.182.248
Host is up (0.090s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 de:92:0b:86:7c:1b:7c:9d:02:12:14:91:b9:5d:e9:9a (ECDSA)
|_  256 ce:a9:25:fe:f5:66:1b:d9:e1:97:12:31:3e:cd:03:5c (ED25519)

80/tcp   open  http    Werkzeug httpd 3.0.1 (Python 3.10.19)
|_http-title: Login | Hack Smarter
|_http-server-header: Werkzeug/3.0.1 Python/3.10.19

222/tcp  open  ssh     OpenSSH 9.3 (protocol 2.0)
| ssh-hostkey: 
|   256 52:fe:84:67:fc:67:7a:48:ff:bb:91:f6:74:7e:60:99 (ECDSA)
|_  256 25:75:2c:8b:90:ef:47:87:be:df:3a:4f:ff:af:b1:a8 (ED25519)

3000/tcp open  http    Golang net/http server
|_http-title: Gitea: Git with a cup of tea
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Content-Type: text/html; charset=utf-8
|     Set-Cookie: i_like_gitea=3f61e61c75a0b055; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=OuWxTu72RrAIQLPKyn4ZW8QMRTc6MTc3MzY4NDE4MTc3OTcyNDIyNQ; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Mon, 16 Mar 2026 18:03:01 GMT
|     <!DOCTYPE html>
|     <html lang="en-US" class="theme-auto">
|     <head>
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <title>Gitea: Git with a cup of tea</title>
|     <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiR2l0ZWE6IEdpdCB3aXRoIGEgY3VwIG9mIHRlYSIsInNob3J0X25hbWUiOiJHaXRlYTogR2l0IHdpdGggYSBjdXAgb2YgdGVhIiwic3RhcnRfdXJsIjoiaHR0cDovL2xvY2FsaG9zdDozMDAwLyIsImljb25zIjpbeyJzcmMiOiJodHRwOi8vbG9jYWxob3N0OjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9wbmciLCJzaXplcyI6IjU
|   HTTPOptions: 
|     HTTP/1.0 405 Method Not Allowed
|     Allow: HEAD
|     Allow: GET
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Set-Cookie: i_like_gitea=baece25e27d1fe34; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=1ntskfWF7dgihLpEBAjeiz9pXo86MTc3MzY4NDE4MjE3OTMxNjI0MQ; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Mon, 16 Mar 2026 18:03:02 GMT
|_    Content-Length: 0

Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# SSH (22)
## Version
```python
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 de:92:0b:86:7c:1b:7c:9d:02:12:14:91:b9:5d:e9:9a (ECDSA)
|_  256 ce:a9:25:fe:f5:66:1b:d9:e1:97:12:31:3e:cd:03:5c (ED25519)
```
## Auth method
```python
ssh root@implicit.hsm          
The authenticity of host 'implicit.hsm (10.1.182.248)' can't be established.
ED25519 key fingerprint is: SHA256:pMcomFBAGwpjq0ySIc3GEWM+Yw5ozbd3hWvaqWyAA/A
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'implicit.hsm' (ED25519) to the list of known hosts.
root@implicit.hsm: Permission denied (publickey).
```
This has key-based auth
# SSH (222)
## Version
```python
222/tcp  open  ssh     OpenSSH 9.3 (protocol 2.0)
| ssh-hostkey: 
|   256 52:fe:84:67:fc:67:7a:48:ff:bb:91:f6:74:7e:60:99 (ECDSA)
|_  256 25:75:2c:8b:90:ef:47:87:be:df:3a:4f:ff:af:b1:a8 (ED25519)
```
## Auth method
```python
ssh root@implicit.hsm -p 222
The authenticity of host '[implicit.hsm]:222 ([10.1.182.248]:222)' can't be established.
ED25519 key fingerprint is: SHA256:5zAXPAdL/XSCt25rCN1t32Wj+CpHVAJBtf9HZamP6zY
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[implicit.hsm]:222' (ED25519) to the list of known hosts.
root@implicit.hsm: Permission denied (publickey).
```
This is also key based auth

# HTTP (80)
## Nuclei
```python
nuclei -u http://implicit.hsm/                            

[INF] Skipped implicit.hsm:9780 from target list as found unresponsive permanently: cause="port closed or filtered" address=implicit.hsm:9780 chain="connection refused; got err while executing http://implicit.hsm:9780/api/v1/user_assets/nfc"
[ssh-server-enumeration] [javascript] [info] implicit.hsm:22 ["SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.14"]
[ssh-auth-methods] [javascript] [info] implicit.hsm:22 ["["publickey"]"]
[ssh-sha1-hmac-algo] [javascript] [info] implicit.hsm:22
[openssh-detect] [tcp] [info] implicit.hsm:22 ["SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.14"]
[tech-detect:python] [http] [info] http://implicit.hsm/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://implicit.hsm/
[http-missing-security-headers:clear-site-data] [http] [info] http://implicit.hsm/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://implicit.hsm/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://implicit.hsm/
[http-missing-security-headers:permissions-policy] [http] [info] http://implicit.hsm/
[http-missing-security-headers:referrer-policy] [http] [info] http://implicit.hsm/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://implicit.hsm/
[http-missing-security-headers:missing-content-type] [http] [info] http://implicit.hsm/
[http-missing-security-headers:strict-transport-security] [http] [info] http://implicit.hsm/
[http-missing-security-headers:content-security-policy] [http] [info] http://implicit.hsm/
[http-missing-security-headers:x-frame-options] [http] [info] http://implicit.hsm/
[http-missing-security-headers:x-content-type-options] [http] [info] http://implicit.hsm/
[options-method] [http] [info] http://implicit.hsm/ ["OPTIONS, GET, HEAD"]
[INF] Skipped implicit.hsm:5814 from target list as found unresponsive permanently: Get "https://implicit.hsm:5814/autopass": cause="port closed or filtered" address=implicit.hsm:5814 chain="connection refused"
[INF] Skipped implicit.hsm:4040 from target list as found unresponsive permanently: cause="port closed or filtered" address=implicit.hsm:4040 chain="connection refused; got err while executing http://implicit.hsm:4040/jobs/"
[werkzeug-debugger-detect] [http] [info] http://implicit.hsm/console
[caa-fingerprint] [dns] [info] implicit.hsm
```
## Subdomains
```python
nothing found
```
## Foroxbuster
```python
feroxbuster -u http://implicit.hsm/ -C 404

302      GET        5l       22w      189c http://implicit.hsm/logout => http://implicit.hsm/
200      GET      199l      417w     4185c http://implicit.hsm/static/css/style.css
302      GET        5l       22w      207c http://implicit.hsm/oauth/auth => http://implicit.hsm/sso/login
200      GET       65l      225w     2339c http://implicit.hsm/static/js/auth.js
200      GET       25l       75w      878c http://implicit.hsm/
302      GET        5l       22w      189c http://implicit.hsm/dashboard => http://implicit.hsm/
200      GET       45l      144w     1563c http://implicit.hsm/console
```
## Website functionality
![[Pasted image 20260316190148.png|794]]
So this button redirects me to a login form 
Whats interesting is when i test something like `admin:admin` i get a breif snapshot of something
Not vulnerable SQL injection

There is also a register function
The register form has manual username enumeration
```python
POST /sso/register HTTP/1.1
Host: implicit.hsm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 31
Origin: http://implicit.hsm
Connection: keep-alive
Referer: http://implicit.hsm/sso/register
Upgrade-Insecure-Requests: 1
Priority: u=0, i

username=hacker&password=hacker
```
Ive sent this request and in the response:
```python
<div class="error-message">Username already exists. Please choose another.</div>
```
There did not appear to be an `admin` user so i registered one
![[Pasted image 20260316190754.png|696]]
This is the page i am greeted with after logging in
So after intercepting this request in caido after pressing authorize i forward several requests till i see a post request going to `/api/login`

```python
POST /api/login HTTP/1.1
Host: implicit.hsm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://implicit.hsm/
Content-Type: application/json
Content-Length: 74
Origin: http://implicit.hsm
Connection: keep-alive
Cookie: session=eyJzc29fdXNlciI6ImRhcmt0cmFjZSJ9.abhXeQ.AyUDN0Je_JKcj6j_B6LPhBaYfsM
Priority: u=4

{"access_token":"94209c9428f640cbbe9506b56ce511be","username":"admin"}
```

# Access as the administrator
![[Pasted image 20260316192758.png|886]]
So using the request above going to `/api/login` i simply changed the user to administrator and forwarded the request then after a refresh i got admin access

# Source code review
After logging into the gitea instance on port 3000 i found the vulnerable code:
```python
@app.route('/api/login', methods=['POST'])
def api_login():
    client_token = data.get('access_token')
    client_username = data.get('username')
    
    if client_token in valid_tokens:
        session['username'] = client_username  #THE BUG
        return jsonify({"success": True})
```
## What's wrong?
The server validates that the token exists, but never checks who the token actually belongs to. valid_tokens maps token -> username, but that mapping is completely ignored. The server just trusts whatever username the client sends in the JSON body.

## The fix
I will remove:
```python
@app.route('/api/login', methods=['POST'])
def api_login():
    client_token = data.get('access_token')
    client_username = data.get('username')
    
    if client_token in valid_tokens:
        session['username'] = client_username  # ← THE BUG
        return jsonify({"success": True})
```

And replace it with:
```python
@app.route('/api/login', methods=['POST'])
def api_login():
    data = request.get_json()
    if not data:
        return jsonify({"success": False, "message": "Invalid request."}), 400

    client_token = data.get('access_token')

    # Resolve identity from the token — never trust client-supplied username
    token_owner = valid_tokens.get(client_token)

    if token_owner is None:
        return jsonify({"success": False, "message": "Invalid or expired access token."}), 401

    # Invalidate the token after use (one-time use)
    del valid_tokens[client_token]

    session['username'] = token_owner  # ← identity comes from the server, not the client
    return jsonify({"success": True})
```
This code now eradicates the vulnerability