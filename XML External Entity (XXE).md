# XML External Entity (XXE)
#pentest/tryhackme

## What
It's a vulnerability that abuses features of XML parsers/data

## Types
There are two types of XXE attacks: in-band and out-of-band (OOB-XXE).

1. An in-band XXE attack is the one in which the attacker can receive an immediate response to the XXE payload.
2. out-of-band XXE attacks (also called blind XXE), there is no immediate response from the web application and attacker has to reflect the output of their XXE payload to some other file or their own server.

## Payloads
* [PayloadsAllTheThings/XXE Injection at master · swisskyrepo/PayloadsAllTheThings · GitHub](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection#classic-xxe)

## Attacks
Most of XXE attacks will make use of the !ENTITY attribute referencing external resources. They follow the format:

```xml
<!ENTITY entity-name SYSTEM "URI/URL"> 

/// Example:

<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY read SYSTEM 'file:///etc/passwd'>]>
<root>&read;</root>
```

