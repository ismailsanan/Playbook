**IDN Homograph Attacks** exploit visually identical or near-identical Unicode characters from different scripts to register fake domains that look exactly like legitimate ones. The victim sees `apple.com` but is actually visiting an attacker-controlled server using Cyrillic or Greek lookalike characters.

```
Legitimate:  apple.com        (Latin characters)
Fake:        аррӏе.com        (Cyrillic/Greek characters — looks identical)

Browser converts the fake domain to Punycode internally:
аррӏе.com  →  xn--pple-43d.com

But in the address bar, older browsers and email clients show the Unicode version
which is visually indistinguishable from the real domain
```

## Common Homoglyph Pairs

|Legitimate (Latin)|Lookalike|Script|Unicode|
|---|---|---|---|
|`a`|`а`|Cyrillic|U+0430|
|`e`|`е`|Cyrillic|U+0435|
|`o`|`о`|Cyrillic|U+043E|
|`o`|`ο`|Greek|U+03BF|
|`p`|`р`|Cyrillic|U+0440|
|`p`|`ρ`|Greek|U+03C1|
|`c`|`с`|Cyrillic|U+0441|
|`x`|`х`|Cyrillic|U+0445|
|`y`|`у`|Cyrillic|U+0443|
|`i`|`і`|Ukrainian|U+0456|
|`l`|`ӏ`|Chechen|U+04CF|
|`n`|`η`|Greek|U+03B7|
|`u`|`υ`|Greek|U+03C5|

---

## Attack Flow

```
1. Identify target domain (bank.com, paypal.com, apple.com)
2. Find visually identical Unicode replacements for one or more characters
3. Register the homograph domain (аррӏе.com)
4. Set up a cloned version of the legitimate site on your server
5. Deliver the fake link via phishing email, SMS, or ad
6. Victim clicks, sees familiar-looking URL, enters credentials
7. Credentials captured on attacker's server
```

---

## Checklist

### Offensive (Red Team / Phishing Assessment)

- [ ]  Identify target domain and map characters susceptible to homoglyph substitution
- [ ]  Use a homoglyph generator to create candidate domains:

```
	https://www.irongeek.com/homoglyph-attack-generator.php
	https://util.unicode.org/UnicodeJsps/confusables.jsp
```

- [ ]  Convert your candidate to Punycode to check registerability:

```bash
	python3 -c "import encodings.idna; print(encodings.idna.ToACE('аррӏе.com'))"
	# or use: https://www.punycodeworld.com/
```

- [ ]  Check if the domain is already registered:

```bash
	whois xn--pple-43d.com
```

- [ ]  Register via a registrar that supports IDN domains (Namecheap, GoDaddy)
- [ ]  Clone the target site and host it on your infrastructure
- [ ]  Set up credential capture (modify the login form to POST to your collector)
- [ ]  Deliver via phishing email — embed the Unicode domain in the `href` while showing the clean version as link text:

```html
	<a href="http://аррӏе.com/login">https://apple.com/login</a>
```

### Defensive / Detection

- [ ]  Check if your domain has registered homograph variants — monitor for lookalike domains:

```bash
	# Use dnstwist to enumerate lookalike domains
	dnstwist --homoglyphs apple.com
```

- [ ]  Check if your browser displays Punycode for mixed-script domains — it should
- [ ]  Verify email links by hovering before clicking — check the status bar shows the real domain
- [ ]  Implement DMARC, DKIM, SPF to reduce email spoofing alongside domain spoofing

---

## Tools

**Generate homoglyph candidates:**

```bash
# dnstwist — comprehensive lookalike domain scanner
pip install dnstwist
dnstwist --homoglyphs --registered apple.com

# URLCrazy — typosquatting and homograph generation
urlcrazy apple.com
```

**Convert to Punycode:**

```bash
python3 -c "print('аррӏе.com'.encode('idna').decode())"
```

**Check browser rendering:**

- Chrome: displays Punycode for mixed-script domains
- Firefox: shows Unicode only if all chars are from the same script as the TLD
- Always test your fake domain in the target browser

## Punycode Reference

When a domain uses non-ASCII characters it is encoded as Punycode, prefixed with `xn--`:

```
аррӏе.com     →   xn--pple-43d.com
pаypal.com    →   xn--pypal-4ve.com
gооgle.com    →   xn--oogle-qmc2a.com
```

Some browsers show Punycode in the address bar automatically for mixed-script domains — but not all, and email clients almost never do.

---

## Defensive Code (Application Level)

**Detect and reject IDN/homograph domains in user-submitted URLs:**

```java
@Component
public class UrlValidator {

    public boolean isSuspiciousUrl(String url) throws Exception {
        URI uri = new URI(url);
        String host = uri.getHost();

        // check if hostname contains non-ASCII characters
        if (!host.equals(new String(host.getBytes(StandardCharsets.US_ASCII),
                                     StandardCharsets.US_ASCII))) {
            return true; // contains non-ASCII → potential homoglyph
        }

        // check if Punycode encoded form differs from original (IDN domain)
        String punycode = java.net.IDN.toASCII(host);
        if (!punycode.equals(host)) {
            return true; // IDN domain — flag for review
        }

        return false;
    }
}
```

**Check submitted URLs against known lookalike patterns:**

```java
// Normalise domain to detect homoglyphs before processing
public String normalizeDomain(String domain) {
    // convert to NFC unicode normalization first
    String normalized = Normalizer.normalize(domain, Normalizer.Form.NFC);
    // then convert to Punycode
    return IDN.toASCII(normalized, IDN.ALLOW_UNASSIGNED);
}

// Compare against your own domain — reject if suspiciously similar
private static final String OWN_DOMAIN = "apple.com";

public boolean isSpoofingOurDomain(String submittedDomain) {
    String ascii = normalizeDomain(submittedDomain);
    // exact match after normalization is fine, anything else with xn-- is suspect
    return ascii.contains("xn--") && ascii.contains("apple");
}
```

## Resources

- [Homoglyph Attack Generator](https://www.irongeek.com/homoglyph-attack-generator.php)
- [Unicode Confusables](https://util.unicode.org/UnicodeJsps/confusables.jsp)
- [dnstwist](https://github.com/elceef/dnstwist)
- [URLCrazy](https://github.com/urbanadventurer/urlcrazy)
- [Punycode Converter](https://www.punycodeworld.com/)