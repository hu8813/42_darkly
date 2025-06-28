# Exposed Admin Credentials Vulnerability

## Overview
Admin credentials were exposed via an unprotected `.htpasswd` file, letting attackers access the admin panel.

---

## How It Happened

1. **robots.txt Clue:**  
   `robots.txt` lists disallowed paths:
   ```
   Disallow: /whatever
   Disallow: /.hidden
   ```

2. **Finding the File:**  
   Visit `/whatever/htpasswd` to find:
   ```
   root:437394baff5aa33daa618be47b75cb49
   ```

3. **Cracking the Password:**  
   Decode the MD5 hash (`437394baff5aa33daa618be47b75cb49`) using [dcode.fr](https://www.dcode.fr/md5-hash) or [crackstation.net](https://crackstation.net)  
   → Password: `qwerty123@`

4. **Login:**  
   Use `root:qwerty123@` at `/admin`  
   The flag is displayed.

---

## Flag

`d19b4823e0d5600ceed56d5e896ef328d7a2b9e7ac7e80f4fcdb9b10bcb3e7ff`

---

## Prevention

1. **Store sensitive files outside web root**
2. **Restrict access at server config**
3. **Hash passwords securely (bcrypt, not MD5)**
4. **Don't trust robots.txt for real security**
5. **Regularly scan for exposed files**

---