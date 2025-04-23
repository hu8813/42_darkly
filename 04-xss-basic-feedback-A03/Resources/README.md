# Stored Cross-Site Scripting (XSS) Vulnerability on Feedback Page

## Overview

A stored XSS vulnerability exists on the "Leave a Feedback" page. The form fails to properly validate and sanitize user input, allowing the injection and storage of script-related keywords that can trigger client-side code execution.

---

## Discovery & Exploitation

- Submitting the feedback form with empty fields triggers a JS error:  
  `Uncaught ReferenceError: checkForm is not defined`
- The form's submit button uses:  
  `<input name="btnSign" type="Submit" value="Sign Guestbook" onclick="return checkForm();">`  
  but the `checkForm` function is missing, so no client-side validation is performed.
- Entering keywords like `script`, `alert` (without `<` or `>`) in the Name or Message field stores them as-is.
- When these keywords appear on the page, the application reveals the flag.

**Flag:**  
`0fbb54bbf7d099713ca4be297e1bc7da0173d8b3c21c1811b916a3a86652724e`

---

## Vulnerability

**Stored XSS** lets an attacker inject code that will execute for every user visiting the affected page.  
Potential consequences:
- Theft of cookies/session tokens
- Phishing and data theft
- Defacement or malicious redirection

---

## Prevention

1. **Sanitize User Input**
    - Use functions like `strip_tags()` and `htmlspecialchars()` in PHP to remove or escape dangerous input.
2. **Validate Input**
    - Implement server-side and client-side validation for all fields.
3. **Use HttpOnly Cookies**
    - Set cookies with the `HttpOnly` flag to prevent JS access.
4. **Content Security Policy (CSP)**
    - Use a CSP header to restrict executable sources.
5. **Modern Frameworks**
    - Use frameworks with built-in XSS protections.
6. **Web Application Firewall (WAF)**
    - Deploy a WAF to block common XSS attack patterns.

---

> Always sanitize and validate user input, encode output, and use modern security practices to prevent XSS.