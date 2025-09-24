

an attacker can trick a web cache into storing sensitive , dynamic content that would cause a disagreements between how the cache server and origin server handles the request.

# scenario

- an attacker persuades a victim to visit a URL ,  inducing the browser into making requests for sensitive content.
- the cache misinterprets this into  request for a static resource and stores the response.
- the attacker can the  request the same url to access the cached response



# web cache system

its a system that sits between the origin server and the user. when a client requests a static resource the request is first directed to the cache. if the cache doesn't contain the resource "cache miss",  the request is then sent to the server  which processes and responds to the request. 

the response is then sent to the cache before being sent to the user.



When the cache receives an http request, it must decide whether there is a cache response that it can serve directly or whether it has to forward the request to  the origin server. 

the cache makes this decision by generating a cache key from elements of the HTTP request. typically  include url path and query param.

cache rules  often set up to store static resources,  dynamic contentent isnt usually cached  how does it work:
- for static file it could be that it detects file extensions `.css ` , `.js`
- other possibility would be through  directory  path `/static` , `/assets` 
- or file name `robots.txt`



# attack

1) identify endpoint that return a dynamic response containing sensitive information ,  Focus on `GET` , `HEAD` , `OPTIONS`
2)  identify  how the cache and origin server parse the URL PATH  - map URL to resource , Process delimiter char , normalize path 
3)  craf a malicious  URL to trick the cache into storing a dynamic response. when the victim accessses the URL, their response is stored in the cache 



### Cache buster


web cache exploit  to use it install the extension param miner then in the settings add dynamic  cachebuster 


### Detecting cached responses


its crucial that your able to identify cached responses  headers :
- X-Cache
-  `X-Cache: hit`
- `X-Cache: miss`
- `X-Cache: dynamic` -> response not  suitable for caching 
- `X-Cache: refresh` ->  content outdated and needs to be refreshed

# Exploiting static extension cache rules

common paths .css , .js  an attacker may be able to craft a request for a dynamic resource with a static extension that is ignored by the origin server viewed by the cache 



# Path mapping discrepancies


URL path mapping  is a process of associating URL path with resources on a server , such as files , scripts or command executions.

2 methods : 

traditional URL: 
`/path/in/filesystem/` represents the directory path in the server's file system.
`resource.html` is the specific file being accessed.


REST-style URLs:
`/path/resource/` is an endpoint representing a resource.
`param1` and `param2` are path parameters used by the server to process the request.


`http://example.com/user/123/profile/wcd.css`

- An origin server using REST-style URL mapping may interpret this as a request for the `/user/123/profile` endpoint and returns the profile information for user `123`, ignoring `wcd.css` as a non-significant parameter.

cache that uses traditional URL mapping may view this as a request for a file named `wcd.css` located in the `/profile` directory under `/user/123`. It interprets the URL path as `/user/123/profile/wcd.css`. If the cache is configured to store responses for requests where the path ends in `.css`, it would cache and serve the profile information as if it were a CSS file.


## Delimiter discrepancies 

`?` generally used to separate the URL path from the query string. 
Discrepancies in how the cache and origin server use characters and strings as delimiters can result in web cache deception vulnerabilities exmaple /profile;foo.css


he Java Spring framework uses the `;` character to add parameters known as matrix variables
hence ; as a delimeter in java spring  trunactes /profile and returns profile info 

other  dont  use ; as delimiter 

Therefore, a cache that doesn't use Java Spring is likely to interpret `;` and everything after it as part of the path


onsider these requests to an origin server running the Ruby on Rails framework, which uses `.` as a delimiter to specify the response format

`/profile` - This request is processed by the default HTML formatter, which returns the user profile information.

`/profile.css` - This request is recognized as a CSS extension. There isn't a CSS formatter, so the request isn't accepted and an error is returned.

`/profile.ico` - This request uses the `.ico` extension, which isn't recognized by Ruby on Rails. The default HTML formatter handles the request and returns the user profile information. In this situation, if the cache is configured to store responses for requests ending in `.ico`, it would cache and serve the profile information as if it were a static file.

Encoded characters may also sometimes be used as delimiters. For example, consider the request `/profile%00foo.js`:

The OpenLiteSpeed server uses the encoded null `%00` character as a delimiter. other frameworks respond with an error if `%00` is in the URL. However, if the cache uses Akamai or Fastly, it would interpret `%00` and everything after it as the path.


### Exploiting delimiter discrepancies

we may use delimiter to add static extension to the path but not the origin server.  but if we want to detect chars that are used as delimeter we need to test them on the server not cache 

- modify /settings/users/list -> /settings/users/listaaa -> ... /list;aaaa
- if the server is identical to the base response this indicate that ; is delimeter  if not then no

- once this is set we can start testing it on the cache. add a static extension to the end of the path if the response is cached this indicates that the cache does use the delimiter  and interprets the full URL , that there is  a cache rule to store responses for requests ending in .js
- make sure to test all .css , .ico , .exe 

### Delimiter decoding discrepancies

some special chars are usually encoded for a special meaning , some parsers decode certain chars before processing the URL. If a delimiter character is decoded, it may then be treated as a delimiter, truncating the URL path. example ``profile%23wcd.css`, which uses the URL-encoded `#` character:`  


Paths to check : `URL path prefixes, like `/static`, `/assets`, `/scripts`, or `/images``

### Detecting normalization by the cache server

MOST IMPORTANT understand the behaviour of the cache and server  since


look for potential static directories, look for requests with common static directories prefixes and cached responses.   
example  ``/aaa/..%2fassets/js/stockCheck.js`` -> - If the response is no longer cached, this indicates that the cache isn't normalizing the path before mapping it to the endpoint. It shows that there is a cache rule based on the `/assets` prefix.
- If the response is still cached, this may indicate that the cache has normalized the path to `/assets/js/stockCheck.js`

expoiting them  we look for this `/<static-directory-prefix>/..%2f<dynamic-path>`


or basically `/<dynamic-path>%2f%2e%2e%2f<static-directory-prefix>` --> /../

consider the payload `/profile;%2f%2e%2e%2fstatic`. The origin server uses `;` as a delimiter:

- The cache interprets the path as: `/static`
- The origin server interprets the path as: `/profie`

# LAB
# Exploiting path mapping for web cache deception


send a request to `/my-account/abc.js`   notice that the response is contain  a response header of  cache miss and cache age  meaning that particular request is cached  with the api key loaded in the page 

now  
```js
<script>document.location="https://YOUR-LAB-ID.web-security-academy.net/my-account/wcd.js"</script>
```

then visit the page 


# Exploiting path delimiters for web cache deception

; is used as a delimiters 

what happened is that when adding some chars account/abc -> not found 
but the static files are cached something.js 
account;something.js its cached  

``` js

<script>document.location="https://0a8e00b703bb629181859e9800ac003d.web-security-academy.net/my-account;abc.js"</script>
```



# Exploiting origin server normalization for web cache deception

oi notices that the static files are caches and my account not so we can do a path traversal tricking the cache to cache this  i noticed that query strings ?  have a longer age while without its age 0

```js

<script>document.location="https://0acd00cd038c753a8190ca60004400e3.web-security-academy.net/resources/..%2fmy-account?wcd" </script>
```



# Exploiting cache server normalization for web cache deception
send it to the intruder with the list of delimieters 

`/my-account§§abc`  -> 200 means its the one  This indicates that they're used by the origin server as path delimiters

```js
<script>document.location="https://0aee00d0043a9dfa810016c400b80033.web-security-academy.net/my-account%23%2f%2e%2e%2fresources?wcd"</script>
```

# list of delimiters

```
``! " # $ % & ' ( ) * + , - . / : ; < = > ? @ [ \ ] ^ _ ` { | } ~ %21 %22 %23 %24 %25 %26 %27 %28 %29 %2A %2B %2C %2D %2E %2F %3A %3B %3C %3D %3E %3F %40 %5B %5C %5D %5E %5F %60 %7B %7C %7D %7E``
```
