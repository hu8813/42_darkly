# Directory Enumeration Vulnerability

## Overview
Sensitive info was hidden in deeply nested folders under `/.hidden/`. No real security—just "security through obscurity."

---

## How It Works

1. **http://<IP_ADDRESS>/robots.txt** gave clues:
   ```
   Disallow: /whatever
   Disallow: /.hidden
   ```
2. **Exploring `/.hidden/`** revealed:
   - README files and 3 subfolders at each level, repeating for 3 levels.

---

## Exploitation

### **Shell Script**

- Download everything and search for the flag:
  ```bash
        mkdir -p tmp && cd tmp
        wget -r -np -e robots=off http://<IP_ADDRESS>/.hidden/

        grep -r "flag" .
  ```

---

## Flag

`d5eec3afc10840462a44b987e7b9c283`

---

## Security Impact

- **Info Disclosure:** Sensitive files can be found by persistent attackers.
- **Automated Scraping:** Easy for bots to find everything.
- **Obscurity ≠ Security:** No real controls in place.

---

## Prevention

1. **Never rely on directory structure for security**
2. **Require authentication for sensitive data**
3. **Use proper authorization checks**
4. **robots.txt is NOT a security tool**
5. **Disable directory listing on servers**

---
