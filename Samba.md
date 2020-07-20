# Samba
#pentest/tryhackme

## What
Samba is the standard Windows interoperability suite of programs for Linux and Unix. It allows end users to access and use files, printers and other commonly shared resources on a companies intranet or internet. Its often refereed to as a network file system.

Samba is based on the common client/server protocol of Server Message Block (SMB). SMB is developed only for Windows, without Samba, other computer platforms would be isolated from Windows machines, even if they were part of the same network.

## Enumerating shares
Using **nmap** we can enumerate a machine for SMB shares. Nmap has the ability to run to automate a wide variety of networking tasks. There is a script to enumerate shares!

`nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse <machine-ip>`

## Inspecting a share
On most distributions of Linux smbclient is already installed. Lets inspect one of the shares.
`smbclient //<ip>/anonymous`

## Downloading smbshares
You can recursively download the SMB share too. Submit the username and password as nothing.
`smbget -R smb://<ip>/anonymous`


- - - -

## Better tool
A more powerful tool would be [GitHub - ShawnDEvans/smbmap: SMBMap is a handy SMB enumeration tool](https://github.com/ShawnDEvans/smbmap), with functionalities like:

* Pass-the-Hash Support
* File upload/download/delete
* Permission enumeration (writable share, meet Metasploit)
* Remote Command Execution
* Distrubted file content searching (beta!)
* File name matching (with an auto downoad capability)


