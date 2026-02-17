
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

- Captcha 
	1. Assess CAPTCHA challenges and attempt automating solutions depending on difﬁculty.
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
	(for example if CAPTCHA is the ﬁrst step in a login process, try simply submitting the second step [username andpassword]).
	10. Check for alternative methods that might not have CAPTCHA enforced, such as an API endpoint meant to facilitate mobile app access.


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


-  Reset Password 
	- Check if the reset password depends on the host Param 
		- change the host param in the HTTP  to the collab and send the password  it may send to the arbitraty email to the collab leaking token and or username
			-
- VM 
	- Windows Task Manager (press CTRL+ALT+DEL or CTRL+ALT+END, select the Task Manager, and then access File > Run new task > cmd.exe)
	- Windows sticky keys (press the shift key 5 times to access a hidden menu and the Control Panel)