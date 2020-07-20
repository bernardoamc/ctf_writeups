# Metasploit Intro
#pentest #pentest/tryhackme/metasploit

## Initialize the database
`msfdb`

## Advanced options we can trigger for starting the console
`msfconsole -h`

## Are we connected to the database?
`db_status`

## Help within msfconsole
`help` or `?`

## Finding various modules we have at our disposal within Metasploit is one of the most common commands we will leverage in the framework. What is the base command we use for searching?
`search`

## Once we’ve found the module we want to leverage, what command we use to select it as the active module?
`use <path or number>`

## How about if we want to view information about either a specific module or just the active one we have selected?
`info`

##  Metasploit has a built-in netcat-like function where we can make a quick connection with a host simply to verify that we can ‘talk’ to it. What command is this?
`connect`

## How to show which variables we can change within a module?
`show options`

## What command do we use to change the value of a variable?
`set <variable> <value>`

## Metasploit supports the use of global variables, something which is incredibly useful when you’re specifically focusing on a single box. What command changes the value of a variable globally?
`setg <variable> <value>`

## How do we inspect these values we have set?
`get`

## How about changing the value of a variable to null/no value?
`unset <variable>`

## Record screen
`spool`

## What command can we use to store the settings/active datastores from Metasploit to a settings file? This will save within your .msf4 (or .msf5) directory within your user directory and can be undone easily by simply removing the created settings file. 
`save`




