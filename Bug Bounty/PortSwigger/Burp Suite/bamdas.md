
### Bamdas used for proxy columns

Current preferred default layout. (Selections are custom columns)
![[Pasted image 20240807113837.png]]

**Reflected** - shows HTTP parameters and the values if the Value was reflected in the HTTP response body
```java
if (!requestResponse.hasResponse()) {
    return "";
}

HttpRequest request = requestResponse.request();
HttpMessage response = requestResponse.response();

List<ParsedHttpParameter> params = request.parameters();
String responseBody = response.bodyToString();

HashMap<String, String> reflected = new HashMap<>();

for (HttpParameter param : params) {
    if (responseBody.contains(param.value())) {
        reflected.put(param.name(), param.value());
    }
}

if (reflected.isEmpty()) {
    return "";
} else {
    StringBuilder result = new StringBuilder();
    for (Map.Entry<String, String> entry : reflected.entrySet()) {
        //result.append(entry.getKey()).append(": ").append(entry.getValue()).append(" | ");
        result.append("?").append(entry.getKey()).append("=").append(entry.getValue()).append(" | ");
    }
    if (result.length() > 0) {
        result.setLength(result.length() - 3);
    }
    return result.toString();
}
```

**Server** - Shows server response header
```java
if (!requestResponse.hasResponse()) {
return "";
}

var response = requestResponse.response();

return response.hasHeader("Server")
? response.headerValue("Server")
: "";
```

**Set-Cookie** - Shows response header of Set-Cookie
```java
if (!requestResponse.hasResponse()) {
    return "";
}
var response = requestResponse.response();
return response.hasHeader("Set-Cookie")
    ? response.headerValue("Set-Cookie")
    : "";
```