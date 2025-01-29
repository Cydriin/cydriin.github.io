#### semscan

will analyze code and fit vulns

the output cant be in json or it wont read correctly it seems.

ways i got to work:
- use `httpx -sr` to save responses to files
- in burpsuite -> highlight all request in logger -> right click, save to file -> uncheck the encode base64

from directory that you saved all responses
```bash
semgrep scan --config=r/all .
```

#### grep

The tried and true method of exploring data responses is through grep, grep for things you may find interesting, you can find keywords to grep for by using a tool such as tok along with sort.

To recursively grep through your data, the following grep command will show line numbers and filenames as well as be case insensitive.

```bash
grep -Hrni ‘set-cookie’ —-color=always | batcat
```

#### qsreplace + ffuf

Using qsreplace and ffuf, we can look for vulnerabilities such as path traversal or SQL Injection vulnerabilities. Simply set qsreplace to your payload of choice and then set the regex match `-mr ‘REGEX’` to a relevant regex

```bash
function fuzz() {
	payload=“$1”
	regex=“$2”

	cat crawl.txt | grep “?” | qsreplace “$payload” | ffuf -u “FUZZ” -w - -mr “$REGEX”
}

fuzz “../../../../../../../etc/passwd” “^root:”
```
or
```bash
cat crawl.txt | grep "?" | qsreplace ../../../../../../../etc/passwd | ffuf -u 'FUZZ' -w - -mr '^root:'
```

#### gf

Gf allows you to define your grep patterns in JSON files and refer to them with an alias. A list of example rules can be found here: https://github.com/tomnomnom/gf/tree/master/examples

Using gf, search for AWS keys inside responses:
```bash
gf aws-keys
```

#### tok

Tok is a hugely powerful tokenizer tool, this means that it extracts words from given files and removes special characters and spaces.

```bash
find -type f | tok | sort | uniq -c | sort -rn | head -n 40
```

We extract all the words from the HTTP responses and sort them in ascending order of frequency of occurrence.  These words can be added to a wordlist for further enumeration, or grepped for to identify context.
![[Pasted image 20231120155826.png | 500]]

