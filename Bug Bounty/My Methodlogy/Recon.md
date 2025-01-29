#### Recon checklist

- [ ] perform subdomain recon


using **subfinder**
```bash
subfinder -dL scope.txt -o subfinder_domains.txt
```

amass subdomain finder - active and brute force
```bash
amass enum -active -d gymshark.com -d gymshark.io -nocolor -brute -config ~/.config/amass/config.yaml -o amass_raw.txt
```

From Amass output will store all the domains in a text file. Def better way to do this but what i got for now

```shell
cat amass_raw.txt | grep ptr_record | awk -F 'ptr_record' '{print $2}' | awk -F ' ' '{print $3}' | tee -a amass_raw.txt | cat amass_passive_gymshark.txt| awk -F ' ' '{p  
rint $1}' | grep gym | sort -u | tee amass_domains.txt
```


From Amass output will store all the cname records into a file
	- this could be useful to get a good idea of  third-party services or providers. Knowing these can help identify external dependencies.
	- also can be used for subdomain takeover if the record still exist but the domain is no longer active
```shell
cat amass_raw.txt | grep cname_record | sed 's/(FQDN)//g' | grep gymshark | sort -u | tee amass_cname_records.txt
```


combining  subfinder and amass results

```bash
```


using the harvester (not sure if its worth it yet to use in conjunction with subfinder, need more testing)

sources.txt file
```text
baidu
bufferoverun
crtsh
hackertarget
otx
projecdiscovery
rapiddns
sublist3r
threatcrowd
trello
urlscan
vhost
virustotal
zoomeye
```

Combined the 2 commands below and performed cleanup on the files

command needs editing for 
```bash
export TARGET="lares.com"
cat sources.txt | while read source; do theHarvester -d "${TARGET}" -b $source -f "${source}_${TARGET}";done
cat *.json *.xml | jq -r '.hosts[]' 2>/dev/null | cut -d':' -f 1 | sort -u > "theHarvester_domains.txt" && rm -f *.json *.xml
```

##### DNS

using dnsrecon to try and perform a Zone Transfer
```bash
dnsrecon -a -d gymshark.com
```

**dnsx**

active hostnames from the list of found passive subdomains
```bash
subfinder -silent -d hackerone.com | dnsx -silent

	cat subdomains.txt | dnsx -silent
```

Resolve and print records for list of subdomains 
- can list any record or combination (`A`, `cname`,`MX`, `soa`, `ns`, `txt`, `axfr`, `any`, `srv`, `aaaa` )
`-resp` - will print domain + IP
`-resp-only` - will print only the IP address

```bash
subfinder -silent -d hackerone.com | dnsx -silent -a -resp

cat subdomains.txt | dnsx -silent -a -cname -resp-only
```

Extract subdomains from given network range using `PTR` query
```bash
echo 173.0.84.0/24 | dnsx -silent -resp-only -ptr
```

###### BF and permutation

using `dnsx` to bruteforce subdomains
```shell
dnsx -silent -d domains.txt -w dns_worldlist.txt
```

using `dnsx` to bruteforce TLDs
```shell
$ cat tld.txt

com
by
de
be
al
bi
cg
dj
bs
```

```bash
dnsx -d gymshark.FUZZ -w tld.txt -resp
```


using `alterx` for permutation then bf subdomains

`-enrich` - will populate known subdomains (around 600k)
```shell
cat subdomains.txt | alterx -silent > permutation_list.txt
```

or pipe into `dnsx`
```shell
cat subdomains.txt | alterx -silent -enrich | dnsx -silent
```
#### getting alive domains

`-nf` - does both http and https
```bash
httpx -l urls.txt -silent -random-agent -t {input number} -nf -retries 2 -o alive_domains.txt
```

#### fingerprinting

```
```


#### Screenshotting

GoWitness is the most accurate screenshot tool i've tried
```bash
gowitness file -f alive_domains.txt --delay 10
```

to view report you can serve the built in webserver 
```bash
gowitness report serve
```

can also export the results to a zip that puts all the screenshots in a `html` file for viewing.
```bash
gowitness report export -f <zip file name> 
```

If you just have screenshots, this bash script will put into a `html` file for easy viewing
```bash
#!/bin/bash

echo "<HTML><BODY><TABLE>" > web.html

counter=0
filenames=""
images=""

for file in *.png; do
    filenames+="<TD align='center'>$file:</TD>"
    images+="<TD align='center'><IMG SRC=\"$file\" width=300></TD>"
    ((counter++))

    if [ $counter -eq 3 ]; then
        echo "<TR>$filenames</TR><TR>$images</TR>" >> web.html
        counter=0
        filenames=""
        images=""
    fi
done

[ $counter -ne 0 ] && echo "<TR>$filenames</TR><TR>$images</TR>" >> web.html

echo "</TABLE></BODY></HTML>" >> web.html
```

#### crawling/spidering

**Katana**

my own testing, best command for full enum
```shell
katana -list urls.txt -js-crawl -form-extraction -jsluice -depth 5 -known-files all -automatic-form-fill -silent -headless -v -c 20 -o katana_output
```


---- This is from osmedeus -----

- standard mode is much faster than `-headless` but if the application  utilizes DOM manipulation and/or asynchronous events it'll be missed
- `-jc` turns on JavaScript parsing/crawling
-  `-kf` finds and crawls specified known files(off by default)

```bash
katana -list urls.txt -silent -headless -depth 5 -jc -kf robotstxt,sitemapxml>> url_links.txt
```


#####  Saving HTTP responses

`defaultUA` = "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4581.0 Safari/537.36"

on links found with katana
```bash
ExecCmd("cat {{linkFile}} | {{Binaries}}/httpx -silent -nf -no-color -t {{httpThreads}} -H '{{defaultUA}}' --store-response-dir {{httpResponse}} ")
```

on alive domains
```bash
ExecCmd("cat {{httpFile}} | {{Binaries}}/httpx -silent -nf -no-color -t {{httpThreads}} -H '{{defaultUA}}' --store-response-dir {{httpResponse}} ")
```


###### using Trufflehog on responses

```bash
ExecCmd("trufflehog --concurrency={{trufflehogThreads}} filesystem {{httpResponse}} > {{Output}}/linkfinding/trufflehog-{{Workspace}}-output.txt 2>&1")
```

#### subdomain takeover scan

`httpfile` is alive domains after httpx
`domainfile` is all subdomains before httpx

```bash     
"{{Binaries}}/nuclei -no-color -silent -c {{stoThreads}} -t ~/nuclei-templates/dns -l {{domainFile}} | grep -v '[info]' | tee -a {{Output}}/sto/sto-{{Workspace}}-dns.txt"
```

```bash
"{{Binaries}}/nuclei -no-color -silent -c {{stoThreads}} -t ~/nuclei-templates/takeovers -l {{httpFile}} | tee -a {{Output}}/sto/sto-{{Workspace}}-content.txt"
```

from reconFTW

```bash
cat subdomains.txt .tmp/webs_all.txt 2>/dev/null | nuclei -silent -nh -tags takeover -severity info,low,medium,high,critical -retries 3 -rl $NUCLEI_RATELIMIT -t ${NUCLEI_TEMPLATES_PATH} -o subtakeover.txt
```


## Vuln scanning

jaeles


```bash
jaeles scan -c 50 -s <signature> -U <list_urls> -p 'dest=xxx.burpcollaborator.net'
jaeles scan -c 50 -s '~/.jaeles/base-signatures/products/.*' -U <list_urls> -o output_file
jaeles scan -c 50 -s '~/.jaeles/base-signatures/sensitive/.*' -U <list_urls> -o output_file_sensitive
```

nuclei


#### #### semscan

will analyze code and fit vulns

the output cant be in json or it wont read correctly it seems.

ways i got to work:
- use `httpx -sr` to save responses to files
- in burpsuite -> highlight all request in logger -> right click, save to file -> uncheck the encode base64
- with logger++ -> export as CSV (configure first and make sure to uncheck the base64 request and responses)
- with logger++ setup to autosave CSV

from directory that you saved all responses
```bash
semgrep scan --max-target-bytes=100.0GB --pro --no-git-ignore -o semgrep.txt --config=r/all .
```

example output on a Hackthebox (found api key)
- trufflehog didnt
```bash
semgrep scan --max-target-bytes=100.0GB --config=r/all .
               
               
┌─────────────┐
│ Scan Status │
└─────────────┘
  Scanning 5 files (only git-tracked) with 2299 Code rules:
            
  CODE RULES
  Scanning 10 files with 236 <multilang> rules.
                    
  SUPPLY CHAIN RULES
                                                                       
  💎 Sign in with `semgrep login` and run               
     `semgrep ci` to find dependency vulnerabilities and
     advanced cross-file findings.                                     
                                                                       
          
  PROGRESS
   
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:07                                                                                                                        
Warning: 1 timeout error(s) in burp_logger when running the following rules: [generic.secrets.security.detected-
username-and-password-in-uri.detected-username-and-password-in-uri]
Warning: 1 timeout error(s) in LoggerPlusPlus.csv when running the following rules: [generic.secrets.security.detected-
username-and-password-in-uri.detected-username-and-password-in-uri]
Warning: 19 rule(s) were skipped because they require Pro (try `--pro`), for example:
apex.lang.security.ncino.dml.apexcsrfconstructor.apex-csrf-constructor
                   
                   
┌─────────────────┐
│ 5 Code Findings │
└─────────────────┘
                                     
    LoggerPlusPlus.csv
     ❱ generic.secrets.gitleaks.generic-api-key.generic-api-key
          A gitleaks generic-api-key was detected which attempts to identify hard-coded credentials.  It is   
          not recommended to store credentials in source-code, as this risks secrets being leaked  and used by
          either an internal or external malicious adversary. It is recommended to use  environment variables 
          to securely provide credentials or retrieve credentials from a  secure vault or HSM (Hardware       
          Security Module). This rule can introduce a lot of false positives,  it is not recommended to be    
          used in PR comments.                                                                                
          Details: https://sg.run/1KZv                                                                        
                                                                                                              
           13┆ Sec-WebSocket-Key: uMHJaZAlpFBJJ8vEsvktGQ==",,0,Fri Aug 02 14:42:26 PDT 2024,0,Proxy,null  
               ,true,https://alive.github.com/_sockets/u/101487630/ws?session=eyJ2IjoiVjMiLCJ1IjoxMDE0ODc2
               MzAsInMiOjEzNzY0Njg1MTcsImMiOjE4MzIwODY0MDQsInQiOjE3MjI2MzQyMDB9--2d02c7b5462887e4a644ea2f6
               e1734009ec23625745e86a8af8b9fe1121de859&shared=true&p=330255398_1722632644.2,GET,/_sockets/
               u/101487630/ws,session=eyJ2IjoiVjMiLCJ1IjoxMDE0ODc2MzAsInMiOjEzNzY0Njg1MTcsImMiOjE4MzIwODY0
               MDQsInQiOjE3MjI2MzQyMDB9--2d02c7b5462887e4a644ea2f6e1734009ec23625745e86a8af8b9fe1121de859&
               shared=true&p=330255398_1722632644.2,/_sockets/u/101487630/ws?session=eyJ2IjoiVjMiLCJ1IjoxM
               DE0ODc2MzAsInMiOjEzNzY0Njg1MTcsImMiOjE4MzIwODY0MDQsInQiOjE3MjI2MzQyMDB9--2d02c7b5462887e4a6
               44ea2f6e1734009ec23625745e86a8af8b9fe1121de859&shared=true&p=330255398_1722632644.2,https,t
               rue,No,alive.github.com,https://alive.github.com,443,,alive.github.com,,,true,true,false,tr
               ue,_octo=GH1.1.474214466.1715200489; preferred_color_mode=dark; tz=America%2FLos_Angeles; c
               olor_mode=%7B%22color_mode%22%3A%22auto%22%2C%22light_theme%22%3A%7B%22name%22%3A%22light%2
               2%2C%22color_mode%22%3A%22light%22%7D%2C%22dark_theme%22%3A%7B%22name%22%3A%22dark%22%2C%22
               color_mode%22%3A%22dark%22%7D%7D; logged_in=yes; dotcom_user=Cydriin;,3,"[session, shared, 
               p]",https://github.com,"connection: Upgrade                                                
            ⋮┆----------------------------------------
          149┆ <script defer src=""https://cdn.jsdelivr.net/ghost/sodo-search@~1.1/umd/sodo-     
               search.min.js"" data-key=""37395e9e872be56438c83aaca6"" data-                     
               styles=""https://cdn.jsdelivr.net/ghost/sodo-search@~1.1/umd/main.css"" data-sodo-
               search=""http://ghost.htb/"" crossorigin=""anonymous""></script>                  
            ⋮┆----------------------------------------
          403┆ <script defer src=""https://cdn.jsdelivr.net/ghost/sodo-search@~1.1/umd/sodo-     
               search.min.js"" data-key=""37395e9e872be56438c83aaca6"" data-                     
               styles=""https://cdn.jsdelivr.net/ghost/sodo-search@~1.1/umd/main.css"" data-sodo-
               search=""http://ghost.htb/"" crossorigin=""anonymous""></script>                  
            ⋮┆----------------------------------------
          793┆ <script defer src=""https://cdn.jsdelivr.net/ghost/sodo-search@~1.1/umd/sodo-     
               search.min.js"" data-key=""37395e9e872be56438c83aaca6"" data-                     
               styles=""https://cdn.jsdelivr.net/ghost/sodo-search@~1.1/umd/main.css"" data-sodo-
               search=""http://ghost.htb/"" crossorigin=""anonymous""></script>                  
            ⋮┆----------------------------------------
          1102┆ <script defer src=""https://cdn.jsdelivr.net/ghost/sodo-search@~1.1/umd/sodo-    
               search.min.js"" data-key=""37395e9e872be56438c83aaca6"" data-                     
               styles=""https://cdn.jsdelivr.net/ghost/sodo-search@~1.1/umd/main.css"" data-sodo-
               search=""http://ghost.htb/"" crossorigin=""anonymous""></script>                  

                
                
┌──────────────┐
│ Scan Summary │
└──────────────┘
Some files were skipped or only partially analyzed.
  Partially scanned: 3 files only partially analyzed due to parsing or internal Semgrep errors

Ran 236 rules on 5 files: 5 findings.
💎 Missed out on 1051 pro rules since you aren't logged in!
⚡ Supercharge Semgrep OSS when you create a free account at https://sg.run/rules.

⏫  A new version of Semgrep is available. See https://semgrep.dev/docs/upgrading
```

