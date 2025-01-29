
## Content Discovery

* First thing is to always walk the app manually. Proxy through burp, and try to do as much as you can manually. Good starting point to build out the site tree in burp for spidering later on. Try for ~30 min.
* Content discover consists of trying to discover all the routes, paths, parameters, functions, and files of an application. (Not just directory brute forcing)
### Spidering
* also used in recon phase with CLI apps
* burps crawler will attempt to go through every link and build out the site tree. Good with dynamic pages utilizing JavaScript
### Tools
* Turbo Intruder, Gobuster, ffuf, Feroxbuster, dirsearch, wfuzz
### Tech
* using lists based on the type of technology the application uses
	* can be at its framework level, the web server level, or whatever part of the application your doing content discovery on (API)

wordlists used based on technology - https://wordlists.assetnote.io/
![[Pasted image 20231121104646.png | 900 x 375]]

technology to host mappings wordlists
![[Pasted image 20231121105355.png | 450]]

### Known Pathing
If the application is open source software or paid software (COTS), we can install the software locally and understand the pathing and underlying non-custom code
#### Local install
If we can install the software locally (open source project). We first download the project, then put this [tool](https://github.com/danielmiessler/Source2URL/tree/master) at the web route of our local install and point it at Burp Suite. Then run the shell script. It will pull out all the directories and tunnel through burp to our target domain. 
#### Demos
If the application is paid software or COTS the vendor could have a live demo instance that we can potentially use. We can proxy with burp to get all the paths for that software.
	- important to get access to admin functionality of the software to grabs those paths, routes, and parameters
	- can help find access control vulnerabilities when testing the actual web application
#### Installed and leaked
developers might post container images on Dockerhub. Search through Dockerhub to find installed instances of the software which in the containers might have the source code in the container for further analysis. Can also search on GitHub.

### Custom wordlists

* using custom wordlists to discover paths and routes. Taking contextual words related to the application to build a wordlist.
	* tools such as CEWL, GAU
	* ex: `echo bugcrowd.com | gau | wordlistgen | sort -u`
	* tomnomnom's guide https://tomnomnom.com/talks/wwwww.pdf

### Historical 

searching sources that archive URL data using sites such as Wayback machine, alien vault, common crawl, and URL scan.
- tools such as GAU used before, new tool [waymore](https://github.com/xnl-h4ck3r/waymore) goes more in depth now
	- The biggest difference between **waymore** and other tools is that it can also **download the archived responses** for URLs
- ex: `python3 waymroe.py -i domains.txt`

### Tips

* For **401 Not Authorized** responses, it can be beneficial to recursively brute force that path. Resources past that path might have have the same access controls replied. Also 401 replies should be investigated with **waybackmachine** to see if authentication was ever not applied

* Often mobile application binaries can contain the same pathing of the same website, usually this is an API hanging off the main domain.
	* can use [APKleads](https://github.com/dwisiswant0/apkleaks) to parse out paths from the APK file to get additional route and API endpoints and parameters

* try and change to maybe a mobile user agent in tools or burp suite using the replace function for all requests, because it could allow access to more content

### JavaScript Analysis

* Gareth Heyes - JavaScript for hackers book (I have downloaded on google drive)
* **Most** of the time looking for  endpoints, parameters, routes, secrets, and domains
* if the application is a heavy JavaScript framework (angular, NodeJS)  most of the functionality with routes and parameters are in the client-side JS code

[GAP](https://github.com/xnl-h4ck3r/GAP-Burp-Extension) - burp extension where you can right click on entire scope and it will grab links, endpoints, and parameters not only from .JS files but also inline JavaScript
	- other tools do this but only on **.JS** files
	- builds a custom contextual wordlist
	- can take the links run through something like httpx while proxying through burp to get the links in ur site tree on burp
		- also we can take our custom **iptables** alias(or Proxychains) to proxy the traffic through burp if the tool doesn't have a proxy feature

[JSMiner](https://portswigger.net/bappstore/0ab7a94d8e11449daaf0fb387431225b) - Burp extension used to look for hard-coded secrets in JS files. Can be used passively and actively.
	- not just regex, it uses (Shannon) entropy function for things that might be interesting

[metasec.js](https://github.com/LewisArdern/metasecjs)-  Uses static code source analysis on downloaded JavaScript files
	- wrapped with **npm-audit**, **yarn-audit**, and **semgrep** engines

[webpack exploder](https://spaceraccoon.github.io/webpack-exploder/) - Unpacks webpacked JavaScript files
	- common with mobile

# The Big Questions

* First question he asks himself is "How does this application **pass data**?"
	* does it use a: **resource**, **parameter**, **value** format
		* ex: `https://app/resource?parameter=value&param2=value`
	* or does it use a **RESTful** format (uses **routes**)
		* all done with `/` doesn't use parameters (how restful APIs work)
		* ex: `https://app.com/route/resource/sub-resource/...`
	* Important for FUZZing
		* If the parameter/value format, usally FUZZing the parameter or value
		* If route format, usually FUZZing the sub-resource or pass that

* Next question, "How/where does the application **talk about users**?". Understanding how users are referenced and where in the application is pivotal to finding classes of bugs such as: **Access**, **Authorization**, **Logic**, and **Information Disclosure** 
	* If I'm querying information does it just use my **cookie** or does it use other information such as: **username**, **UID**, **email**, **UUID** ?
		* If it does, use one of those, can we specify somebody else's? (for **IDOR** bugs)

* Does the application have **different user levels** or **multi-tenancy**?
	* different permissions
		* The application most likely has different parts available to it based on the role of the logged in user
		* coding that access control for multi-tenancy sites can be difficult. Which leaves a lot of access bugs

- Does the application have a **unique threat model**?
	- what data (besides like normal PII) in the app is more **valued** or **unique** to that app
	- ex: a **stream key** in twitch.tv can give you full access to a streamers account

* Has there been past **security research and vulnerabilities** on this application?
	* can check hackerone reports, blogs, google
	* can do regression testing
		* can build nuclei templates based on past reports

* How does the application **store data**?
	* where are image and file uploads going?
	* what kind of database do I think they are using?
		* can i guess based on type of application and framework? common developer practice? blog post? linkedin job post?
			* can help when trying to do SQL injection

# Heat Mapping
Jason Haddix's common vulnerability locations based on his experience
### File uploads
Common places you see a vulnerability is whereover they allow you to upload files (any format such as images or documents)
	- important note is if a document upload exists that data comes down to XML data. So can be subject to XML based vulnerabilities like XXE
	- payloads like PHP web shells isn't as common nowadays
	- image XSS 
		- via binary of the image, such as using a HEX editor to input a payload
		- via the file name
		- in the metadata of the image
		- try something like a popup alert and also a blind/OOB payload
![[Pasted image 20231121162153.png | 350]]
### Content Types

* anytime a request or response includes a **multipart-form** or it returns or sends **XML**, or sends or returns **JSON** data
	* something to just be aware of

![[Pasted image 20231121162912.png | 350]]

### APIs

Usually bugs are more inclined to be that the API itself did not enforce authentication to pull down sensitive data. Less and less injection attacks in APIs
### Accounts

This is where a lot of data ends up being **stored** or **persisted** (prime for **stored XSS**)
	- can turn self XSS into blind XSS (ex: calling customer service having them access your account)
Usually where you can set up additional integrations. The integrations that allow applications to connect to each other can be subject to various types of vulnerabilities
	- prime for **SSRF**

![[Pasted image 20231122081719.png | 250]]
### Errors

When you see in your proxy many errors coming back you have the ability through logger in burp to understand what triggered that error
	- If it was certain **meta character**, or **injection attempt**, you have the opportunity to play with that request in repeater or intruder to see if you can exploit an **injection** or cause a **DOS**.
### Passing Paths

Anytime you see an application passing a path or URL as part of a value of a parameter or in a route. If the web application is taking a path or URL it need to parse that in some way.
	- URL and path parsers are known for being subject to **redirect** vulnerabilities and **SSRF**

### Help Chatbots

* when you see a chat bot that is designed to help you connect with the customer service of the application, it should be tested for blind XSS

#### AI Chatbots
many companies are rolling out helpful chat bots like the help chatbot but have AI enabled
	- the LLMs are trained on the business's data
	- new skill for the next generation will be **prompt injection** to try and smuggle out data from the company
	- https://gandalf.lakera.ai/ - CTF using prompt injection
		- also https://learnprompting.org/

## Web Fuzzing
Same as **Dynamic Scanning** which will use different payload for insertion points. CVE scanning is static and will just scan for the known issue.
### Burp Policies
In Burp suite, create new custom scanning configurations
* should make a scan policy for each big web vulnerability (mostly injection vulns)
	* XSS, SQLi, Command Injection/Path traversal, XML injection, ETC
* Can this FUZZ a specific spot on a request by right clicking -> scan defined insertion point -> scan template you want
#### XSS Config
* **Issues Reported** -> Uncheck all issues -> select all the XSS issues
* **Handling Application Errors During Audit** -> change to the 2,2,15 to 5,5,30
* **Insertion Point Types** -> deselect Cookie parameter values (hard to prove on actual bug bounty)
* **Modifying Parameter Locations** -> select URL to body and Body to URL
* **Frequently Occurring Insertion Points** -> deselect URL parameter values and body parameter values
* **Misc Insertion Point Options** -> change to 5
* **JavaScript Analysis** -> Disable static analysis techniques (doesn't usually work), change time to 15 seconds

#### SQLi Config
* **Issues Reported** -> deselect (select all by ctrl + click,  then right click -> disable) -> Select SQL injections except the client-side ones
	* Can make specific SQL templates for deferent types of database to save time we already know it from previous info gathering
		*  right click on the issue -> edit detection methods -> deselect the DBs you don't want (MYSQL, Oracle)
* **Handling Application Errors** -> change to 10, 10
* **Insertion Point Types** - > deselect cookie parameter values, parameter name
* **Modifying Parameter Locations** -> select URL to body, Body to URL
* **Frequently Occurring Insertion Points** -> deselect URL parameter values, body parameter values
* **MISC Insertion Point Options** -> change to 15
* **JavaScript Analysis** -> deselect static analysis techniques

### Backslash Powered
More in-depth dynamic scanning can be done by trying to elicit errors from the application. Using the burp **backslash powered scanner** extension you can fuzz routes and parameters and elicit the errors. 
	- Sends special characters to elicit the errors
	- sends special character then sends another requests with a backslash (\\) and  special character to try and escape the special character 

### SSWLR - Interpreting Results
How to interpret results from fuzzing

- Did any of these change? If so look into it!
**S**ensitive = **S**tatus Code
**S**ecrets = **S**ize
**W**ere = **W**ord Count
**L**eaked = **L**ines
**R**ecently = **R**esponse Time

## Vulnerability Types
Not Complete, just his experience

### XSS
- best app tester he's seen is **Ashar Javed** ([training](https://hss-opus.ub.ruhr-uni-bochum.de/opus4/frontdoor/deliver/index/docId/4522/file/diss.pdf) for manually testing XSS)
	- also Gareth Hayes who has the JavaScript for Pentesters book

**Ashar Javed's** methodology 
![[Pasted image 20231122115521.png | 900]]

![[Pasted image 20231122115602.png | 900]]

#### Blind
- since **XSSHunter** is depreciated, **BXSSHunter** is the closest to it

#### Common parameters 
```text
q, s, search, id, lang, keyword, query, page, keywords, year, view, email, type, name, p, month, image, list, type, url, terms, categoryid, key, login, begindate, enddate
```

### IDOR

[**InsiderPHD**](https://www.youtube.com/watch?v=2WzqH6N-Gbc) did a really good video for using the **Autorize** burp extension, which is the best extension for **IDORs**

#### Common parameters 
```text
id, user, account, number, order, no, doc, key, email, group, profile, edit, REST numeric paths
```

### SSRF
Sometimes SSRF can be as simple as embedding an image tag with your Burp collaborator URL as the source `<img src="https://burpcollaborator"/>` but sometimes you just shove that EVERYWHERE 

You can grab all URLs from your target, from **GAU**, or **Waymore**, pass them to **qsreplace** and add the collaborator URL:
```bash
cat all_urls.txt | grep "=" | qsreplace http://burpcollaburl > ssrf.txt
```

```bash
cat ssrf.txt | httpx -fr
```

if a request is made back to your collab server, to find which URL it was will use  **binary search** method
	- will cut the **ssrf.txt** file in half and send the requests again and see if the request was still made.
		- if yes, will keep cutting the list in half until only 2 URLs are left and you can figure out which one
		- if no, will use the other half not used, then repeat the process

- best resource in SSRF for alternate encodings **once** you find SSRF is still the payloadsallthethings

#### Common Parameters
```text
dest, redirect, uri, path, continue, url, window, next, data, reference, site, html, val, validate, domain, callback, return, page, feed, host, port, to, out, view, dir, show, navigation, open
```

- also commonly found in webhooks, XML, DOC uploads, and headers

### XXE

- best resource for XXE payloads is [payload box repo](https://github.com/payloadbox/xxe-injection-payload-list)

Commonly where you think that an XML file will be processed or an XML portion of the application is sending and receiving data

### File Uploads

- best [presentation](https://www.youtube.com/watch?v=QyPbPGyTPxI) was by **Soroush Dalili**, most of this presentation was uploaded to the OWASP cheatrsheet.
	- old but still relevant 

### SQLi

* best automation tool for this has always been **SQLmap** until this year. Now new tool [**ghauri**](https://github.com/r0oth3x49/ghauri) seems to do better on blind injections, time injections, and WAF invasion
- new tool [HBSQLI](https://github.com/SAPT01/HBSQLI) performs blind SQL injection in headers better than SQLmap
#### Common Parameters
```text
id, select, report, role, update, query, user, name, sort, where, search, params, process, row, view, table, from, sel, results, sleep, fetch, order, keyword, column, field, delete, string, number, filter
```























































