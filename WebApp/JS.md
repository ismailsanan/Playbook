

#### Tagged Templates

`Tagged template literals` offers you the opportunity to parse template literals in whatever way you want. It works by combining functions with `template literals`. There are two parts of a `tagged template literal`, the first one being the `tag function` and the second, the `template literal`.

```
const coolVariable = "Cool Value";
```

```
const anotherCoolVariable = "Another Cool Value";
```

```
randomTagFunctionName`${coolVariable} in a tagged template literal with ${anotherCoolVariable} just sitting there`
```

Now, the first parameter in the `tag function` is a variable containing an array of strings in the template literal:

```
function randomTagFunctionName(firstParameter) {
```

```
console.log(firstParameter);     // ["", " in a tagged template literal with ", " just sitting there"]
```

```
}
```



#### References

https://portswigger.net/research/the-seventh-way-to-call-a-javascript-function-without-parentheses

https://www.freecodecamp.org/news/a-quick-introduction-to-tagged-template-literals-2a07fd54bc1d/