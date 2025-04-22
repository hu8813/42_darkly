


async function findFlag() {
  const baseUrl = window.location.href.includes('/.hidden/') 
    ? window.location.href 
    : new URL('./.hidden/', window.location.href).href;
  
  console.log(`Starting search from ${baseUrl}`);
  
  // Queue for processing directories depth-first
  const queue = [{ url: baseUrl, depth: 0 }];
  const processedUrls = new Set();
  const uniqueContents = new Map(); // Map content to URL for reference
  
  let processedCount = 0;
  let readmeCount = 0;
  
  while (queue.length > 0) {
    // Get next directory from queue
    const { url, depth } = queue.shift();
    
    // Skip if already processed or too deep
    if (processedUrls.has(url) || depth > 3) continue;
    processedUrls.add(url);
    
    processedCount++;
    if (processedCount % 100 === 0) {
      console.log(`Processed ${processedCount} directories, found ${readmeCount} README files, ${uniqueContents.size} unique contents`);
    }
    
    try {
      // Fetch directory listing
      const response = await fetch(url);
      const html = await response.text();
      const parser = new DOMParser();
      const doc = parser.parseFromString(html, 'text/html');
      const links = Array.from(doc.querySelectorAll('a'));
      
      // Process README files first (they are our priority)
      for (const a of links) {
        const name = a.textContent.trim();
        if (name === 'README') {
          const readmePath = new URL(a.getAttribute('href'), url).href;
          try {
            const readmeResponse = await fetch(readmePath);
            const content = (await readmeResponse.text()).trim();
            readmeCount++;
            
            // Check if this is new content
            if (!uniqueContents.has(content)) {
              uniqueContents.set(content, readmePath);
              console.log(`New unique content found at ${readmePath}: "${content}"`);
              
              // Looking specifically for flag patterns
              if (content.includes('flag') || content.includes('d5eec3')) {
                console.log(`🚩 POTENTIAL FLAG FOUND: ${content}`);
                console.log(`🚩 FLAG URL: ${readmePath}`);
              }
            }
          } catch (readmeError) {
            // Skip errors on individual README files
          }
        }
      }
      
      // Then queue subdirectories (depth-first priority for deeper directories)
      for (const a of links) {
        const href = a.getAttribute('href');
        const name = a.textContent.trim();
        
        if (name.startsWith('..')) continue;
        
        if ((href.endsWith('/') || name.endsWith('/')) && depth < 3) {
          const fullPath = new URL(href, url).href;
          // Add to the beginning of the queue for depth-first processing
          queue.unshift({ url: fullPath, depth: depth + 1 });
        }
      }
      
      // Small delay to avoid overwhelming the browser
      await new Promise(resolve => setTimeout(resolve, 10));
      
    } catch (error) {
      // Skip errors on directory listing
    }
  }
  
  console.log(`\nSearch complete! Processed ${processedCount} directories, found ${readmeCount} README files`);
  console.log(`Found ${uniqueContents.size} unique contents:`);
  
  console.log('\n--- ALL UNIQUE README CONTENTS ---\n');
  for (const [content, url] of uniqueContents.entries()) {
    console.log(`${content}`);
    console.log('---');
  }
  
  return Object.fromEntries([...uniqueContents.entries()]);
}

(async () => {
  console.log('Starting sequential search for README contents...');
  window.flagResults = await findFlag();
  console.log('Search complete! Results in window.flagResults');
})();


