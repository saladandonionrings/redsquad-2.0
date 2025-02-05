# Overview
>Web Cache Deception allows an attacker to trick a web cache into storing sensitive, dynamic content. It is caused by discrepancies between how the cache server and origin server handle requests.

Here is how it is done (usually) :
1. The victim's browser requests the resource `http://www.example.com/home.php/non-existent.css`
2. The requested resource is searched for in the cache server, but it's not found (resource not in cache).
3. The request is then forwarded to the main server.
4. The main server returns the content of `http://www.example.com/home.php`, most probably with HTTP caching headers that instruct not to cache this page.
5. The response passes through the cache server.
6. The cache server identifies that the file has a CSS extension.
7. Under the cache directory, the cache server creates a directory named home.php and caches the imposter "CSS" file (non-existent.css) inside it.
8. When the attacker requests `http://www.example.com/home.php/non-existent.css`, the request is sent to the cache server, and the cache server returns the cached file with the victim's sensitive `home.php` data.

![[Pasted image 20241024171402.png]]


## Differences between cache deception and poisoning : 
- Web cache poisoning manipulates cache keys to inject malicious content into a cached response, which is then served to other users.
- Web cache deception exploits cache rules to trick the cache into storing sensitive or private content, which the attacker can then access.

# Exploit
## Overview
1. Identify a target endpoint that returns a dynamic response containing sensitive information. Review responses in Burp, as some sensitive information may not be visible on the rendered page. Focus on endpoints that support the `GET`, `HEAD`, or `OPTIONS` methods as requests that alter the origin server's state are generally not cached.
2. Identify a discrepancy in how the cache and origin server parse the URL path. This could be a discrepancy in how they:
    - Map URLs to resources.
    - Process delimiter characters.
    - Normalize paths.
3. Craft a malicious URL that uses the discrepancy to trick the cache into storing a dynamic response. When the victim accesses the URL, their response is stored in the cache. Using Burp, you can then send a request to the same URL to fetch the cached response containing the victim's data. Avoid doing this directly in the browser as some applications redirect users without a session or invalidate local data, which could hide a vulnerability.

**Example**
- Real endpoint : `/my-account`
- Going to `/my-account/abc` works too
- Add a static extension to the URL : `/my-account/abc.js`
- Send the request
	- Check for headers like : `X-Cache`
		- `Cache-Control: max-age` : Meaning it is stored for xx seconds