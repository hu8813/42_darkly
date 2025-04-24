# Form Value Tampering Vulnerability

## Overview
The survey form at `/page=survey` trusts user input and doesn’t check the value server-side. This lets anyone submit values outside the allowed range.

---

## How It Works

1. **Find the Survey Form:**  
   Go to `/page=survey`.
2. **Inspect the Dropdown:**  
   Only allows numbers 1–10:
   ```html
   <option value="1">1</option>
   ...
   <option value="10">10</option>
   ```
3. **Tamper the Value:**  
   - Open DevTools, right-click any `<option>`, and choose "Edit as HTML".
   - Change it to:
     ```html
     <option value="1000">1000</option>
     ```
   - Select the new option; the form submits with this value and reveals the flag.

---

## Security Impact

- **Arbitrary Input:** Attackers can send any value.
- **Data Corruption:** Survey results become unreliable.
- **Logic Bypass:** Application rules are ignored.

---

## Flag

`03a944b434d5baff05f46c4bede5792551a2595574bcafc9a6e25f67c382ccaa`

---

## Prevention

1. Validate input values on the server.
2. Only accept known, expected values (whitelisting).
3. Check data types and ranges.
4. Validate on both client and server sides.
5. Show proper errors for invalid submissions.

---