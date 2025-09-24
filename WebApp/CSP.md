
The **Content Security Policy (CSP)**  allows you to specify which resources (e.g., scripts, styles, images) that can be loaded by the browser, thereby providing  control over the content your site can load and execute.

### **Overview of Content Security Policy (CSP)**

**CSP Header Format**:
The CSP header is set by the web server and looks like this:

```
Content-Security-Policy: <policy>
```

**Example**:
```
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.cdn.com; style-src 'self' https://trusted.cdn.com; img-src 'self' data:;
```

### **Key Directives in CSP**

Here’s a breakdown of some important CSP directives and what they control:

#### **1. `default-src`**
- **Description**: Serves as a fallback for other resource types if no specific directive is provided.
- **Example**: `default-src 'self'` means only resources from the same origin are allowed.

#### **2. `script-src`**
- **Description**: Controls which scripts can be executed. This includes inline scripts and scripts from external sources.
- **Example**: `script-src 'self' https://trusted.cdn.com` allows scripts from the same origin and the specified CDN.

#### **3. `style-src`**
- **Description**: Controls which stylesheets can be applied. This includes inline styles and styles from external sources.
- **Example**: `style-src 'self' https://trusted.cdn.com` allows styles from the same origin and the specified CDN.

#### **4. `img-src`**
- **Description**: Specifies which sources of images are allowed. This can include data URIs, external domains, etc.
- **Example**: `img-src 'self' data:` allows images from the same origin and inline images (base64 encoded).

#### **5. `connect-src`**
- **Description**: Controls which sources can be accessed via `XMLHttpRequest`, `Fetch API`, or WebSockets.
- **Example**: `connect-src 'self' https://api.example.com` allows connections to the same origin and the specified API endpoint.

#### **6. `frame-ancestors`**
- **Description**: Specifies which sources are allowed to embed the page in an iframe. This is used to prevent clickjacking.
- **Example**: `frame-ancestors 'none'` prevents any domains from framing the content.

#### **7. `object-src`**
- **Description**: Controls which sources can be used for `<object>`, `<embed>`, and `<applet>` elements.
- **Example**: `object-src 'none'` disallows all `<object>`, `<embed>`, and `<applet>` elements.

#### **8. `form-action`**
- **Description**: Specifies the allowed destinations for form submissions.
- **Example**: `form-action 'self'` allows forms to be submitted only to the same origin.

#### **9. `base-uri`**
- **Description**: Restricts the URLs that can be used as a base URL for relative URLs.
- **Example**: `base-uri 'self'` restricts the base URL to the same origin.

#### **10. `sandbox`**
- **Description**: Provides an additional layer of security by restricting the capabilities of the content inside an iframe.
- **Example**: `sandbox allow-scripts` allows scripts but restricts other features like form submissions and navigation.




### **Impact of a Well-Configured CSP**

- **Prevents XSS Attacks**: By restricting the sources from which scripts can be loaded and executed, CSP reduces the risk of XSS attacks.
- **Mitigates Data Injection Attacks**: Controls the sources for various types of content, preventing malicious content from being injected and executed.
- **Protects Against Clickjacking**: Using the `frame-ancestors` directive prevents your site from being framed by unauthorized sources.
