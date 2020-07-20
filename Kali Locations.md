# Kali Locations
#pentest/tryhackme

## Webshells location
`/usr/share/webshells/php/`

## Windows tools
`/usr/share/windows-resources/`

## Username/password Wordlists 
`/usr/share/wordlists`

## Directory wordlists (dirb stands for dirbuster)
`/usr/share/dirb/wordlists/`

## Listen for inbound connections:
`nc -l -p port`

## Firefox
`/usr/test/firefox/firefox`

## Change desktop environments
```
$ sudo apt-get update
$ dpkg --configure -a. (abort any pending package configs)
$ sudo tasksel
$ sudo update-alternatives --config x-session-manager.  (example:  /usr/bin/startxfce4
$ sudo reboot
```