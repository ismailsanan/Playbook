when is this useful
- when we have a dependency/ies in a specific value of a request where  

Useful Scenario

```
<soap:Header>
<Message GId="XXX" MId="c808784e-5f31-4f5c-9c85-c606eb6d04e2">
```

where each request requires a unique MessageID `MId` in this case i can't do an active scan using burp and it would make testing much more complex to change it manually 

Create a Custom in Custom Vertor

![[Screenshot from 2025-09-22 11-05-50.png]]

- Choose a Custom Tag of our choice
- choose a language of out choice
- start scripting


![[Pasted image 20250922110626.png]]

Note that we can use we need to use `output` as the actual output or  we can imagine it as the return of the script
and we can use `input` for inputs that are selected between the tags

![[Screenshot from 2025-09-22 11-15-32.png]]

Add the custom tag in the desired input
### Reference

inspired by:
- https://www.pmnh.site/post/howto-hackvertor-request-resigning/

