
# Recon

#### subfinder

#### shuffledns



#### alterx



#### dnsx

```shell
# list active hostnames
cat subdomains.txt | dnsx -silent

# Extract subdomains from given network range using `PTR` query
echo 173.0.84.0/24 | dnsx -silent -resp-only -ptr

# can list any record or combination (`A`, `cname`,`MX`, `soa`, `ns`, `txt`, `axfr`, `any`, `srv`, `aaaa` )
cat subdomains.txt | dnsx -silent -a -cname -resp-only

# bruteforce subdomains
dnsx -silent -d domains.txt -w dns_worldlist.txt

# brute force tlds
dnsx -d gymshark.FUZZ -w tld.txt -resp

# using with alterx
cat subdomains.txt | alterx -silent -enrich | dnsx -silent
```

#### tlsx

can get way more hosts based off of the TLS certs
```shell
# get host
cat {{ips or urls}} | tlsx -silent -dns

# can pipe to dnsx
cat {{ips or urls}} | tlsx -silent -dns | dnsx -silent

# Get cert information and TLS misconfigorations and all supported cipher versions
cat {{ips or urls}} | tlsx -silent -tls-version -cipher -expired -self-signed -mismatched -revoked -untrusted -ve
```

- can use the [testssl](https://github.com/drwetter/testssl.sh) tool to find further vulnerabilities

#### httpx

```shell
# test if has running `http` service
httpx -silent -retries 2

# fingerprint basic
httpx -silent -fr -sc -title -td -retries 2

# fingerprint more
httpx -silent -sc -title -cl -td -websocket -cdn -cname -fr -retries 2

# screenshots
httpx -silent -screenshot -fr -retries 2
httpx -silent -screenshot -path fuzz_path.txt -u https://example.com

# store the respones
httpx -silent -sr -fr

# testing all http methods
httpx -silent -x all -method -sc -fr -cl -title -fc 400,501

# option for efficient File/Path Bruteforcing
httpx -l urls.txt -path /v1/api -sc

# Body preview shows first N characters of response. And strip html tags in response
httpx -silent -bp 200 -strip
```

Other useful options
```shell
-p, -ports string[]        ports to probe (nmap syntax: eg http:1,2-10,11,https:80)
-H, -header string[]          custom http headers to send with request
-body string                  post body to include in http request
-fep, -filter-error-page            filter response with ML based error page detection
-bp, -body-preview            display first N characters of response body (default 100)
-r, -resolvers string[]       list of custom resolver (file or comma separated)
-pa, -probe-all-ips           probe all the ips associated with same host
-probe                        display probe status
-cl, -content-length          display response content-length
-ct, -content-type            display response content-type
-hash string                  display response body hash (supported: md5,mmh3,simhash,sha1,sha256,sha512)
-path string                  path or list of paths to probe (comma-separated, file)


EXTRACTOR:
-er, -extract-regex string[]   display response content with matched regex
-ep, -extract-preset string[]  display response content matched by a pre-defined regex (ipv4,mail,url)
```

Filters/Matchers
```shell
   -fc, -filter-code string            filter response with specified status code (-fc 403,401)
   -fep, -filter-error-page            filter response with ML based error page detection
   -fl, -filter-length string          filter response with specified content length (-fl 23,33)
   -flc, -filter-line-count string     filter response body with specified line count (-flc 423,532)
   -fwc, -filter-word-count string     filter response body with specified word count (-fwc 423,532)
   -ffc, -filter-favicon string[]      filter response with specified favicon hash (-ffc 1494302000)
   -fs, -filter-string string          filter response with specified string (-fs admin)
   -fe, -filter-regex string           filter response with specified regex (-fe admin)
   -fcdn, -filter-cdn string[]         filter host with specified cdn provider (cloudfront, fastly, google)
   -frt, -filter-response-time string  filter response with specified response time in seconds (-frt '> 1')
   -fdc, -filter-condition string      filter response with dsl expression condition
   -strip                              strips all tags in response. supported formats: html,xml (default html)

   -mc, -match-code string            match response with specified status code (-mc 200,302)
   -ml, -match-length string          match response with specified content length (-ml 100,102)
   -mlc, -match-line-count string     match response body with specified line count (-mlc 423,532)
   -mwc, -match-word-count string     match response body with specified word count (-mwc 43,55)
   -mfc, -match-favicon string[]      match response with specified favicon hash (-mfc 1494302000)
   -ms, -match-string string          match response with specified string (-ms admin)
   -mr, -match-regex string           match response with specified regex (-mr admin)
   -mcdn, -match-cdn string[]         match host with specified cdn provider (cloudfront, fastly, google)
   -mrt, -match-response-time string  match response with specified response time in seconds (-mrt '< 1')
   -mdc, -match-condition string      match response with dsl expression condition
```

upload the results to the Cloud UI Dashboard
```shell
export PDCP_TEAM_ID={{api_key}}
httpx -dashboard
```
#### katana

best command for full enum (ommit `-sr` if don't wanna save the responses)
```shell
katana -silent -js-crawl -form-extraction -jsluice -depth 7 -known-files all -automatic-form-fill -headless -c 20 -p 5 -sr -sf ufile,path,url,qurl,dir,key,value,kv,email -field-scope {change} -o katana_output
```

**adjusting scope**
```shell
-field-scope {rdn/fqdn/dn}
	rdn - crawling scoped to root domain name and all subdomains (e.g. *example.com) (default)
	fqdn - crawling scoped to given sub(domain) (e.g. www.example.com or api.example.com)
	dn - crawling scoped to domain name keyword (e.g. example)

-crawl-scope {string/regex} - For advanced scope control, comes with regex support
-crawl-scope {file} - For multiple in-scope rules, file input with multiline string/regex
-crawl-out-scope {string/regex} - For defining what not to crawl
-crawl-out-scope {file} - For defining multiple rules OOS rules
```

**Other Options**
```shell
-H, -headers {string[]/file}      custom header/cookie to include in request (can supply a headers file for multiple)
-em, -extension-match string[]    match output for given extension (eg, -em php,html,js)
-ho, -headless-options string[]   start headless chrome with additional options (e.g. -headless-options --disable-gpu,proxy-server=http://127.0.0.1:8080)
-parallelism, -p                  option to define number of target to process at same time from list input
-concurrency, -c                  option to control the number of urls per target to fetch at the same time
```










