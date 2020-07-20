# ZAP
#pentest #pentest/tryhackme

### Running
```sh
$ zaproxy
```

### Setting the proxy
Tools -> Proxy -> Loca Proxies
We can get the certificate and add to Firefox through: Tools -> Proxy -> Dynamic SSL Certificates -> Save

### Why wouldn’t I use Burp Suite?

That’s a GOOD question! Most people in the Info-sec community DO just use Burp Suite. But OWASP ZAP has a few benefits and features that the Burp Suite does not and it’s my preferred program of the two. 

### What are the benefits to OWASP ZAP?
It’s completely open source and free. There is no premium version, no features are locked behind a paywall, and there is no proprietary code.
There’s a couple of feature benefits too with using OWASP ZAP over Burp Suite:

* Automated Web Application Scan: This will automatically passively and actively scan a web application, build a sitemap, and discover vulnerabilities. This is a paid feature in Burp. 
* Web Spidering: You can passively build a website map with Spidering. This is a paid feature in Burp.
* Unthrottled Intruder: You can bruteforce login pages within OWASP as fast as your machine and the web-server can handle. This is a paid feature in Burp.
* No need to forward individual requests through Burp: When doing manual attacks, having to change windows to send a request through the browser, and then forward in burp, can be tedious. OWASP handles both and you can just browse the site and OWASP will intercept automatically. This is NOT a feature in Burp. 

If you’re already familiar with Burp the keywords translate over like so:

![](ZAP/KpTw37QyRfu1WTf5rIGz4ZDwEPm7s4CutgFTSrFBsXJT97a8HT8HiIcuUciFCyVXxcfVmCMuX1TbhOAM2gZLBYgkzPQzpF3tcd_DpcudC6JElYRDoZhUcXeP4mtn83UeUa7gKnWG.png)

### Main screen:

![](ZAP/1351DB6F-8ABB-487A-B98B-AF6B90931433.png)

### Scanner modes

![](ZAP/18172C54-990B-4C5D-ABF9-9BE9F3A33ECB.png)