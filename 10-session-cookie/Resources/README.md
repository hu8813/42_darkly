# Cookie Manipulation Vulnerability

## Overview
This vulnerability allows unauthorized admin access by changing the `I_am_admin` cookie value, bypassing authentication controls.

---

## Discovery & Exploitation

### 1. Identify the Cookie
- In Chrome Dev Tools (Application → Cookies), find `I_am_admin` with value:
  ```
  68934a3e9455fa72420237eb05902327
  ```
  (This is the MD5 hash for `"false"`—you can decode it using [dcode.fr](https://www.dcode.fr/md5-hash) or [crackstation.net](https://crackstation.net).)

### 2. Exploit
1. Generate the MD5 hash of `"true"`:  
   `b326b5062b2f0e69046810717534cb09`
2. Edit the cookie in your browser:
   - Name: `I_am_admin`
   - Value: `b326b5062b2f0e69046810717534cb09`
3. Refresh the page.
4. You now have admin access and see the flag.

---

## Flag

`df2eb4ba34ed059a1e3e89ff4dfc13445f104a1a52295214def1c4fb1693a5c3`

---

## Prevention

1. **Server-Side Session Validation:** Never trust client-side cookies for auth state.
2. **HMAC Signatures:** Sign cookies to detect tampering.
3. **JWT with Encryption:** Use secure tokens instead of simple cookies.
4. **Cookie Security Flags:** Use HttpOnly, Secure, and SameSite.
5. **Session Regeneration:** Change session IDs after privilege changes.

---