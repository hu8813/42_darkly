# Open Redirect Vulnerability

## Overview
The site’s redirect feature lets anyone send users to any external link by changing the `site` parameter—no checks are done.

---

## How It Works

1. Social icons link to:
   ```
   index.php?page=redirect&site=facebook
   ```
2. Change `site` to anything, e.g.:
   ```
   index.php?page=redirect&site=https://evil.com
   ```
3. Visiting this link sends the user to the chosen site.

---

## Impact

- **Phishing**: Users can be tricked into fake login pages.
- **Malware**: Redirects to dangerous downloads.
- **Trust Abuse**: Uses your domain to trick users.

---

## Flag

`b9e775a0291fed784a2d9680fcfad7edd6b8cdf87648da647aaf4bba288bcab3`

---

## Prevention

1. Only allow redirects to trusted domains (whitelist).
2. Use tokens instead of direct URLs.
3. Show users a warning before redirecting.
4. Block suspicious or unknown URLs.
5. Avoid external redirects if possible.

---