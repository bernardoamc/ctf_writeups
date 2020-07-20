# sqlmap
#pentest/tryhackme

## What
Tool to exploit sql injection vulnerabilities and dump databases

## How
1. Save a request to a field vulnerable to sqli
2. Run `sqlmap` with that request file

```shell
$ sqlmap -r request.txt -dbms mysql --dump
```


## Using burpsuite
1. Intercept a request and send to repeater
2. Copy the request to a file (right click -> copy to file)
3. Check for vulnerabilities using `sqlmap`

```shell
sqlmap -r <filename.req> --batch 
```