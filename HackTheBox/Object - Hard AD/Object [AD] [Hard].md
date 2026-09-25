# Enumeration
## Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.129.96.147 -vv

Nmap scan report for 10.129.96.147
Host is up, received user-set (0.014s latency).
Scanned at 2026-09-25 16:32:08 BST for 66s
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE    REASON
80/tcp   open  http       syn-ack
5985/tcp open  wsman      syn-ack
8080/tcp open  http-proxy syn-ack
```

## Nmap
```python
nmap -p 80,5985,8080 -A --min-rate=2000 -sT -Pn 10.129.96.147
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-25 16:33 +0100
Nmap scan report for 10.129.96.147
Host is up (0.014s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Mega Engines
| http-methods: 
|_  Potentially risky methods: TRACE
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
8080/tcp open  http    Jetty 9.4.43.v20210629
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
|_http-server-header: Jetty(9.4.43.v20210629)
| http-robots.txt: 1 disallowed entry 
|_/
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1903 - 22H2 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

# HTTP (80)

![](Pasted%20image%2020260925163525.png)

 IIS

Also a link to a automation server?

Also a potential user `ideas`

# HTTP (8080)

![](Pasted%20image%2020260925163701.png)

It goes to `object.htb`

![](Pasted%20image%2020260925163820.png)

Jenkins install

It also looks to accept user registration

![](Pasted%20image%2020260925164500.png)

Ill register an account, and now i can find the version

# Executing malicious project

https://blog.cyberadvisors.com/technical-blog/blog/jenkins-remote-execution-via-malicious-jobs

![](Pasted%20image%2020260925171242.png)

Ill set the time to execute every minute

![](Pasted%20image%2020260925171301.png)

And ill set a simple test command first

Ill then save the job and after i minute i get the job show up in the build history

![](Pasted%20image%2020260925171424.png)

I can then view the console output

![](Pasted%20image%2020260925171452.png)

I have RCE

# Reverse shell (Fail)

However after trying several different things, i cannot get a connection back to myself

![](Pasted%20image%2020260925171853.png)

Ill enumerate the firewall rules

![](Pasted%20image%2020260925172020.png)

And after some further enumeration, it doesnt look like im going to be able to get a reverse shell due to the rules, but i can look for some credentials

# Credential hunting

![](Pasted%20image%2020260925172935.png)

Ill start looking around the config for any credentials

![](Pasted%20image%2020260925173116.png)

Ill look into admin

![](Pasted%20image%2020260925173215.png)

There is a config.xml file

```python
C:\Users\oliver\AppData\Local\Jenkins\.jenkins\workspace\test>powershell -c type ../../users/admin_17207690984073220035/config.xml 
<?xml version='1.1' encoding='UTF-8'?>
<user>
  <version>10</version>
  <id>admin</id>
  <fullName>admin</fullName>
  <properties>
    <com.cloudbees.plugins.credentials.UserCredentialsProvider_-UserCredentialsProperty plugin="credentials@2.6.1">
      <domainCredentialsMap class="hudson.util.CopyOnWriteMap$Hash">
        <entry>
          <com.cloudbees.plugins.credentials.domains.Domain>
            <specifications/>
          </com.cloudbees.plugins.credentials.domains.Domain>
          <java.util.concurrent.CopyOnWriteArrayList>
            <com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl>
              <id>320a60b9-1e5c-4399-8afe-44466c9cde9e</id>
              <description></description>
              <username>oliver</username>
              <password>{AQAAABAAAAAQqU+m+mC6ZnLa0+yaanj2eBSbTk+h4P5omjKdwV17vcA=}</password>
              <usernameSecret>false</usernameSecret>
            </com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl>
          </java.util.concurrent.CopyOnWriteArrayList>
        </entry>
      </domainCredentialsMap>
    </com.cloudbees.plugins.credentials.UserCredentialsProvider_-UserCredentialsProperty>
    <hudson.plugins.emailext.watching.EmailExtWatchAction_-UserProperty plugin="email-ext@2.84">
      <triggers/>
    </hudson.plugins.emailext.watching.EmailExtWatchAction_-UserProperty>
    <hudson.model.MyViewsProperty>
      <views>
        <hudson.model.AllView>
          <owner class="hudson.model.MyViewsProperty" reference="../../.."/>
          <name>all</name>
          <filterExecutors>false</filterExecutors>
          <filterQueue>false</filterQueue>
          <properties class="hudson.model.View$PropertyList"/>
        </hudson.model.AllView>
      </views>
    </hudson.model.MyViewsProperty>
    <org.jenkinsci.plugins.displayurlapi.user.PreferredProviderUserProperty plugin="display-url-api@2.3.5">
      <providerId>default</providerId>
    </org.jenkinsci.plugins.displayurlapi.user.PreferredProviderUserProperty>
    <hudson.model.PaneStatusProperties>
      <collapsed/>
    </hudson.model.PaneStatusProperties>
    <jenkins.security.seed.UserSeedProperty>
      <seed>ea75b5bd80e4763e</seed>
    </jenkins.security.seed.UserSeedProperty>
    <hudson.search.UserSearchProperty>
      <insensitiveSearch>true</insensitiveSearch>
    </hudson.search.UserSearchProperty>
    <hudson.model.TimeZoneProperty/>
    <hudson.security.HudsonPrivateSecurityRealm_-Details>
      <passwordHash>#jbcrypt:$2a$10$q17aCNxgciQt8S246U4ZauOccOY7wlkDih9b/0j4IVjZsdjUNAPoW</passwordHash>
    </hudson.security.HudsonPrivateSecurityRealm_-Details>
    <hudson.tasks.Mailer_-UserProperty plugin="mailer@1.34">
      <emailAddress>admin@object.local</emailAddress>
    </hudson.tasks.Mailer_-UserProperty>
    <jenkins.security.ApiTokenProperty>
      <tokenStore>
        <tokenList/>
      </tokenStore>
    </jenkins.security.ApiTokenProperty>
    <jenkins.security.LastGrantedAuthoritiesProperty>
      <roles>
        <string>authenticated</string>
      </roles>
      <timestamp>1634793332195</timestamp>
    </jenkins.security.LastGrantedAuthoritiesProperty>
  </properties>
</user>
```

There is a password hash and what looks like an encrypted password

The hash type is very hard to crack, so ill likely try to decrpyt the password

