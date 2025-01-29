Author: Troy Haynes
# Crawler
Ref: https://portswigger.net/burp/documentation/scanner/crawling
- Overview of how crawler works, go through config library
	- why its better than other tools
	- how it crawls from the application state when it found the link vs just going to the link
		- revisits the full path history,  instead of just going directly to the URL
		- **incy wincy** setting doesn't do this. Useful for more "dumb" web apps, instead it will just paste all the urls in the browser
			- uses burps cookie jar
			- make sure to put session killing links out of scope of the crawl such as logout pages (will null the session cookie its using)
	- if the web app is a SPA (uses client-side routing) make sure to enable the "Application uses fragments for routing" option

# Scanner
#### JavaScript Analysis
- **static** -  It identifies the tainted sources that are potentially controllable by an attacker, and the dangerous sinks that could be used to perform an attack. It also analyzes data flows through the code, to find potential paths for malicious data to be propagated from a tainted source to a dangerous sink.
	- finds some vulnerabilities that dynamic analysis cannot by identifying code paths that could be executed in the right circumstances
	- more prone to **false positives**   because the scan sees some combinations of code branches as executable when they are not and it doesn't understand custom data validation logic

- **Dynamic**  -  loads the responses into an embedded headless browser. It injects payloads into the DOM at locations that are potentially controllable by an attacker, and executes the JavaScript within the response. It  monitors the dangerous sinks that could be used to perform an attack and identifies any injected payloads that reach those sinks
	-  Provides concrete evidence of a vulnerability
	- more prone to **false negatives** in situations where the tainted data that it injects doesn't reach a sink

### Insertion points

Burp Scanner uses insertion points to place payloads into various locations within requests. It represents a piece of data within a request that might be processed by the server-side application.
- applies encoding to payloads based on the insertion point type to make sure that the raw payloads reach the relevant application functions

#### Modifying Parameter locations

Some applications place an input into one type of parameter, but still process the input if it is submitted in a different type of parameter. This happens because some of the platform APIs that applications use to retrieve input from requests do not process the type of parameter that holds the input. 
- Some application security measures, such as firewalls, might apply to the original parameter type only

Burp Scanner can change the insertion point parameter types. This creates requests that might bypass protections and reach vulnerable application functionality. 
- if a payload is submitted within a URL query string parameter, it can also submit corresponding payloads within a body parameter and a cookie header

## Scan policies

Can create new custom scanning configurations, should make a scan policy for each big web vulnerability (mostly injection vulns)
    - EX: XSS, SQLi, Command Injection/Path traversal, XML injection, ETC
    
Example scan policies for **XSS**:
1. Go to **Settings** -> **Configuration Library** -> **New** -> **Auditing**
2. Example settings for **XSS** optimization
	- **Issues Reported** -> Uncheck all issues -> select all the XSS issues
	- **Handling Application Errors During Audit** -> change to the 2,2,15 to 5,5,30
	- **Insertion Point Types** -> deselect Cookie parameter values (hard to prove on actual bug bounty)
	- **Modifying Parameter Locations** -> select URL to body and Body to URL
	- **Frequently Occurring Insertion Points** -> deselect URL parameter values and body parameter values
	- **Misc Insertion Point Options** -> change to 5

#### Example usage:

Can **FUZZ** a specific spot that you identified could be suitable  by:  (more granular approach, can also just run a new scan from a base URL)
1. right clicking on the request -> scan defined insertion point
![[Pasted image 20240415155053.png]]
2. on scan configuration, select your desired policy
![[Pasted image 20240415155146.png | 600]]

3. monitor the scan & results on the dashboard

## bchecks
https://github.com/PortSwigger/BChecks

- Uses templates (similar to Nuclei) to run checks

some examples for using bchecks:
- Detecting services running on the system
- Collecting resources from the target’s HTTP responses
- Performing automated recognition of assets
- Scanning input sections for potential vulnerabilities
- Automatically creating your own wordlists
- Discovering errors in the application
- Scanning for CVEs

To run a BCheck scan:
1. enable the bcheck templates you want to use
	1. go to extensions -> bchecks -> select the templates for use
	2. go to dashboard ->  new scan ->  scan configuration -> use a custom configuration
	3. select Audit Checks - Bchecks only
	4. configure other settings (scope of scan, application logins, resource pool)
	5. run scan

# Steps to setup private collaboration server
1. get the location of the burp suite pro jar file
2. run the command `java -jar <location>/burpsuite_pro.jar --collaborator-server` 
	1. this will use the default configuration of collaboration can also specify a `.config` file for customization
	2. to use burp infiltrator you need DNS so you would have to set that up in a custom `.config` file if you wanted to use
3. in burp suite go to settings -> Project -> collaborator
4. select the use a private collaborator radio button
5. input the IP address of your host machine you started the server on
6. run health check (encrypted services and DNS won't work on the default setup)
# Extensions
## Collaborator everywhere

Collaborator Everywhere is a simple but useful burp extension dedicated to SSRF vulnerabilities Developed by James Kettle will inject “non-invasive” headers, designed to reveal backend systems by causing pingbacks to Burp Collaborator.
- need to have internet access for default collaboration server or **setup a private server**
- 
## Content Type Converter
This extension converts data submitted within requests between various common formats:
- JSON To XML
- XML to JSON
- Body parameters to JSON
- Body parameters to XML
- 
This is useful for discovering vulnerabilities that can only be found by converting the content type of a request. For example, if an original request submits data using JSON, we can attempt to convert the data to XML, to see if the application accepts data in XML form. If so, we can then look for vulnerabilities like XXE injection which would not arise in the context of the original JSON endpoint. 

It might also be possible to find vulnerabilities behind web application firewalls or other filters that assume the incoming data is in a specific format, while the application tolerates data in other formats.

## Autorize
The extension automatically repeats every request with the session of the low privileged user and detects authorization vulnerabilities. It is also possible to repeat every request without any cookies in order to detect authentication vulnerabilities in addition to authorization ones. It then logs the status of these attempts in a color-coded table.
- It is sufficient to give to the extension the cookies of a low privileged user and navigate the website with a high privileged user
- helps with detecting **IDOR** vulnerabilities

To setup:
1. create 2 accounts on the web application
2. Login to one of the accounts (preferably the one with high access unless they are both the same access levels)
3. Get the cookie/session value by intercepting the response or proxy history
4. go to the autorize tab -> and paste the value here: 
![[Pasted image 20240403142906.png  | 400]]
5. on the "interception filter" click the "in scope items only" then add
6. now login to the web application as the other user account and walk/browse/test the web app

## 403BYPASSER

 When you get a server responding with a status code “403 Forbidden” for a directory, this means `you should try to bypass this restriction`. By using PassiveScan (default enabled), each 403 request will be **automatically** scanned by this extension, so just add to burpsuite and if its successful will show up in the issues.
## JavaScript Analysis

### JS Miner
- can identify interesting items in JS files
- can run it actively or passively. Active will send requests to the target seeing if a JS source map file is available
### **JS Link Finder** 
- extract URLs and their components from JavaScript files to identify further attack surface area
	- can right click on a target -> passively scan host -> burp will also use the JS link finder extension
	- has issue extracting **ALL** links from the JS files it seems
### [GAP](https://github.com/xnl-h4ck3r/GAP-Burp-Extension) 
 you can right click on entire scope and it will grab links, endpoints, and parameters not only from .JS files but also inline JavaScript
	- other tools do this but only on **.JS** files, not inline JS
	- builds a custom contextual wordlist










