
 A trick that exploits visually similar characters from different writing systems (Unicode) to create fake domain names that look identical or extremely close to legitimate ones.


- **How it works**:
    
    1. **Homoglyphs:** Attackers register domains using characters that look like letters in the target domain. For example:
        
        - Latin `a` (U+0061) vs. Cyrillic `а` (U+0430)
            
        - Latin `e` (U+0065) vs. Cyrillic `е` (U+0435)
            
        - Latin `o` (U+006F) vs. Greek `ο` (U+03BF)
            
        - Latin `p` (U+0070) vs. Greek `ρ` (U+03C1)
            
        - Latin `c` (U+0063) vs. Cyrillic `с` (U+0441)
            
    2. **Deceptive URL:** The attacker creates a URL like `аррӏе.com` (using Cyrillic/Greek characters) instead of `apple.com` (Latin). To an unsuspecting user, they look nearly identical.
        
    3. **Phishing:** Users receive a link (email, message, ad) to the fake domain. They think they're visiting the real site (e.g., their bank, Apple, PayPal) but are actually on the attacker's server.
        
    4. **The Trap:** The fake site is designed to mimic the real one, stealing login credentials, credit card info, or installing malware.

- **Mitigation**: Modern browsers implement **Punycode conversion** and display warnings or the actual Punycode (like `xn--pple-43d.com`) for mixed-script domains to make the deception obvious. Users should carefully examine URLs before clicking.
    

**Example:** `www.аррӏе.com` (looks like `www.apple.com` but uses Cyrillic/Greek chars) points to a phishing site.