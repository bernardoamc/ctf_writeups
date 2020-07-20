# Tools
#pentest/tryhackme

## Service + Vulnerability Scanners
* [[Nmap]]

## Web Vulnerabilities Scanners
* [[Nikto]]
* [[ZAP]]

## Full fledged scanners (web, system and so on)
* [[Nessus]]

## Steganography
* [[Steganography]]

## Exploit Platform
* [[Metasploit Intro]] and [[Metasploit Modules]] and [[Metasploit Exploit Example]]

## Windows specific
* [GitHub - samratashok/nishang: Nishang - Offensive PowerShell for red team, penetration testing and offensive security.](https://github.com/samratashok/nishang)

## Linux specific
* [[Linux wildcard exploitation]]
* [[Linux Privilege Escalation]]

## Post Exploitation Platform
* [[Empire]]

##  Directory search
* [GitHub - maurosoria/dirsearch: Web path scanner](https://github.com/maurosoria/dirsearch)
	* `python3 dirsearch.py -u 10.10.75.35 --extensions-list`
* [GitHub - OJ/gobuster: Directory/File, DNS and VHost busting tool written in Go](https://github.com/OJ/gobuster)
	* `gobuster dir -u https://buffered.io -w /usr/share/shortlist.txt`

## Extract endpoints from javascript files
* [GitHub - dark-warlord14/LinkFinder: A python script that finds endpoints in JavaScript files](https://github.com/dark-warlord14/LinkFinder)
	* Check the extension script on [[Bash Scrips]]
	* We can also couple this with js-beautify to get more information
		* `npm -g install js-beautify`

## Extract bits of URLs after recon
* [GitHub - tomnomnom/unfurl: Pull out bits of URLs provided on stdin](https://github.com/tomnomnom/unfurl)

## URL fetcher
[GitHub - lc/gau: Fetch known URLs from AlienVault's Open Threat Exchange, the Wayback Machine, and Common Crawl.](https://github.com/lc/gau)

## Web fuzzers
[GitHub - ffuf/ffuf: Fast web fuzzer written in Go](https://github.com/ffuf/ffuf)

## URL extractors
[GitHub - tomnomnom/unfurl: Pull out bits of URLs provided on stdin](https://github.com/tomnomnom/unfurl)

## HTTP Request Smuggling / Dsync Attacks
* [GitHub - defparam/smuggler: Smuggler - An HTTP Request Smuggling / Desync testing tool written in Python 3](https://github.com/defparam/smuggler)
	* [GitHub - defparam/tiscripts: Turbo Intruder Scripts](https://github.com/defparam/tiscripts)

## Login/Password bruteforcing
* [[Hydra]]

## Password cracking
* [[Hashcat]]
* John

## SQL Injection
* [[sqlmap]]

## Joomla
* [[Joomla Recon and Vulnerability checking]]