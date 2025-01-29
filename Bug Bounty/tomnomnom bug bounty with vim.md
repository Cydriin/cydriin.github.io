Tom captured the output of HTTP requests (with meg) of subdomains he found on uber (using assetfinder and piping to httpx)
	- he recalled earlier of seeing `uberinternal` and its in scope, i think just for demo purposes hes doing that
	- he did command `grep -Hnri uberinternal * | vim -` 
		- specifies to search for `uberinternal` in all files in current directory and sub-directories, and pipes to vim
	- did `:%! sort -u`, `:%! grep -v Content-Security`, `:%! grep -vi csp`to filter results more

uses `gf` tool to search for patterns in the files
	- `gf base64 | vim -`
	- (in vim) `:%!awk -F ':' '{print $3}'`  using `awk` parses the results from `gf` so its just the base64 strings
	- `:%s/.$//` - removes the dot from the end of each line
	- `:%! sort -u` - removes duplicates
	- `:%!xargs -n1 -I{} sh -c 'echo {} | base64 -d'` will take each line (that's base64 encoded) and decode it back into vim

1 liner for git repos, takes all the git object files (even past history) and converts to a single text stream so you can `grep` thorough it
	-put what you want to `grep` for at the end
```bash
{ find .git/objects/pack/ -name "*.idx" | while read i; do git show-index < "$i" | awk '{print $2}'; done; find .git/objects/ -type f | grep -v '/pack/' | awk -F'/' '{print $(NF-1)$NF}'; } | while read o; do git cat-file -p $o; done | grep -Ea
```

`gf urls | vim -` extracts all the urls from our response files
	- `:%!sort -u`
	- `:%!unfurl -u paths` extracts all the unique paths from the urls
		- created a paths wordlist essentially
	- `%!unfurl -u keys` extracts all the unique query string parameters
		- can then use as a wordlist for  something like burp param miner
