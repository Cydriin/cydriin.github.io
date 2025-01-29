## Basic Symbols
`.` - Matches any single character except newline
`^` - Matches the start of a string
`$` - Matches the end of a string
`*` - Matches 0 or more repetitions of the preceding element
`+` - Matches 1 or more repetitions of the preceding element
`?` - Matches 0 or 1 repetition of the preceding element
`\` - Escapes a special character
`|` - Alternation (matches either the expression before or the expression after the `|`)

## Character Classes
`\d` - Matches any digit (equivalent to `[0-9]`)
`\D` - Matches any non-digit
`\w` - Matches any word character (alphanumeric & underscore)
`\W` - Matches any non-word character
`\s` - Matches any whitespace character (spaces, tabs, line breaks)
`\S` - Matches any non-whitespace character
`[abc]` - Matches any one of the characters a, b, or c
`[^abc]` - Matches any character except a, b, or c
`[a-z]` - Matches any character in the range a to z

## Quantifiers
`{n}` - Matches exactly n occurrences of the preceding element
`{n,}` - Matches n or more occurrences of the preceding element
`{n,m}` - Matches between n and m occurrences of the preceding element

## Anchors
`\b` - Word boundary
`\B` - Non-word boundary
`\A` - Start of the string
`\Z` - End of the string
`\z` - End of the string (similar to `\Z` but stricter)

## Groups and Lookarounds
`()` - Capturing group
`(?:)` - Non-capturing group
`(?=...)` - Positive lookahead (asserts that the specified pattern matches)
`(?!...)` - Negative lookahead (asserts that the specified pattern does not match)
`(?<=...)` - Positive lookbehind (asserts that the specified pattern precedes)
`(?<!...)` - Negative lookbehind (asserts that the specified pattern does not precede)

## Common Patterns

Matches a US Social Security number (###-##-####)
```regex
\d{3}-\d{2}-\d{4}
```

Matches an email address
```regex
[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}
```

Matches a US ZIP code
```regex
\b\d{5}(-\d{4})?\b
```

Matches a URL
```regex
(http|https):\/\/[^\s]+
```

## Example Expressions

Matches a US Social Security number at the start and end of the string
```regex
^\d{3}-\d{2}-\d{4}$
```

Matches an email address with 2 to 3 character top-level domain
```regex
^\w+@\w+\.\w{2,3}$
```

Matches an email address with a longer top-level domain
```regex
\b[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}\b
```

Matches a number with optional commas and decimal point
```regex
\b\d{1,3}(,\d{3})*(\.\d+)?\b
```

## More Examples

Matches a username (3-16 characters, lowercase, numbers, underscores, hyphens)
```regex
^[a-z0-9_-]{3,16}$
```

Matches a URL (HTTP, HTTPS, FTP)
```regex
(https?|ftp)://[^\s/$.?#].[^\s]*
```

Matches any whole word
```regex
\b\w+\b
```

Matches two letters followed by four digits, preceded by three digits
```regex
(?<=\d{3})[A-Z]{2}\d{4}
```

