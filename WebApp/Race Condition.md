
they occur when websites process requests concurrently without safeguards leading multiple distinct threads interacting with the same data at the same time  resulting **collision**

meaning 2 parallel requests that would lead to collision and passing them both 

the period of time during which a collision is possible  is known as "Race window" 


HTTP/1, it uses the classic last-byte synchronization technique.
HTTP/2, it uses the single-packet attack technique.

single-packet attack is incompatible with HTTP/1

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

### Notes

- Network jitter refers to the packet arriving  at irregular times
-  always check if there is delay in the packages and try to understand if this delay is caused from the backed or  from the network what
### ToCheck



 - [ ] send a warming packet a useless one to see if the latency increased or decreased for the other requests 
 - sending the `GET /cart` request both with and without your session cookie. Confirm that without the session cookie, you can only access an empty cart. From this, you can infer that:
	 - The state of the cart is stored server-side in your session.
	 -  Any operations on the cart are keyed on your session ID or the associated user ID.
- [ ] If you notice that all of your requests are being processed sequentially, try sending each of them using a different session token.
- [ ] test for  the user itself in parallel request to see if its vulnerable to race condition  if so we can use forget password to the user and other users to generate same token and use it 


# Limit overrun race conditions

check that there is a latency between the request POST add coupon and GET coupon error that means that it takes time from when it takes the coupon and when it generates the get db error 

generate a post coupon request place them in a folder duplicate the request 30 times and send them in parallel single packet  

# Bypassing rate limits via race conditions


>intruder login max concurrent request 30-50
```http

csrf=DOaCVRhcGDF1wrwLpCJvPf1cYhKIBEwk&username=wiener&password=peter
```

# Multi-endpoint race conditions

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


# Single-endpoint race conditions

- the concept is colliding now just the email link but also the context of the email could collide meaning email A could verify in the link user B

**parallel**

```http 
email=carlos%40ginandjuice.shop
email=wiener%40exploit-0a5e000303eeba5d83b85903017700d9.exploit-server.net
```


# Exploiting time-sensitive vulnerabilities


- click on the forgot password 
- see the link that is carried
- notice that the token has the same length every time 
- send couple of requests in parallel and cofirm that there wqould be token of the same value in both hence they are vuln to race condition
- then send one for `carlos` and `wiener`   check that they would have the same token by trying to duplicate the URL and change the username 
- send a warmer packet first then do it so that we get the requests at the same time
- change `POST` to `GET` for setting up new session cookies if needed and `CSRF` token if needed