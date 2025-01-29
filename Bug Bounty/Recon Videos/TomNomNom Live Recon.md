[Source](https://www.youtube.com/watch?v=SYExiynPEKM)

`assetfinder` - find subdomains

`anew `- like `tee -a` but it removes duplicates automatically and will output only the unique items (so you don't have to keep `sort -u`)
`httprobe -c 80 --prefer-https`

`fff` - saves the response of all the requests to a file (like curl?)
- runs on all the alive hosts (after finding subdomains and `httprobe`)
- saves each response into 2 separate files `.headers` & `.body`
- `fff -d 1 -S -o roots`
-  `-d 1` - delay of 1 ms
- `-S` - save all responses
- -`o roots`- saves all the responses in a directory called roots

`gf` - tool used for grepping for certain patterns (instead of typing the whole/complex pattern with `grep` every time)
- look for stuff that stands out
- looking for unique headers that aren't on all the servers
- Examples used:`gf debug-pages`, `gf firebase`, `gf http-auth`

`meg`

`urintersting` - `greps` out URLS that look "interstting". Like `admin` in the name `xml` , interesting parameters, etc..

- when Tom finds something interesting usually runs like `grep -Hrni <interesting thing>` in the `roots` directory (with all the responses body headers) to see how many times it shows up
	- `-H` - shows the filename
	- `-r` - recursive
	- `-n` prints the line number
	- `-i` case insensitive 
- also with `anew` and `wc -l` to see how many their are and how unique

`waybackurls` - gets all domains that the **waybackmachine** knows for a URL

- usually performs brute forcing directories on `404` & `403` responses

`inscope`  - filter in-scope subdomains & URLs from input















