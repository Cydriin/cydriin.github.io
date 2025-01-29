
Video by James Kettle that goes in depth for how to use and features
- https://www.youtube.com/watch?v=vCpIAsxESFY
- Article - https://portswigger.net/research/turbo-intruder-embracing-the-billion-request-attack

# Usage

1. Go to a request in burp -> highlight the portion to `FUZZ` or select `FUZZ` points with `%s`
2. change the wordlist to use based on what your trying to `FUZZ`
3. play around with these values based on what the target can handle
	1. also if the target support HTTP2 use that engine because it will be faster
		1. add the keyword argument `engine=Engine.HTTP2`
```python
concurrentConnections=20,
requestsPerConnection=200,
pipeline=True
```

From Kettle
> pick the fastest request engine. Typically Engine.HTTP2 is the fastest if it works, followed by a well-tuned Engine.THREADED, followed by Engine.BURP2 then Engine.BURP. 
> 	
> Next, tune the engine settings. If you ended up with either of the Burp engines, this just means tuning 'concurrentConnections' which is just the number of threads Turbo Intruder uses to send requests. If you're using the first two engines, you'll want to tune the pipeline, requestsPerConnection, and concurrentConnections arguments, probably in that order. Your goal should be to find values that maximise the RPS (Requests Per Second) value, while keep the Retries counter close to 0. The optimum values here are highly server dependent, but the max speed I've achieved to a remote server so far is 30,000 RPS.

if you need yet more speed, take advantage of Turbo Intruder's command-line operation by renting a server as close to the target as possible.

#### Multiple parameters

You can easily use multiple parameters simply by [passing them in as a list](https://github.com/PortSwigger/turbo-intruder/blob/master/resources/examples/multipleParameters.py).

#### Debugging problems

If the 'failed' counter in the stats panel starts rapidly increasing, that's a good sign that something is amiss. It could be a problem with your script, the target website, or Turbo Intruder's network stack. If you're interested in helping make the tool better, I've made a [debug script](https://github.com/PortSwigger/turbo-intruder/blob/master/resources/examples/debug.py) to help you identify and potentially report the source of the problem.

## Long-running attacks

Turbo Intruder can comfortably perform attacks requiring millions of requests, provided you follow two key principles.

First, don't save responses into the results table unless necessary. Every response you place in the table nibbles a bit of your RAM, so it's best to use the decorator system to filter out the junk.

The second point is a general programming principle: conserve memory by streaming data instead of buffering it

For example, don't pre-load wordlists into memory like this:
```python
words = open('/usr/share/dict/words').readlines()
for word in words:
    engine.queue(target.req, word.rstrip())
```

instead, process them line by line:
```python
for word in open('/usr/share/dict/words'):
    engine.queue(target.req, word.rstrip())
```

#### Built-in wordlists

Turbo Intruder has two built-in wordlists - one for launching long-running bruteforce attacks, and one containing all words observed in in-scope proxy traffic. The latter wordlist can lead to some quite interesting findings that would normally require manual testing to identify. You can see how to use these in [specialWordlists.py](https://github.com/PortSwigger/turbo-intruder/tree/master/resources/examples/specialWordlists.py)

#### Command line usage

From time to time, you might find you want to run Turbo Intruder from a server. To support headless use it can be launched directly from the jar, without Burp. 

You'll probably find it easiest to develop your script inside Burp as usual, then save and launch it on the server:
```shell
java -jar turbo.jar <scriptFile> <baseRequestFile> <endpoint> <baseInput>
```

Example:  
```shell
java -jar turbo.jar resources/examples/basic.py resources/examples/request.txt https://example.net:443 foo
```

The command line support is pretty basic - if you try to use this exclusively you'll probably have a bad time. Also, it doesn't support automatic interesting response detection as this relies on various Burp methods.


# Turbo Intruder Decorators

## Response Decorators

Turbo Intruder provides the researcher with the power of the entire python language to describe how to issue HTTP Requests and how to handle HTTP responses. However, because the language is very flexible there are certain patterns of code that are often used for various scenarios of response handling. It is a common practice for a user to potentially want to match or filter responses based on various criteria. For example, a user may only want to handle a response based on the response content matching a specific regex or HTTP Status code. Conversely, a user may want to filter out responses based on a specific regex or status code. Each one of these actions requires writing and pasting various `handleResponse` function implementations to meet these desired goals. Tools like `ffuf` and `dirsearch` have easy ways to enable these matchers and filters, Turbo Intruder unfortunately does not and it is left to the user to write these implementations. With this update Turbo Intruder shall contain matcher and filter implementations built-in to the plugin to provide all users the ability to easily add/remove complex matcher and filter logic to their `handleResponse` implementations.

### Examples

Let’s say we want to view only responses with status 200 and 204, a user may write an implementation which looks like this:
```python
@MatchStatus(200,204)
def handleResponse(req, interesting):
    table.add(req)
```

Conversely, they may want to view all responses with all other status except for 200 and 204
```python
@FilterStatus(200,204)
def handleResponse(req, interesting):
    table.add(req)
```

Python decorators can stack on top of each other and they get evaluated from top to bottom. Lets go ahead and match all responses status 200 and 204 but only those responses between 100 to 1000 bytes
```python
@MatchStatus(200,204)
@MatchSizeRange(100,1000)
def handleResponse(req, interesting):
    table.add(req)
```

Maybe we want to use regular expressions to help out with some cookie matching
```python
@MatchRegex(r".*Set-Cookie.*")
@MatchRegex(r".*SECRETCOOKIENAME.*")
def handleResponse(req, interesting):
    table.add(req)
```

Or perhaps we want to filter out pesky 200 Not Founds
```python
@MatchStatus(200)
@FilterRegex(r".*Not Found.*")
def handleResponse(req, interesting):
    table.add(req)
```

### Supported Response Decorators

|Decorator|Description|
|---|---|
|@MatchStatus(StatusCode, ...)|Matches responses with 1 or more specified status code(s)|
|@FilterStatus(StatusCode, ...)|Filters responses with 1 or more specified status code(s)|
|@MatchSize(RawSize, ...)|Matches responses with 1 or more specified response size(s)|
|@FilterSize(RawSize, ...)|Filters responses with 1 or more specified response size(s)|
|@MatchSizeRange(min, max)|Matches responses whose sizes fall between min and max (inclusive)|
|@FilterSizeRange(min, max)|Filters responses whose sizes fall between min and max (inclusive)|
|@MatchWordCount(WordCount, ...)|Matches responses with 1 or more specified response word count(s)|
|@FilterWordCount(WordCount, ...)|Filters responses with 1 or more specified response word count(s)|
|@MatchWordCountRange(min, max)|Matches responses whose word count fall between min and max (inclusive)|
|@FilterWordCountRange(min, max)|Filters responses whose word count fall between min and max (inclusive)|
|@MatchLineCount(LineCount, ...)|Matches responses with 1 or more specified response line count(s)|
|@FilterLineCount(LineCount, ...)|Filters responses with 1 or more specified response line count(s)|
|@MatchLineCountRange(min, max)|Matches responses whose line count fall between min and max (inclusive)|
|@FilterLineCountRange(min, max)|Filters responses whose line count fall between min and max (inclusive)|
|@MatchRegex(expression)|Matches responses which match the specified regex (case insensitive)|
|@FilterRegex(expression)|Filters responses which match the specified regex (case insensitive)|
|@UniqueSize(instances=1)|Only allows through N instances of responses with a given status/size|
|@UniqueWordCount(instances=1)|Only allows through N instances of responses with a given status/word count|
|@UniqueLineCount(instances=1)|Only allows through N instances of responses with a given status/line count|

### Unique Decorators

Glancing at the table most of the decorators probably seem straight forward. The only decorators that may require a little more explanation are the ones that start with `Unique`. These decorators are useful for fuzzing campaigns against an API or HTTP headers in which the responses stay for the most part static. Unique decorators reduce the signal/noise ratio by keeping a history of keys and only allowing N instances (N=1 by default) of a key to be processed. A key is made up of a status/size, status/word count, and status/line count pair. So for example if we consider the `@UniqueSize(2)` decorator and 1000 responses were received, if 250 of those responses had a status 200 with a size of 270 bytes only the first 2 of the 250 responses would be processed into the `handleResponse` function. If one of the 1000 responses had a status of 200 with a size of 271 that would register into a separate unique key and be processed by `handleResponse`. If you perform fuzzing iterations on JSON structure to post to an endpoint you may use `@UniqueSize()` to catch all unique error messages produced by the endpoint and the decorator will throw away responses associated with duplicate keys.