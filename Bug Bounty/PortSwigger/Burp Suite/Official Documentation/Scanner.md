
Scanner built in configurations - https://portswigger.net/burp/documentation/scanner/scan-configurations/burp-scanner-built-in-configs
## Audit phase indicators

Burp Scanner runs through the following phases when auditing content:

### Passive phases

Burp Scanner has two passive phases:

- Phase 1 - Identify passive issues.
- Phase 2 - Consolidate issues that exist at different locations in the application. Burp then reports on the issues.




### JavaScript phases

Burp Scanner has three JavaScript phases:

- Phase 1 - Analyze JavaScript to detect self-contained DOM-based issues.
- Phase 2 - Analyze reflection of input into JavaScript code to detect reflected DOM-based issues.
- Phase 3 - Analyze stored input in JavaScript code to detect stored DOM-based issues.