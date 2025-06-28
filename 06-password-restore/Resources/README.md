# Hidden Field Manipulation Vulnerability

## Overview
In the password recovery form (`/?page=recover`), a hidden field sets the admin’s email. Anyone can change this field and hijack the password reset.

---

## How It Works

1. **Find the Form:**  
   Go to the recovery page and inspect the form:
   ```html
   <input type="hidden" name="mail" value="webmaster@borntosec.com">
   ```
2. **Exploit:**  
   - Change the value to your own email using DevTools and submit Form
3. **Result:**  
   The app shows the Flag (reset link (and password) to the attacker’s email).

---

## Security Impact

- **Account Takeover:** Attackers can reset admin passwords.
- **Information Disclosure:** Internal emails exposed.
- **Impersonation:** Attackers can pretend to be admins.

---

## Flag

`1D4855F7337C0C14B6F44946872C4EB33853F40B2D54393FBE94F49F1E19BBB0`

---

## Prevention

1. Never trust hidden fields for sensitive data.
2. Always verify identity server-side before sending reset emails.
3. Use sessions or multi-step verification for recovery.
4. Ask challenge questions or use 2FA for password resets.
5. Rate-limit password recovery attempts.

---