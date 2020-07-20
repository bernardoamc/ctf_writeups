# Burpsuite
#pentest/tryhackme

## TL;DR;
* **Proxy** - What allows us to funnel traffic through Burp Suite for further analysis
* **Target** - How we set the scope of our project. We can also use this to effectively create a site map of the application we are testing.
* **Intruder** - Incredibly powerful tool for everything from field fuzzing to credential stuffing and more
* **Repeater** - Allows us to 'repeat' requests that have previously been made with or without modification. Often used in a precursor step to fuzzing with the aforementioned Intruder
* **Sequencer** - Analyzes the 'randomness' present in parts of the web app which are intended to be unpredictable. This is commonly used for testing session cookies
* **Decoder** - As the name suggests, Decoder is a tool that allows us to perform various transforms on pieces of data. These transforms vary from decoding/encoding to various bases or URL encoding.
* **Comparer** - Comparer as you might have guessed is a tool we can use to compare different responses or other pieces of data such as site maps or proxy histories (awesome for access control issue testing). This is very similar to the Linux tool diff.
* **Extender** - Similar to adding mods to a game like Minecraft, Extender allows us to add components such as tool integrations, additional scan definitions, and more!
* **Scanner** - Automated web vulnerability scanner that can highlight areas of the application for further manual investigation or possible exploitation with another section of Burp. This feature, while not in the community edition of Burp Suite, is still a key facet of performing a web application test.

## Configuration for Firefox:
1. Go to: **about:preferences#general**
2. In Manual proxy configuration set:
	1. HTTP Proxy: 127.0.0.1
	2. Port 8080 (same port as going to the Burp Proxy tab -> options)
3. Navigate to a site like http://neverssl.com/ to see if everything is working
4. Download the SSL certificate by going to: http://burp/ and clicking on CA Certificate
5. In firefox go to: **about:preferences#general** and search for **Cert**
6. Click **View Certificates**, then **Authorities** then **Import**. From here, go to where you downloaded Burps file (and select it). Select the **both trust checkboxes** (this is important otherwise it will not work) and then click ok.
7. Open  https://google.com and everything should be working

## Intercepting only the appropriate domain name
Proxy -> Options -> Scroll down to Intercept Client Requests -> Click Add -> Select the match type as domain name and set an IP or domain address

## Exploiting a certain request
1. After the request is intercepted right click on it and select **Send to Intruder**
2. In the **Positions** burp will identify potential fields that can be targeted
3. There are 4 types of:  [Attack types](http://www.theitcareer.com/site/?p=1696) 
	1. Sniper: allows a single payload tried against every input field you select
	2. Battering Ram: tries a wordlist across all selected inputs at the same time
	3. Pitch Fork: Has 2 wordlists, matches first word with first/second/.. field in 2nd list
	4. Cluster Bomb: tries every combination of the the selected wordlists.
4. We can modify the automatic fields list with **Clear $** and **Add $** respectively
5. In the **Payloads tab**, we can see the dropdown for Payload type.
	1. There are many different types of payload types which you can read about   .** [here](https://portswigger.net/burp/documentation/desktop/tools/intruder/payloads/types) 
6. We can load wordlists by clicking in **Load**
7. And start attacking by going to the **Options tab** -> **Start Attack**






