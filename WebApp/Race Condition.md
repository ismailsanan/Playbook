tactic: Privilege Escalation

**Race Conditions** occur when a web application processes concurrent requests without proper safeguards, causing multiple threads to interact with the same data simultaneously. The result is a **collision** — two requests that should be mutually exclusive both succeed.

The period during which a collision is possible is called the **race window**.

```
Normal flow (sequential):
Request 1: check coupon → valid → apply → mark as used
Request 2: check coupon → already used → reject

Race condition (parallel):
Request 1: check coupon → valid ─────────────┐
Request 2: check coupon → valid ─────────────┤  both pass the check
Request 1: apply + mark used                 │  before either marks it used
Request 2: apply + mark used                 ┘
```



### Detecting and exploiting multi step

- Identify a single-use that has some kind of security impact or other useful purpose
- Issue multiple requests to this endpoint in quick succession to see if you can overrun this limit
the primary challenge is timing the requests so that at least two race windows line up, causing a collision

we typically need 2 or more requests that trigger operations on the same record  

- send group in sequence
- use the single packet attack HTTP/2 not supported
- send group in parallel


The single-packet attack enables you to completely neutralize interference from network jitter by using a single TCP packet to complete 20-30 requests simultaneously

- For HTTP/1, it uses the classic last-byte synchronization technique.
- For HTTP/2, it uses the single-packet attack technique


## Techniques

**HTTP/2 Single-Packet Attack** — sends 20-30 requests in a single TCP packet, completely eliminating network jitter. All requests arrive at the server simultaneously. Use this when the target supports HTTP/2.

**HTTP/1 Last-Byte Synchronization** — hold back the final byte of each request, then send all last bytes simultaneously to synchronize arrival. Use when HTTP/2 is not supported.

> Single-packet attack is incompatible with HTTP/1. Always check which protocol the target uses.

**Network jitter** refers to packets arriving at irregular times. A warming request (a cheap, throwaway request sent first) warms up the connection so subsequent requests have consistent low latency.

---

## Injection Points

- Coupon and discount code redemption endpoints
- Gift card and voucher application
- Password reset token generation
- Email verification link endpoints
- Cart and checkout flows
- Rate-limited login endpoints
- One-time use token endpoints
- Any endpoint that checks a condition then acts on it (check-then-act)

 **Attack Setup in Burp**
 
Burp Repeater has 3 options when sending a group of requests:

**Send group in sequence (single connection)** — sends all requests one after another over the same TCP connection. Use this to warm up the connection and measure baseline latency before attempting a race.

**Send group in sequence (separate connections)** — sends each request over its own connection. Use this to check if requests are being processed sequentially vs concurrently, or when testing with different session tokens.

**Send group in parallel** — sends all requests simultaneously. For HTTP/2 this uses the single-packet attack (all requests in one TCP packet, eliminates jitter). For HTTP/1 this uses last-byte synchronization. This is the option you use for the actual race condition attack.

```
1. Add all requests to a Repeater tab group
2. Add a warming GET request as the first item
3. Send group in sequence (single connection) first to warm up and check latency
4. Then switch to Send group in parallel for the actual attack
5. Check responses for collision signs (same token, double discount, skipped checks)
```

---


---

## Checklist

### Detection

- [ ]  Send a warming packet first (a harmless GET request) to establish connection and reduce jitter:

```
	GET /   ← warming request, send this first
```

- [ ]  Check if there is a latency gap between the check and the database update — that gap is your race window:

```
	POST /cart/coupon  →  latency before  →  GET /coupon error
```

- [ ]  Send the same request 20-30 times in parallel using Burp's "Send group in parallel (single-packet attack)"
- [ ]  If all requests are processed sequentially, try sending each with a different session token

### Limit Overrun (Coupon/Discount Abuse)

- [ ]  Find a one-time-use coupon or discount endpoint
- [ ]  Add the coupon request to a Burp group, duplicate it 20-30 times
- [ ]  Send all in parallel using single-packet attack
- [ ]  Check if the discount was applied multiple times

### Bypassing Rate Limits (Login Brute Force)

- [ ]  Set Intruder max concurrent requests to 30-50
- [ ]  Send login attempts in parallel to bypass the rate limit window:

```http
	POST /login
	csrf=TOKEN&username=wiener&password=§password§
```

### Multi-Endpoint Race Conditions (Cart Abuse)

- [ ]  Add an item you can afford to the cart
- [ ]  Also add an item you cannot afford
- [ ]  Send the following group in parallel — warming request first:

```
	GET /          ← warming request
	POST /cart     ← add affordable item
	POST /cart/checkout  ← checkout while expensive item is in race window
```

- [ ]  Both requests need to arrive at the same speed — use single-packet attack

### Single-Endpoint Race Conditions (Email Verification Collision)

- [ ]  The concept: if two users trigger email verification simultaneously, the verification context can cross — email A verifies user B's account
- [ ]  Send two parallel requests to the same endpoint with different email values:

```http
	email=carlos%40ginandjuice.shop
	email=wiener%40exploit-server.net
```

- [ ]  Check if the token sent to one email verifies the other account

### Time-Sensitive Token Collision (Password Reset)

- [ ]  Request a password reset and observe the token — check if it has consistent length every time (suggests timestamp-based generation)
- [ ]  Send multiple reset requests in parallel for the same user and confirm if tokens match:

```
	POST /forgot-password  →  username=wiener (×2 in parallel)
```

- [ ]  If both produce the same token → time-sensitive vulnerability confirmed
- [ ]  Then send one request for your account and one for the target simultaneously:

```
	POST /forgot-password  username=carlos
	POST /forgot-password  username=wiener   ← you receive this token
```

- [ ]  Use the token you received on the carlos reset link
- [ ]  Send a warming packet first, then both requests together so they arrive simultaneously
- [ ]  If CSRF token is required, change `POST` to `GET` to bypass it, or pre-fetch a valid CSRF token

---
## White Box Testing

**Vulnerable, check-then-act without locking:**

```java
@PostMapping("/cart/coupon")
public ResponseEntity<?> applyCoupon(@RequestParam String code,
                                      @AuthenticationPrincipal UserDetails user) {
    Coupon coupon = couponRepository.findByCode(code);

    // check and act are two separate DB operations with no lock between them
    if (coupon.isUsed()) {
        return ResponseEntity.badRequest().body("Coupon already used");
    }

    // race window exists between the check above and the update below
    cartService.applyDiscount(user, coupon.getDiscount());
    coupon.setUsed(true);           // two parallel requests both pass the check
    couponRepository.save(coupon);  // before either sets isUsed = true
    return ResponseEntity.ok().build();
}
```

**Vulnerable, password reset token generated from timestamp:**

```java
public String generateResetToken(String username) {
    // tokens generated at the same millisecond will be identical
    long timestamp = System.currentTimeMillis();
    return DigestUtils.md5Hex(username + timestamp); // predictable and race-able
}
```

**Vulnerable, cart checkout without atomic stock check:**

```java
@PostMapping("/cart/checkout")
public ResponseEntity<?> checkout(@AuthenticationPrincipal UserDetails user) {
    Cart cart = cartService.getCart(user);
    for (CartItem item : cart.getItems()) {
        // check and deduct are separate operations, no pessimistic lock
        if (item.getProduct().getStock() < item.getQuantity()) {
            return ResponseEntity.badRequest().body("Out of stock");
        }
        // race window here, another thread can pass the check simultaneously
        productService.deductStock(item.getProduct(), item.getQuantity());
    }
    return ResponseEntity.ok(orderService.createOrder(cart));
}
```

**Secure:**

```java
// Use database-level locking to eliminate the race window
@PostMapping("/cart/coupon")
@Transactional
public ResponseEntity<?> applyCoupon(@RequestParam String code,
                                      @AuthenticationPrincipal UserDetails user) {
    // pessimistic lock prevents any other transaction reading this row until commit
    Coupon coupon = couponRepository.findByCodeWithLock(code); // SELECT ... FOR UPDATE

    if (coupon.isUsed()) {
        return ResponseEntity.badRequest().body("Coupon already used");
    }

    cartService.applyDiscount(user, coupon.getDiscount());
    coupon.setUsed(true);
    couponRepository.save(coupon);
    return ResponseEntity.ok().build();
}

// Repository method using pessimistic lock
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT c FROM Coupon c WHERE c.code = :code")
Coupon findByCodeWithLock(@Param("code") String code);

// Use cryptographically secure random token, not timestamp-based
public String generateResetToken() {
    byte[] bytes = new byte[32];
    new SecureRandom().nextBytes(bytes);
    return Base64.getUrlEncoder().withoutPadding().encodeToString(bytes);
}

// Atomic stock deduction using conditional update
@Modifying
@Query("UPDATE Product p SET p.stock = p.stock - :qty WHERE p.id = :id AND p.stock >= :qty")
int deductStockAtomic(@Param("id") Long id, @Param("qty") int qty);
// returns 0 if stock was insufficient, preventing oversell without a race window
```

 **Bypassing rate limits via race conditions**


>intruder login max concurrent request 30-50
```http

csrf=DOaCVRhcGDF1wrwLpCJvPf1cYhKIBEwk&username=wiener&password=peter
```

 **Multi-endpoint race conditions**

idea is that we add something we can afford to buy  then try to lunch a race condition to a post on item with a checkout to pass that item while we are in the timeframe of the checkout

- add a gift card that you can afford in the cart
- add something u cant afford

> send in parallel since we need it at the same speed rate not 2 different speed latency

```bash
GET / for warmer request
GROUP TAB 
POST /cart
POST /cart/checkout
```


 **Single-endpoint race conditions**

- the concept is colliding now just the email link but also the context of the email could collide meaning email A could verify in the link user B

**parallel**

```http 
email=carlos%40ginandjuice.shop
email=wiener%40exploit-0a5e000303eeba5d83b85903017700d9.exploit-server.net
```


 **Exploiting time-sensitive vulnerabilities**


- click on the forgot password 
- see the link that is carried
- notice that the token has the same length every time 
- send couple of requests in parallel and cofirm that there wqould be token of the same value in both hence they are vuln to race condition
- then send one for `carlos` and `wiener`   check that they would have the same token by trying to duplicate the URL and change the username 
- send a warmer packet first then do it so that we get the requests at the same time
- change `POST` to `GET` for setting up new session cookies if needed and `CSRF` token if needed

## Resources

- [PortSwigger Race Conditions](https://portswigger.net/web-security/race-conditions)
- [James Kettle Race Conditions Research](https://portswigger.net/research/smashing-the-state-machine)
- [HackTricks Race Conditions](https://book.hacktricks.xyz/pentesting-web/race-condition)