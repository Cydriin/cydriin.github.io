
Unofficial Documentation for the settings: https://github.com/nikitastupin/param-miner-doc
PortSwigger guide for checking hidden inputs: https://portswigger.net/burp/documentation/desktop/testing-workflow/analyzing/hidden-inputs

- FUZZ parameters in HTTP requests
- Makes it easy to find potential vectors for a web cache poisoning attack. It's capable of guessing up to 65,000 parameter names per request
- can do so much more

From James Kettles research article: [Listen to the whispers: web timing attacks that actually work](https://portswigger.net/research/listen-to-the-whispers-web-timing-attacks-that-actually-work#front-end-impersonation)

Misconfigured proxies offer an elegant alternative way to bypass header-overwriting defenses and perform front-end impersonation attacks. To try this out:
- Use Param Miner's 'Detect scoped SSRF' scan to detect a reverse proxy
- Run 'Exploit scoped SSRF' to find alternative routes to internal systems
- On each alternate route, run 'Guess headers' to find useful headers

Applying this successfully requires a robust mental visualization of what's happening behind the scenes. To help out, I've made a little CTF at [listentothewhispers.net](https://listentothewhispers.net/)






