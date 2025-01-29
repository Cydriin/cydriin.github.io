
#### GitHub repo of great BB tools
https://github.com/vavkamil/awesome-bugbounty-tools

----

** Not Burp extensions but placed here for now  until i get more organized**

**[jsluice](https://github.com/BishopFox/jsluice)** - made by TomNomNom - can extract content from JavaScript that is otherwise missed by regex-based URL/path and secret scanning tools
- can download JS files from something like **uproot-js** first 
- pipe to **jq** command for easier parsing

[trufflehog](https://github.com/trufflesecurity/trufflehog)- Find and verify credentials

[gron](https://github.com/tomnomnom/gron) - made by TomNomNom - makes JSON easily greppable

[LinkFinder](https://github.com/GerbenJavado/LinkFinder)- finds endpoints in JavaScript files

----

#### **Param Miner** 
- FUZZ parameters in HTTP requests
- Makes it easy to find potential vectors for a web cache poisoning attack. It's capable of guessing up to 65,000 parameter names per request

----

#### **Turbo Intruder**
- way faster intruder utilizing HTTP pipelineing (built by James Kettle)
- set the pipeline variable to **TRUE** to utilize HTTP pipelineing

Video by James Kettle that goes in depth for how to use and features
- https://www.youtube.com/watch?v=vCpIAsxESFY

----

#### **Active Scan++**

----
#### **JS Miner** 
- can identify interesting items in JS files
- can run it actively or passively. Active will send requests to the target seeing if a JS source map file is available

----

#### **JS Link Finder** 
- extract URLs and their components from JavaScript files to identify further attack surface area
	- can right click on a target -> passively scan host -> burp will also use the JS link finder extension
	- has issue extracting **ALL** links from the JS files it seems

----

#### [uproot-js](https://github.com/0xDexter0us/uproot-JS/)
- use to download in scope JavaScript files from Burp to your local system
- wont grab source maps
- might not grab all?

----

#### **Reshaper**
- can be configured with a set of rules that take some action based on some condition with HTTP requests and responses
- **Example use case:** can highlight, add comments, etc to the HTTP responses based if it **Matches the text: Password**
	- will show up in proxy history

----

#### **Logger++** 
stores all Burp's requests and responses in an easily exported and sortable table

----

#### **BChecks** 
can create custom scan/audit checks without having to create a whole extension
- can create checks for specific files, features, etc
- uses templates using portswiggers "format" check documentation 

----

#### **Autorize**
It is sufficient to give to the extension the cookies of a low privileged user and navigate the website with a high privileged user. The extension automatically repeats every request with the session of the low privileged user and detects authorization vulnerabilities. It is also possible to repeat every request without any cookies in order to detect authentication vulnerabilities in addition to authorization ones. It then logs the status of these attempts in a color-coded table.

#### **AuthMatrix**
gives a simple matrix grid to define the desired levels of access privilege within an organization/web app. It then tests each function for different types of user. One of AuthMatrix's best features is a "chain" mode, which enables [cross-site request forgery (CSRF)](https://portswigger.net/web-security/csrf) tokens to be grabbed from requests and attached to subsequent attempts.


----

#### **Backslash Powered Scanner**
helps find interesting items to investigate manually by mimicking human intuition (built by James Kettle)

----

#### **Upload Scanner**
It has the ability to upload a number of different file types, laced with different forms of payload. Upload Scanner can test for vulnerabilities including server-side request forgery (SSRF) and XML external entity (XXE) injection using common file types like JPEG, PDF, and MP4 as vectors.

----

#### **Retire.js**
a repository of JavaScript libraries that include known bugs, and this dedicated plugin makes it available within Burp Suite Pro as a passive scan check

----

#### **JSON Beautifier**
gives you the option to either "beautify" or "minify" (crunch back up) your target's JSON content

----

#### [GAP](https://github.com/xnl-h4ck3r/GAP-Burp-Extension) 
burp extension where you can right click on entire scope and it will grab links, endpoints, and parameters not only from .JS files but also inline JavaScript
	- other tools do this but only on **.JS** files
	- builds a custom contextual wordlist

----


# Not in BApp Store

#### nowafpls

[Github Link](https://github.com/assetnote/nowafpls)

Used for bypassing WAFs

**Usage:**
1. Send any request to the repeater tab that you'd like to bypass WAF for.
2. Put your cursor in a place you would like to insert junk data.
3. Right click -> Extensions -> nowafpls
4. Select how much junk data you want to insert
5. Click "OK"

**nowafpls** will insert junk data as per the type of request (URLEncoded/XML/JSON) automatically.
