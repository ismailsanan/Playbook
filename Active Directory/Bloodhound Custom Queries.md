


> Update neo4j queries
```sh
# access the neo4j docker 
docker exec -it <container iD> cypher-shell -u <user> -p <pass>

 MATCH (c1:Computer{samaccountname:"AREA"})-[r:HasSession]->(u:User{samaccountname:"pepe"}) delete r;

```
## Practical Custom Query Examples

Basic Syntax **Cypher**

```
MATCH (X:Type) WHERE X.property = "value" RETURN X
```

### 1. Find Users with "Password Never Expires"

```
MATCH (u:User {dontreqpreauth: true}) 
RETURN u
```
### 2. Find Workstations Where Domain User is Local Admin


```
MATCH (u:User {name: "DOMAIN_USER@DOMAIN.LOCAL"}), (c:Computer)
MATCH (u)-[:AdminTo]->(c)
RETURN u, c
```
### 3. Find Kerberoastable Users with Path to Domain Admin


```
MATCH (u:User {hasspn: true})
MATCH (g:Group {name: "DOMAIN ADMINS@DOMAIN.LOCAL"})
MATCH p = shortestPath((u)-[*1..]->(g))
RETURN p
```

### 4. Find Users with Multiple Admin Rights

```
MATCH (u:User)
MATCH (u)-[:AdminTo]->(c:Computer)
WITH u, COUNT(c) as adminCount
WHERE adminCount > 5
RETURN u, adminCount
```

### 5. Find Computers with LAPS (ms-Mcs-AdmPwd attribute)


```
MATCH (c:Computer) 
WHERE c.haslaps = true
RETURN c
```
### 6. Find Cross-Domain Relationships



```
MATCH (u:User)-[:MemberOf]->(g:Group)-[:MemberOf]->(g2:Group)
WHERE g.domain <> g2.domain
RETURN u, g, g2
```

---

## Advanced Custom Queries

### Find Shortest Path Including Specific Properties:

```
MATCH (u:User {enabled: true}), (g:Group {name: "DOMAIN ADMINS@DOMAIN.LOCAL"})
MATCH p = shortestPath((u)-[r:MemberOf|AdminTo|HasSession*1..]->(g))
WHERE NONE(x IN NODES(p) WHERE x.owned = true)
RETURN p
```

### Find All Paths to High-Value Targets:

```
MATCH (u:User), (targets:Group)
WHERE targets.name =~ ".*ADMIN.*"
MATCH p = allShortestPaths((u)-[*1..]->(targets))
RETURN p
```

---

## Pro Tips for Custom Queries

**Common Node Types:**

- `(u:User)` - User accounts
- `(c:Computer)` - Computer objects
- `(g:Group)` - Security groups
- `(d:Domain)` - Domains

**Common Relationships:**

- `:MemberOf` - Group membership
- `:AdminTo` - Local admin rights
- `:HasSession` - Logged-on users
- `:TrustedBy` - Domain trusts

 **Useful Node Properties:**

- `u.enabled` - Whether account is enabled
- `u.hasspn` - Kerberoastable accounts
- `u.dontreqpreauth` - AS-REP roastable
- `c.operatingsystem` - OS information


## Reference

- https://neo4j.com/docs/cypher-manual/current/introduction/