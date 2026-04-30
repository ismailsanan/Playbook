tactic: Initial Access

**Authentication** is the process of verifying who a user is. Vulnerabilities arise when this process can be bypassed, brute-forced, or manipulated giving an attacker access to accounts they don't own.
  
```
User submits credentials → Server verifies → Issues session token / cookie
```


Three factors:

- **Something you know** — password, security question
- **Something you have** — OTP, hardware token
- **Something you are** — biometrics

## Injection Points

- Login forms (username, password)
- Password reset flows (token, email param)
- Account registration (duplicate usernames, email verification bypass)
- "Remember me" cookies
- HTTP headers (`X-Forwarded-For`, `X-Real-IP`) used for rate limiting
- MFA code submission endpoints

## Checklist

### Username Enumeration

- [ ]  Submit valid vs invalid usernames — compare **response time**, **status code**, **body length**, **error messages**
    - `"Invalid username"` vs `"Invalid password"` → username confirmed
    - Response takes longer for valid username (server checks password hash) → timing oracle
- [ ]  If rate limited — rotate IP using `X-Forwarded-For` header with Burp Intruder:

```
	X-Forwarded-For: §RANDOM_IP§
```

```
Set pitchfork attack: payload 1 = IP list, payload 2 = username/password list
```

- [ ]  Try HTTP header variations if `X-Forwarded-For` is blocked:

```
	X-Real-IP
	X-Client-IP
	X-Remote-IP
	True-Client-IP
```

### Password Brute Force

- [ ]  Check if account lockout exists — how many attempts before block?
- [ ]  Check if lockout resets after successful login — try: `valid_pass → wrong → wrong → valid_pass → repeat`
- [ ]  Check if rate limit is IP-based only → bypass with `X-Forwarded-For`
- [ ]  Check if rate limit is session-based → rotate session cookies

### Response Timing Attack

- [ ]  Send login request with a very long password for a valid username — does it take longer?
    - Server only computes bcrypt/hash if username exists → timing difference leaks valid usernames
- [ ]  Automate with Intruder using **Pitchfork**:
    - Position 1: `X-Forwarded-For` → random IPs
    - Position 2: `username` → wordlist
    - Position 3: `password` → fixed very long string
    - Sort results by **response time** — longer = valid username

### Password Reset Flaws

- [ ]  Check if reset token is short, predictable, or timestamp-based
- [ ]  Check if token expires — try using an old token
- [ ]  Check if token is validated server-side or just client-side
- [ ]  Try changing the `Host` header on the reset request to your server → does the reset link go to you?

```
	Host: evil.com
```

- [ ]  Check if the `username` or `email` param in the reset request can be tampered after token is issued

### MFA Bypass

- [ ]  After submitting valid credentials, directly navigate to authenticated pages — is MFA enforced?
- [ ]  Check if the MFA code endpoint has rate limiting — brute force 4–6 digit codes
- [ ]  Try reusing a previously valid MFA code
- [ ]  Check if MFA code is leaked in the response body or cookies

### "Remember Me" Cookie

- [ ]  Decode the cookie — is it predictable? (username+timestamp, base64 encoded, etc.)
- [ ]  Check if it's tied to server-side session or just validated client-side

---

## White Box Testing

**Vulnerable — timing difference leaks valid usernames:**

```java
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest request) {
    User user = userRepository.findByUsername(request.getUsername());

    if (user == null) {
        return ResponseEntity.status(401).body("Invalid username"); // ← returns immediately
    }

    // ← only reached for valid usernames — bcrypt takes ~300ms
    if (!passwordEncoder.matches(request.getPassword(), user.getPasswordHash())) {
        return ResponseEntity.status(401).body("Invalid password"); // ← different message + timing
    }

    return ResponseEntity.ok(generateSession(user));
}
```

**Vulnerable — IP rate limiting bypassable via header:**

```java
@PostMapping("/login")
public ResponseEntity<?> login(HttpServletRequest httpRequest,
                                @RequestBody LoginRequest request) {
    // ← trusts X-Forwarded-For header directly — attacker rotates IPs to bypass lockout
    String clientIp = httpRequest.getHeader("X-Forwarded-For");
    if (isRateLimited(clientIp)) {
        return ResponseEntity.status(429).body("Too many attempts");
    }
    // ...
}
```

**Vulnerable — predictable password reset token:**

```java
public String generateResetToken(User user) {
    // ← timestamp-based token — predictable and brute-forceable
    return Base64.encode(user.getEmail() + System.currentTimeMillis());
}
```

**Vulnerable — Host header not validated in reset email:**

```java
@PostMapping("/forgot-password")
public ResponseEntity<?> forgotPassword(HttpServletRequest request,
                                         @RequestParam String email) {
    String host = request.getHeader("Host"); // ← attacker-controlled
    String resetLink = "https://" + host + "/reset?token=" + generateToken(email);
    emailService.send(email, resetLink); // ← link goes to attacker's server
    return ResponseEntity.ok("Reset email sent");
}
```

**Secure:**

```java
// Use constant-time comparison to prevent timing attacks
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest request) {
    User user = userRepository.findByUsername(request.getUsername());
    String hash = (user != null) ? user.getPasswordHash() : DUMMY_HASH;

    // ← always runs bcrypt regardless of whether user exists — eliminates timing difference
    boolean valid = passwordEncoder.matches(request.getPassword(), hash);

    if (user == null || !valid) {
        return ResponseEntity.status(401).body("Invalid credentials"); // ← same message always
    }

    return ResponseEntity.ok(generateSession(user));
}

// Use cryptographically secure random token for password reset
public String generateResetToken() {
    byte[] bytes = new byte[32];
    new SecureRandom().nextBytes(bytes);
    return Base64.getUrlEncoder().withoutPadding().encodeToString(bytes);
}

// Never trust X-Forwarded-For alone — combine with session-based lockout
// Use a hardcoded base URL for reset links — never from request headers
@Value("${app.base-url}")
private String baseUrl;

String resetLink = baseUrl + "/reset?token=" + token;
```

---

## Resources

- [PortSwigger Authentication Labs](https://portswigger.net/web-security/authentication)
- [HackTricks Brute Force](https://book.hacktricks.xyz/pentesting-web/login-bypass/password-brute-forcing)
- [OWASP Authentication Cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)