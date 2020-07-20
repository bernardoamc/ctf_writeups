# Contents
#pentest/tryhackme


## VirtualBox 
```bash
kali/kali

# Mounting shared folder
sudo mount -t vboxsf kali ~/share

# Run OpenVPN
sudo openvpn ~/share/k4eros.ovpn
```

## File locations
* [[Kali Locations]]

## WEB Vulnerabilities
* [[CSRF]]
* [[XSS]]
* [[XML External Entity (XXE)]]
* [[SSTI]]
* [[XML External Entity (XXE)]]
* [[Local File Inclusion In-depth]]
* [[JWT]]
	* [[JWT Walkthrough with HS256 algorithm]]
	* [[JWT Walkthrough with None Algorithm]]

## Linux Privilege Escalation
* [[Linux Privilege Escalation]]
* [[Linux wildcard exploitation]]

## Windows Privilege Escalation
* [[winPEAS - Windows Privilege Escalation Recon]]
* [[PowerShell, Metasploit and Privilege Escalation through Misconfiguration]]

## Samba
* [[Samba]]

## Resources
* [HackTricks - HackTricks](https://book.hacktricks.xyz/)
* [Upgrading Simple Shells to Fully Interactive TTYs - ropnop blog](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/)

## Tools
* [[Tools]]

## Walkthroughs Windows
* [[Corp Writeup]]
	* [[Kerberos]]
	* PowerUp1 for privilege escalation
	* UnnatendPath misconfiguration
*  [[Hackpark writeup]]
	* BlogEngine vuln
	* hydra
	* metasploit
	* 	msfvenom
	* winPEAS
* [[Alfred Writeup]]
	* Jenkins vuln
	* metasploit
	* privilege escalation through permissions 
* [[Steel Mountain Writeup]] 
	* HttpFileServer vuln
	* powerUP script for enumeration
*  [[Blue Machine Walkthrough]] 
	* Eternal Blue
* [[Empire]]

## Walkthrough Web
* [[Ignite Machine Walkthrough]]
* [[ToolsRUs Room Writeup]]
* [[Pickle Rick Room Writeup]]
* [[game Zone writeup]]
	* sqlmap

## Linux Writeup
* [[Linux Privesc Playground Writup]]
* [[LFI Room]] 
* [[Skynet writeup]]
	* dirsearch
	* nmap
	* samba
	* LFI
	* Wildcard exploitation with `tar`, see [[Linux wildcard exploitation]]
	* crontabs
	* squirrelmail
	* CuppaCMS
* [[Daily Bugle Writeup]]
	* Joomla
	* GTFObins (yum)
	* dirsearch
* [[Agent Sudo Writeup]]
	* Steganography with `steghide` and `foremost`
	* Hydra to crack `ftp`
	* John with zip2john
	* Burpsuite intruder
	* [sudo 1.8.27 - Security Bypass - Linux local Exploit](https://www.exploit-db.com/exploits/47502)