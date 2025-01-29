# Project setup
- [ ] If testing a mobile application, use [this setup guide](https://portswigger.net/burp/documentation/desktop/mobile/config-ios-device)
- [ ] Configure Scope
- [ ] Turn on logging to disk from settings -> Project/Logging
- [ ] Logger++ -> Auto save to CSV ( configure what to log first and  uncheck the base64 request and responses)
- [ ] Settings -> Network -> Connections -> enable upstream proxy servers 
	- if proxying traffic through another proxy in addition to burp
- [ ] Settings -> Network - TLS -> Client TLS Certificates
	- if application requires authentication/use of client TLS certificates 
- [ ] Settings -> Proxy -> Proxy Listeners -> Support invisible proxying (also adjust other settings if needed)
	- if testing an atypical application or  non-browser-based HTTP clients
- [ ] Add recorded login sequence with Burp Suite Navigation Recorder
	- To capture -> use the chrome extension [Burp Suite Navigation Recorder](https://chromewebstore.google.com/detail/burp-suite-navigation-rec/anpapjclbjicacakeoggghfldppbkepg?hl=en-GB)  (use incognito mode whenever possible to avoid issues with stateful behavior) -> start recording -> browse to the website -> complete login sequence
	- To add -> from Dashboard, click **New Scan** -> Application login -> Use recorded Sequences -> New and paste the JSON data from the extension
	- if the application uses any of these (they are not compatible with two-factor authentication, character-select passwords, or CAPTCHA)
		- Single sign-on
		- Multi-step logins in which the username and password are not entered in the same form
		- Login forms that contain, for example, extra fields or checkboxes
	-  this can also be used to show steps for demoing a full  vulnerability workflow for reporting

# Content Discovery
- [ ] 