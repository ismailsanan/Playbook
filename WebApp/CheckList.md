
-  information leak ?
	-  check using search engines `<leaked data> cve or rce `
	- check using search engine `<leaked data>  pentest`


- Dir Enum
	- GoBuster wordlist `directories` , `files`
	- check always `/dev`
	- if we have an `/admin` do a nested enum
	- nested enum for intersting dirs


- Login 
	- check for SQLI 
	- check for default credentials  `root:root , admin:admin , root: ,admin:   `
	- check online for default credentials 
	- Parameter modiﬁcation
	- Session ID prediction if session ID generation is predictable

- Lockout Mechanism
	1.  Attempt to log in with an incorrect password 3 times.
	

- Unlock Mechanism
	- Return a consistent message for both existent and non-existent accounts
	- timing attacks
	- does not Use a side-channel to communicate the method to reset their password.
	- Ensure that generated tokens or codes are NOT :
	    - Randomly generated using a cryptographically safe algorithm.
	    - Sufficiently long to protect against brute-force attacks.
	    - Stored securely.
	    - Single use and expire after an appropriate period.
	- Do not make a change to the account until a valid token is presented, such as locking out the account.

- Remember Me

- VM 
	- Windows Task Manager (press CTRL+ALT+DEL or CTRL+ALT+END, select the Task Manager, and then access File > Run new task > cmd.exe)
	- Windows sticky keys (press the shift key 5 times to access a hidden menu and the Control Panel)


- Registration

	- Duplicate registration (try with uppercase, +1@..., dots in name, etc)

	- Overwrite existing user (existing user takeover)

	- Username uniqueness

	- Weak password policy (user=password, password=123456,111111,abcabc,qwerty12)

	- [Insufficient email verification process](https://www.pentest-book.com/enumeration/web/email-attacks) (also my%00email@mail.com for account tko)

	- Weak registration implementation or allows disposable email addresses

	- Fuzz after user creation to check if any folder have been overwritten or created with your profile name
	    
	- Add only spaces in password
	    
	- Long password (>200) leads to DoS
	    
	- Corrupt authentication and session defects: Sign up, don't verify, request change password, change, check if account is active.
	    
	- Try to re-register repeating same request with same password and different password too
	    
	- If JSON request, add comma {“email”:“victim@mail.com”,”hacker@mail.com”,“token”:”xxxxxxxxxx”}
	    
	- Lack of confirmation -> try to register with company email.
	    
	- Check OAuth with social media registration
	    
	- Check state parameter on social media registration
	    
	- Try to capture integration url leading integration takeover
	    
	- Check redirections in register page after login
	    
	- Rate limit on account creation
	    
	- XSS on name or email
    

- Authentication

	- Username enumeration
	    
	- Resilience to password guessing
	    
	- Account recovery function
	    
	- "Remember me" function
	    
	- Impersonation function
	    
	- Unsafe distribution of credentials
	    
	- Fail-open conditions
	    
	- Multi-stage mechanisms
	    
	- [SQL Injections](https://www.pentest-book.com/enumeration/web/sqli)
	    
	- Auto-complete testing
	    
	- Lack of password confirmation on change email, password or 2FA (try change response)
	    
	- Weak login function over HTTP and HTTPS if both are available
	    
	- User account lockout mechanism on brute force attack
	    
	- Check for password wordlist ([cewl](https://github.com/digininja/CeWL) and [burp-goldenNuggets](https://github.com/GainSec/GoldenNuggets-1))
	    
	- Test 0auth login functionality for [Open Redirection](https://www.pentest-book.com/enumeration/web/ssrf)
	    
	- Test response tampering in [SAML](https://www.pentest-book.com/enumeration/webservices/onelogin-saml-login) authentication
	    
	- In OTP check guessable codes and race conditions
	    
	- OTP, check response manipulation for bypass
	    
	- OTP, try bruteforce
	    
	- If [JWT](https://www.pentest-book.com/enumeration/webservices/jwt), check common flaws
	    
	- Browser cache weakness (eg Pragma, Expires, Max-age)
	    
	- After register, logout, clean cache, go to home page and paste your profile url in browser, check for "login?next=accounts/profile" for open redirect or XSS with "/login?next=javascript:alert(1);//"
	    
	- Try login with common [credentials](https://github.com/ihebski/DefaultCreds-cheat-sheet)
	    


- Session

	- Session handling
	    
	- Test tokens for meaning
	    
	- Test tokens for predictability
	    
	- Insecure transmission of tokens
	    
	- Disclosure of tokens in logs
	    
	- Mapping of tokens to sessions
	    
	- Session termination
	    
	- Session fixation
	    
	- [Cross-site request forgery](https://www.pentest-book.com/enumeration/web/csrf)
	    
	- Cookie scope
	    
	- Decode Cookie (Base64, hex, URL etc.)
	    
	- Cookie expiration time
	    
	- Check HTTPOnly and Secure flags
	    
	- Use same cookie from a different effective IP address or system
	    
	- Access controls
	    
	- Effectiveness of controls using multiple accounts
	    
	- Insecure access control methods (request parameters, Referer header, etc)
	    
	- Check for concurrent login through different machine/IP
	    
	- Bypass [AntiCSRF](https://www.pentest-book.com/enumeration/web/csrf#csrf-token-bypass) tokens
	    
	- Weak generated security questions
	    
	- Path traversal on cookies
	    
	- Reuse cookie after session closed
	    
	- Logout and click browser "go back" function (Alt + Left arrow)
	    
	- 2 instances open, 1st change or reset password, refresh 2nd instance
	    
	- With privileged user perform privileged actions, try to repeat with unprivileged user cookie.
	    


- Profile/Account details

	- Find parameter with user id and try to tamper in order to get the details of other users
	    
	- Create a list of features that are pertaining to a user account only and try [CSRF](https://www.pentest-book.com/enumeration/web/csrf)
	    
	- Change email id and update with any existing email id. Check if its getting validated on server or not.
	    
	- Check any new email confirmation link and what if user doesn't confirm.
	    
	- File [upload](https://www.pentest-book.com/enumeration/web/upload-bypasses): [eicar](https://secure.eicar.org/eicar.com.txt), No Size Limit, File extension, Filter Bypass, [burp](https://github.com/portswigger/upload-scanner) extension, RCE
	    
	- CSV import/export: Command Injection, XSS, macro injection
	    
	- Check profile picture URL and find email id/user info or [EXIF Geolocation Data](http://exif.regex.info/exif.cgi)
	    
	- Imagetragick in picture profile upload
	    
	- [Metadata](https://github.com/exiftool/exiftool) of all downloadable files (Geolocation, usernames)
	    
	- Account deletion option and try to reactivate with "Forgot password" feature
	    
	- Try bruteforce enumeration when change any user unique parameter.
	    
	- Check application request re-authentication for sensitive operations
	    
	- Try parameter pollution to add two values of same field
	    
	- Check different roles policy
    


Forgot/reset password

- Invalidate session on Logout and Password reset
    
- Uniqueness of forget password reset link/code
    
- Reset links expiration time
    
- Find user id or other sensitive fields in reset link and tamper them
    
- Request 2 reset passwords links and use the older
    
- Check if many requests have sequential tokens
    
- Use username@burp_collab.net and analyze the callback
    
- Host header injection for token leakage
    
- Add X-Forwarded-Host: evil.com to receive the reset link with evil.com
    
- Email crafting like victim@gmail.com@target.com
    
- IDOR in reset link
    
- Capture reset token and use with other email/userID
    
- No TLD in email parameter
    
- User carbon copy email=victim@mail.com%0a%0dcc:hacker@mail.com
    
- Long password (>200) leads to DoS
    
- No rate limit, capture request and send over 1000 times
    
- Check encryption in reset password token
    
- Token leak in referer header
    
- Append second email param and value
    
- Understand how token is generated (timestamp, username, birthdate,...)
    
- Response manipulation

- Check if the reset password depends on the host Param  change the host param in the HTTP  to the collab and send the password  it may send to the arbitraty email to the collab leaking token and or username




- Input handling

	- Fuzz all request parameters (if got user, add headers to fuzzer)
	    
	- Identify all reflected data
	    
	- [Reflected XSS](https://www.pentest-book.com/enumeration/web/xss)
	    
	- HTTP [header injection](https://www.pentest-book.com/enumeration/web/header-injections) in GET & POST (X Forwarded Host)
	    
	- RCE via Referer Header
	    
	- SQL injection via User-Agent Header
	    
	- Arbitrary redirection
	    
	- Stored attacks
	    
	- OS command injection
	    
	- Path [traversal](https://www.pentest-book.com/enumeration/web/lfi-rfi), LFI and RFI
	    
	- Script injection
	    
	- File inclusion
	    
	- SMTP injection
	    
	- Native software flaws (buffer overflow, integer bugs, format strings)
	    
	- SOAP injection
	    
	- LDAP injection
	    
	- SSI Injection
	    
	- XPath injection
	    
	- [XXE](https://www.pentest-book.com/enumeration/web/xxe) in any request, change content-type to text/xml
	    
	- Stored [XSS](https://www.pentest-book.com/enumeration/web/xss)
	    
	- [SQL](https://www.pentest-book.com/enumeration/web/sqli) injection with ' and '--+-
	    
	- [NoSQL](https://www.pentest-book.com/enumeration/webservices/nosql-and-and-mongodb) injection
	    
	- HTTP Request [Smuggling](https://www.pentest-book.com/enumeration/web/request-smuggling)
	    
	- [Open redirect](https://www.pentest-book.com/enumeration/web/ssrf)
	    
	- Code Injection (\<h1>six2dez\</h1> on stored param)
    
	- [SSRF](https://www.pentest-book.com/enumeration/web/ssrf) in previously discovered open ports
	    
	-  user enumeration
	    
	- HTTP dangerous methods OPTIONS PUT DELETE
	    
	- Try to discover hidden parameters ([arjun](https://github.com/s0md3v/Arjun) or [parameth](https://github.com/maK-/parameth))
	    
	- Insecure deserialization
    


- Error handling

	- Access custom pages like /whatever_fake.php (.aspx,.html,.etc)
	    
	- Add multiple parameters in GET and POST request using different values
	    
	- Add "[]", "]]", and "[[" in cookie values and parameter values to create errors
	    
	- Generate error by giving input as "/~randomthing/%s" at the end of URL
	    
	- Use Burp Intruder "Fuzzing Full" List in input to generate error codes
	    
	- Try different HTTP Verbs like PATCH, DEBUG or wrong like FAKE


- Application Logic

	- Identify the logic attack surface
	    
	- Test transmission of data via the client
	    
	- Test for reliance on client-side input validation
	    
	- Thick-client components (Java, ActiveX, Flash)
	    
	- Multi-stage processes for logic flaws
	    
	- Handling of incomplete input
	    
	- Trust boundaries
	    
	- Transaction logic
	    
	- Implemented CAPTCHA in email forms to avoid flooding
	    
	- Tamper product id, price or quantity value in any action (add, modify, delete, place, pay...)
	    
	- Tamper gift or discount codes
	    
	- Reuse gift codes
	    
	- Try parameter pollution to use gift code two times in same request
	    
	- Try stored XSS in non-limited fields like address
	    
	- Check in payment form if CVV and card number is in clear text or masked
	    
	- Check if is processed by the app itself or sent to 3rd parts
	    
	- IDOR from other users details ticket/cart/shipment
	    
	- Check for test credit card number allowed like 4111 1111 1111 1111 ([sample1](https://www.paypalobjects.com/en_GB/vhelp/paypalmanager_help/credit_card_numbers.htm) [sample2](http://support.worldpay.com/support/kb/bg/testandgolive/tgl5103.html))
	    
	- Check PRINT or PDF creation for IDOR
	    
	- Check unsubscribe button with user enumeration
	    
	- Parameter pollution on social media sharing links
	    
	- Change POST sensitive requests to GET
    


- Other checks
- Infrastructure
	
	- Segregation in shared infrastructures
	    
	- Segregation between ASP-hosted applications
	    
	- Web server vulnerabilities
	    
	- Dangerous HTTP methods
	    
	- Proxy functionality
	    
	- [Virtual](https://www.pentest-book.com/enumeration/webservices/vhosts) hosting misconfiguration ([VHostScan](https://github.com/codingo/VHostScan))
	    
	- Check for internal numeric IP's in request
	    
	- Check for external numeric IP's and resolve it
	    
	- Test [cloud](https://www.pentest-book.com/enumeration/cloud/cloud-info-recon) storage
	    
	- Check the existence of alternative channels (www.web.com vs m.web.com)
	    

- CAPTCHA

	- Send old captcha value.
	    
	- Send old captcha value with old session ID.
	    
	- Request captcha absolute path like www.url.com/captcha/1.png
	    
	- Remove captcha with any adblocker and request again
	    
	- Bypass with OCR tool ([easy one](https://github.com/pry0cc/prys-hacks/blob/master/image-to-text))
	    
	- Change from POST to GET
	    
	- Remove captcha parameter
	    
	- Convert JSON request to normal
	    
	- Try header injections
	
	-  Assess CAPTCHA challenges and attempt automating solutions depending on difﬁculty.
	2. Attempt to submit request without solving CAPTCHA via the normal UI mechanism(s).
	3. Attempt to submit request with intentional CAPTCHA challenge failure.
	4. Attempt to submit request without solving CAPTCHA (assuming some default values may be passed by client-side code, etc) while using a testing proxy (request submitted directly server-side).
	5. Attempt to fuzz CAPTCHA data entry points (if present) with common injection payloads or special characters
	sequences.
	6. Check if the solution to the CAPTCHA might be the alt-text of the image(s), ﬁlename(s), or a value in an associated
	hidden ﬁeld.
	7. Attempt to re-submit previously identiﬁed known good responses.
	8. Check if clearing cookies causes the CAPTCHA to be bypassed (for example if the CAPTCHA is only shown after a
	number of failures).
	9. If the CAPTCHA is part of a multi-step process, attempt to simply access or complete a step beyond the CAPTCHA
	(for example if CAPTCHA is the ﬁrst step in a login process, try simply submitting the second step [username and password]).
	10. Check for alternative methods that might not have CAPTCHA enforced, such as an API endpoint meant to facilitate mobile app access.

- Security Headers

	- X-XSS-Protection
	    
	- Strict-Transport-Security
	    
	- Content-Security-Policy
	    
	- Public-Key-Pins
	    
	- X-Frame-Options
	    
	- X-Content-Type-Options
	    
	- Referer-Policy
	    
	- Cache-Control
	    
	- Expires


### Reference

- https://www.pentest-book.com/others/web-checklist
- OWASP PDF
- Hacktricks