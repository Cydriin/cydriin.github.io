#### GitHub repo of great BB tools
https://github.com/vavkamil/awesome-bugbounty-tools

**Not Burp extensions but placed here for now  until i get more organized**

**[jsluice](https://github.com/BishopFox/jsluice)** - made by TomNomNom - can extract content from JavaScript that is otherwise missed by regex-based URL/path and secret scanning tools
- can download JS files from something like **uproot-js** first 
- pipe to **jq** command for easier parsing

[trufflehog](https://github.com/trufflesecurity/trufflehog)- Find and verify credentials

[gron](https://github.com/tomnomnom/gron) - made by TomNomNom - makes JSON easily greppable

[LinkFinder](https://github.com/GerbenJavado/LinkFinder)- finds endpoints in JavaScript files
## IDOR/Access Controls
#### **Autorize**

It is sufficient to give to the extension the cookies of a low privileged user and navigate the website with a high privileged user. The extension automatically repeats every request with the session of the low privileged user and detects authorization vulnerabilities. It is also possible to repeat every request without any cookies in order to detect authentication vulnerabilities in addition to authorization ones. It then logs the status of these attempts in a color-coded table.

To setup:
1. create 2 accounts on the web application
2. Login to one of the accounts (preferably the one with high access unless they are both the same access levels)
3. Get the cookie/session value by intercepting the response or proxy history
4. go to the autorize tab -> and paste the value here: 
![[Pasted image 20240403142906.png  | 400]]
5. on the "interception filter" click the "in scope items only" then add
6. now login to the web application as the other user account and walk/browse/test the web app

#### **Auth Analyzer**

Enables the creation of multiple sessions simultaneously to help you test horizontal and vertical privilege escalation by navigating through the web application with a high privileged user and let the Auth Analyzer repeat your requests for any defined non-privileged user. With the possibility to define Parameters the Auth Analyzer is able to extract and replace parameter values automatically. 

To setup:
	1. Start by creating two or more **users’ profiles** as needed in Auth Analyzer by clicking on the button **New Session** then copy/paste cookies and headers for auth
	or
	1.  for each session,  go on your **_HTTP Proxy_ tab**, right click on your session headers (cookie, authorization…) > **_Append Header_** > **_Create New Session_**

With this plugin, you have the possibility to define which parameters are being replaced or extracted before the request for the given session is repeated. It’s like the “match and replace” feature in a proxy on steroids. Several modes are available:
- **_Auto Extract_**: the parameter value will be extracted if it occurs in a response which contains the defined field name (works in _Set-Cookie Header, HTML Document Response, JSON Response_),
- **_Static Value_**: if you want to define a static parameter value,
- **_From String To String_** : useful when you want to replay a login process to auto extract and renew your sign-in, no need to copy/paste again your auth headers,
- **_Prompt for Input_**: auth analyzer will ask you to enter the value (for example in the scenario of an 2FA authentication).







## Fuzzing
----

#### **Param Miner** 
- FUZZ parameters in HTTP requests
- Makes it easy to find potential vectors for a web cache poisoning attack. It's capable of guessing up to 65,000 parameter names per request

----

#### **Turbo Intruder**
- way faster intruder utilizing HTTP pipelineing (built by James Kettle)
- set the pipeline variable to **TRUE** to utilize HTTP pipelineing

----

## JavaScript Analysis

#### **JS Miner** 
- can identify interesting items in JS files
- can run it actively or passively. Active will send requests to the target seeing if a JS source map file is available
#### **JS Link Finder** 
- extract URLs and their components from JavaScript files to identify further attack surface area
	- can right click on a target -> passively scan host -> burp will also use the JS link finder extension
	- has issue extracting **ALL** links from the JS files it seems
#### [uproot-js](https://github.com/0xDexter0us/uproot-JS/)
- use to download in scope JavaScript files from Burp to your local system
- wont grab source maps
- might not grab all?

#### [GAP](https://github.com/xnl-h4ck3r/GAP-Burp-Extension) 
burp extension where you can right click on entire scope and it will grab links, endpoints, and parameters not only from .JS files but also inline JavaScript
	- other tools do this but only on **.JS** files
	- builds a custom contextual wordlist

#### **Retire.js**
a repository of JavaScript libraries that include known bugs, and this dedicated plugin makes it available within Burp Suite Pro as a passive scan check

## Automation

#### Autorepeater
Like an upgraded version of burps built in "match and replace feature". Can also be used instead of Autorize/Auth Analyzer as you have more control on what you can customize

 examples of which rules can be created:
- Add new headers (x-forwarded-for, x-forwarded-host, origin…),
- Remove authentication header or cookie (JWT for example),
- Replace value like _user_ to _admin_, _false_ to _true_,
- Add JSON parameters,
- Change HTTP Method (PUT to POST or vice versa),
- Add GET or POST parameters,
- Add _.old_ or .back to files extension (_.php.old_, _.jsp.old_…),
- Remove CSRF token,
- Math parameters for Open Redirect, reflected XSS or SSRF
- Try to bypass “403 Forbidden” by adding specific headers or things like “.;/” in URL
- Chain AutoRepeater with Hackvertor extension

Has a “Log Highlighter” feature which can be used with conditional match, to highlight only what you exactly want. For example, only match “200 OK” status or if HTTP method is “PUT” or “POST”. To configure this, go on _Logs_ tab and _Log Highlighter_ click on _Add_.

Tips:
- Don't activate autorepeater until you're ready to start browsing.
- Ensure **Extender** is not using cookies from Burp's cookie jar (**Project Options > Session**).
- Check early to ensure your replacements are working as expected.
- Tabs and configuration are preserved after a restart, but data is lost.

**Testing Unauthenticated User Access**
To test whether an unauthenticated user can access the application, configure one rule under Base Replacements to **Remove Header By Name** and then match "Cookie".

**Testing Authenticated User Access**
To test access between authenticated users (e.g. low privilege to higher privilege), you'll need to define replacements for each of the session cookies used:

- Make note of the cookie names and values for the lower-privileged session.
- Configure a rule under Base Replacements for each cookie to Match Cookie Name, Replace Value. Match the cookie name, replace with the lower-privileged user's cookie.
- Repeat for as many roles as you'd like to test.
- Browse the application as the highest-privileged user.
- Review the results.

**Reviewing User Access Results**
- Sort by URL, then by Resp. Len. Diff.. Items with a difference of 0 and identical status codes are strong indicators of successful access.
- Using Logs > Log Filter configure exclusions for irrelevant data (e.g. File Extension = (png|gif|css|ico), Modified Status Code = (403|404)).
- Review the results and manually investigate anything that looks out of place.

---
#### HopLa

https://github.com/synacktiv/HopLa

All the power of PayloadsAllTheThings, without the overhead. This extension adds **autocompletion** support and useful payloads in Burp Suite to make your intrusion easier.

#### **BURP-SEND-TO**

If you use other tools like SQLMap or your own custom scripts, this **Burp-Send-To** extension could be useful to add a little more automation when you want to export your query in ready-to-use mode.

![[Pasted image 20240403151127.png | 350]]

#### Stepper

Lets you create sequences of steps and define regular expressions to extract values from responses which can then be used in subsequent steps. For example, your target require multiple steps to authenticate (login form > 302 redirect > JWT in response), you can use Stepper to automatically extract the JWT and reuse it after. Can be used with **Hackvertor** tags with for additional functionality.

**Usage:**
1. Create a new sequence. Double-click the title to set a suitable name.
2. Optional: Configure the global variables to use for the sequence.
3. Add your steps to the sequence manually, or using the context menu entry.
4. Optional: Define variables for steps, providing a regular expression which will be used to extract the values from the response. Tip: You can execute a single step to test your regular expressions using the button in the top right.
5. Execute the entire sequence using the button at the bottom of the panel.

**Variables:**  
Variables can be defined for use within a sequence. Variables consist of an identifier and a regular expression, or in the case of initial variables defined in the Globals tab, an identifier and value. Step variables, defined with a regular expression, have their values set from the response of the step in which they are defined. The variable is then available for use within the request of subsequent steps after their definition. However, Global variables, defined with a literal initial value, can be used throughout the sequence.

Both step and global variables may be updated in later steps after their definition.

**Regular Expression Variables:**  
Variables which are defined with a regular expression are updated each time the step in which they are defined is executed. The regular expression is executed on the response received, with the first match being used as the new value. If the defined regular expression has no groups defined, the whole match will be used. If the regular expression defines capture groups, the first group will be used. If groups are required but should not be used as the value, a non-capturing group may be used. e.g. (?:REGEX)

**Example:**  
Response: "Hello People, Hello World!"  
Expression: World|Earth, Result: World  
Expression: Hello (World|Earth)!, Result: World  
Expression: (?:Goodbye|Hello) (World)!, Result: World

**Variable Usage:**  
To use a variable in a request after it has been defined, either use the option in the context menu to copy the parameter to the clipboard, or manually insert it by including it as below:
`$VAR:VARIABLE_IDENTIFIER$`

#### ReshaperForBurp
https://github.com/synfron/ReshaperForBurp
Docs: https://synfron.github.io/ReshaperForBurp/

Extension to trigger actions and reshape HTTP request/response and WebSocket traffic using configurable Rules.

 **Example Usage**
- [Use a value from one HTTP message in a following HTTP message](https://synfron.github.io/ReshaperForBurp/Examples.html#tip1)
- [Redirect a request to a different server](https://synfron.github.io/ReshaperForBurp/Examples.html#tip2)
- [Change a value in a returned response](https://synfron.github.io/ReshaperForBurp/Examples.html#tip3)
- [Auto-respond to requests without first sending a request to an external server (response mocking)](https://synfron.github.io/ReshaperForBurp/Examples.html#tip4)
- [Drop a request so it is not sent to an external server (Works on all supported tools: Proxy, Repeater, Intruder, Scanner, Target, Extender (other extensions))](https://synfron.github.io/ReshaperForBurp/Examples.html#tip5)
- [Share or backup/restore Global Variables and Rules](https://synfron.github.io/ReshaperForBurp/Examples.html#tip6)

----
## MISC
## Content Type Converter
This extension converts data submitted within requests between various common formats:
- JSON To XML
- XML to JSON
- Body parameters to JSON
- Body parameters to XML
- 
This is useful for discovering vulnerabilities that can only be found by converting the content type of a request. For example, if an original request submits data using JSON, we can attempt to convert the data to XML, to see if the application accepts data in XML form. If so, we can then look for vulnerabilities like XXE injection which would not arise in the context of the original JSON endpoint. 

It might also be possible to find vulnerabilities behind web application firewalls or other filters that assume the incoming data is in a specific format, while the application tolerates data in other formats.
#### **Reshaper**
- can be configured with a set of rules that take some action based on some condition with HTTP requests and responses
- **Example use case:** can highlight, add comments, etc to the HTTP responses based if it **Matches the text: Password**
	- will show up in proxy history

#### **Hackvertor**
Designed to modify requests dynamically using tags. This functionality eliminates the need for scripting for each new feature, thereby saving time and boosting productivity. One of the perks of Hackvertor is the ability to create **custom tags**, which allows for the execution of advanced actions or repetitive tasks without having to rewrite scripts.

- **Tag-Based Modifications**: This feature simplifies changes made within the context of other Burp Suite features.
- **On-the-Fly Fuzzing**: Hackvertor supports fuzzing with modifications on-the-go, which is particularly useful with Burp Suite's Intruder tool.
- **Encoding and Decoding Support**: Hackvertor provides support for encoding and decoding, encryption and decryption of values.
- **IP Spoofing**: With Hackvertor, you can generate random IP addresses to bypass protection mechanisms and filters.
- **Signature Generation**: Hackvertor allows you to generate signatures effortlessly.
- **Fake Data Input**: You can fill in forms with fake datasets, which can be useful in various testing scenarios.

Example use cases: 
https://www.yeswehack.com/learn-bug-bounty/pimpmyburp-10-Hackvertor
https://trustedsec.com/blog/what-is-hackvertor-and-why-should-i-care

Example:
1. Highlight a section in a Repeater request
2. Right-click on the highlighted section
3. Select “Extensions” from the right-click menu - > Select “Hackvertor -> choose a **function**
4. Tags will get set around your selected text
5. If you want to see what a sent request looks like after the tags have been processed, view the logger tab in Burp 


#### **Logger++** 
stores all Burp's requests and responses in an easily exported and sortable table

----

#### **BChecks** 
can create custom scan/audit checks without having to create a whole extension
- can create checks for specific files, features, etc
- uses templates using portswiggers "format" check documentation 


----

#### **Backslash Powered Scanner**
helps find interesting items to investigate manually by mimicking human intuition (built by James Kettle)

----

#### **Upload Scanner**
It has the ability to upload a number of different file types, laced with different forms of payload. Upload Scanner can test for vulnerabilities including server-side request forgery (SSRF) and XML external entity (XXE) injection using common file types like JPEG, PDF, and MP4 as vectors.


----

#### **JSON Beautifier**
gives you the option to either "beautify" or "minify" (crunch back up) your target's JSON content

#### **AES KILLER**
https://github.com/Ebryx/AES-Killer

More and more web apps are using AES encryption (mobile application principally) but Burp can’t decrypt the traffic easily. This extension will can do that. You still need to have a **Secret Key** and **Initialize Vector** which means you need to make little reverse on your target firs

#### **403BYPASSER**

 When you get a server responding with a status code “403 Forbidden” for a directory, this means “_you should try to bypass this restriction_“. By using PassiveScan (default enabled), each 403 request will be **automatically** scanned by this extension, so just add to burpsuite and if its successful will show up in the issues.

#### **Active Scan++**

## SSRF
#### Collaborator everywhere

Collaborator Everywhere is a simple but useful burp extension dedicated to SSRF vulnerabilities Developed by James Kettle will inject “non-invasive” headers, designed to reveal backend systems by causing pingbacks to Burp Collaborator.